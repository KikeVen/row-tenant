# Django Multi-Tenant Implementation Plan - Part 5: Implementation Steps (Phase 2)

## 5. Implementation Steps - Phase 2: Middleware & Access Control

### Step 5: Create Tenant Middleware (Complexity: MEDIUM)
**Duration**: 4-5 hours

Create `tenancy/middleware.py`:

```python
class TenantMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.tenant = None
        request.user_tenant_role = None
        
        if request.user.is_authenticated:
            # Get user's tenant membership
            membership = self._get_user_membership(request.user)
            if membership:
                request.tenant = membership.tenant
                request.user_tenant_role = membership.role
```

**Add to settings.py**:
```python
MIDDLEWARE = [
    # ... existing middleware
    'tenancy.middleware.TenantMiddleware',
    # Must be after AuthenticationMiddleware
]
```

**Deliverables**:
- TenantMiddleware that sets request.tenant
- Optional TenantAccessMiddleware for strict enforcement
- Configuration in settings.py

### Step 6: Create Access Control Decorators (Complexity: MEDIUM)
**Duration**: 3-4 hours

Create `tenancy/decorators.py`:

```python
def require_tenant_access(view_func):
    @wraps(view_func)
    def wrapper(request, *args, **kwargs):
        if not hasattr(request, 'tenant') or request.tenant is None:
            return HttpResponseForbidden("No tenant access")
        return view_func(request, *args, **kwargs)
    return wrapper

def require_role(*required_roles):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            user_role = getattr(request, 'user_tenant_role', None)
            if user_role not in required_roles:
                return HttpResponseForbidden("Insufficient permissions")
            return view_func(request, *args, **kwargs)
        return wrapper
    return decorator
```

**Deliverables**:
- @require_tenant_access decorator
- @require_role decorator with hierarchy support
- Convenience decorators (@tenant_admin_required, etc.)

### Step 7: Update Views with Tenant Scoping (Complexity: HIGH)
**Duration**: 8-12 hours

Update all business logic views:

```python
# Before
def order_list(request):
    orders = Order.objects.filter(user=request.user)
    return render(request, 'orders.html', {'orders': orders})

# After
@login_required
@require_tenant_access
def order_list(request):
    orders = Order.objects.for_tenant(request.tenant).filter(user=request.user)
    return render(request, 'orders.html', {'orders': orders})
```

**Views to Update**:
- All CRUD operations on tenant-aware models
- List views, detail views, create/update forms
- API endpoints and serializers
- Admin interface views

**Deliverables**:
- All views updated with tenant scoping
- Proper error handling for tenant access
- Maintained backward compatibility where possible

### Step 8: Create Data Migration Strategy (Complexity: HIGH)
**Duration**: 6-8 hours

Create management command for existing data:

```python
# management/commands/assign_tenant_data.py
class Command(BaseCommand):
    def add_arguments(self, parser):
        parser.add_argument('--tenant-slug', required=True)
        parser.add_argument('--dry-run', action='store_true')
    
    def handle(self, *args, **options):
        tenant = Tenant.objects.get(slug=options['tenant_slug'])
        
        # Update all existing records
        models_to_update = [Order, Invoice, Customer]
        for model in models_to_update:
            count = model.objects.filter(tenant__isnull=True).update(tenant=tenant)
            self.stdout.write(f"Updated {count} {model.__name__} records")
```

**Deliverables**:
- Management command for data migration
- Backup strategy for existing data
- Rollback procedures if needed
- Documentation for migration process