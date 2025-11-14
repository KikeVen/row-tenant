# Django Multi-Tenant Implementation Plan - Part 9: Code Examples & Configuration

## 9. Detailed Code Examples

### Complete Model Example
```python
# tenancy/models.py
import uuid
from django.db import models
from django.conf import settings
from .managers import TenantAwareManager

class Tenant(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=255, help_text="Company/Organization name")
    slug = models.SlugField(max_length=100, unique=True, db_index=True)
    subdomain = models.CharField(max_length=63, unique=True, null=True, blank=True)
    is_active = models.BooleanField(default=True)
    settings = models.JSONField(default=dict, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['name']
        indexes = [
            models.Index(fields=['slug']),
            models.Index(fields=['is_active']),
        ]
    
    def __str__(self):
        return self.name

class UserTenantMembership(models.Model):
    ROLE_CHOICES = [
        ('owner', 'Owner'),
        ('admin', 'Administrator'), 
        ('member', 'Member'),
        ('viewer', 'Viewer'),
    ]
    
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE,
                            related_name='tenant_memberships')
    tenant = models.ForeignKey(Tenant, on_delete=models.CASCADE,
                              related_name='user_memberships')
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='member')
    is_active = models.BooleanField(default=True)
    joined_at = models.DateTimeField(auto_now_add=True)
    invited_by = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.SET_NULL,
                                  null=True, blank=True, related_name='sent_invitations')
    
    class Meta:
        unique_together = [('user', 'tenant')]
        indexes = [
            models.Index(fields=['user', 'is_active']),
            models.Index(fields=['tenant', 'role']),
        ]
    
    def __str__(self):
        return f"{self.user.email} - {self.tenant.name} ({self.role})"

# Example of converting existing model
class Order(models.Model):
    # Add this field
    tenant = models.ForeignKey('tenancy.Tenant', on_delete=models.CASCADE,
                              related_name='orders')
    
    # Existing fields
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    total_amount = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    
    # Add custom manager
    objects = TenantAwareManager()
    
    class Meta:
        indexes = [
            models.Index(fields=['tenant', '-created_at']),  # Critical for performance
            models.Index(fields=['tenant', 'user']),
        ]
```

### Complete Middleware Example
```python
# tenancy/middleware.py
import logging
from django.utils.deprecation import MiddlewareMixin
from django.http import HttpResponseForbidden
from tenancy.models import UserTenantMembership

logger = logging.getLogger(__name__)

class TenantMiddleware(MiddlewareMixin):
    TENANT_OPTIONAL_URLS = ['/accounts/', '/admin/', '/static/', '/media/']
    
    def process_request(self, request):
        request.tenant = None
        request.user_tenant_role = None
        
        if self._is_tenant_optional_url(request.path):
            return None
            
        if not request.user.is_authenticated:
            return None
            
        try:
            membership = self._get_user_membership(request.user)
            if membership:
                request.tenant = membership.tenant
                request.user_tenant_role = membership.role
                logger.debug(f"Tenant context: {request.user.email} -> {membership.tenant.slug}")
        except Exception as e:
            logger.error(f"Error setting tenant context: {e}")
        
        return None
    
    def _is_tenant_optional_url(self, path):
        return any(path.startswith(url) for url in self.TENANT_OPTIONAL_URLS)
    
    def _get_user_membership(self, user):
        memberships = UserTenantMembership.objects.filter(
            user=user, is_active=True
        ).select_related('tenant')
        
        count = memberships.count()
        if count == 0:
            return None
        elif count == 1:
            return memberships.first()
        else:
            # Multiple memberships - return first active tenant
            # TODO: Implement tenant switching UI
            return memberships.filter(tenant__is_active=True).first()
```

### Complete View Update Example
```python
# Before tenant implementation
def order_list(request):
    orders = Order.objects.filter(user=request.user).order_by('-created_at')
    return render(request, 'orders/list.html', {'orders': orders})

def create_order(request):
    if request.method == 'POST':
        form = OrderForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.user = request.user
            order.save()
            return redirect('order_detail', order.id)
    else:
        form = OrderForm()
    return render(request, 'orders/create.html', {'form': form})

# After tenant implementation
from tenancy.decorators import require_tenant_access, require_role

@login_required
@require_tenant_access
def order_list(request):
    orders = Order.objects.for_tenant(request.tenant).filter(
        user=request.user
    ).order_by('-created_at')
    return render(request, 'orders/list.html', {
        'orders': orders,
        'tenant': request.tenant
    })

@login_required
@require_tenant_access
@require_role('member')  # At least member role required
def create_order(request):
    if request.method == 'POST':
        form = OrderForm(request.POST)
        if form.is_valid():
            order = form.save(commit=False)
            order.tenant = request.tenant  # Critical: Set tenant
            order.user = request.user
            order.save()
            return redirect('order_detail', order.id)
    else:
        form = OrderForm()
    return render(request, 'orders/create.html', {
        'form': form,
        'tenant': request.tenant
    })
```

## 10. Migration Scripts

### Data Migration Command
```python
# management/commands/migrate_to_tenants.py
from django.core.management.base import BaseCommand
from django.db import transaction
from tenancy.models import Tenant
from myapp.models import Order, Invoice, Customer  # Your models

class Command(BaseCommand):
    help = 'Migrate existing data to tenant structure'
    
    def add_arguments(self, parser):
        parser.add_argument('--tenant-slug', required=True,
                          help='Slug of tenant to assign data to')
        parser.add_argument('--dry-run', action='store_true',
                          help='Show what would be migrated without executing')
    
    def handle(self, *args, **options):
        tenant_slug = options['tenant_slug']
        dry_run = options['dry_run']
        
        try:
            tenant = Tenant.objects.get(slug=tenant_slug)
        except Tenant.DoesNotExist:
            self.stdout.write(self.style.ERROR(f'Tenant {tenant_slug} does not exist'))
            return
        
        models_to_migrate = [
            (Order, 'orders'),
            (Invoice, 'invoices'), 
            (Customer, 'customers'),
        ]
        
        if dry_run:
            self.stdout.write(self.style.WARNING('DRY RUN MODE - No changes will be made'))
        
        for model, name in models_to_migrate:
            unassigned = model.objects.filter(tenant__isnull=True)
            count = unassigned.count()
            
            self.stdout.write(f'{name}: {count} records need tenant assignment')
            
            if not dry_run and count > 0:
                with transaction.atomic():
                    updated = unassigned.update(tenant=tenant)
                    self.stdout.write(
                        self.style.SUCCESS(f'Updated {updated} {name} records')
                    )
```

## 11. Configuration Examples

### Settings Configuration
```python
# settings.py additions
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'tenancy.middleware.TenantMiddleware',  # Add after auth
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

INSTALLED_APPS = [
    # ... existing apps
    'tenancy',
]

# Optional: Tenant-specific settings
TENANT_SETTINGS = {
    'ENABLE_SUBDOMAIN_ROUTING': True,
    'DEFAULT_TIMEZONE': 'UTC',
    'ENABLE_MULTI_TENANT_USERS': True,  # Allow users in multiple tenants
}
```