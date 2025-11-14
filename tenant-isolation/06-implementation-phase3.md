# Django Multi-Tenant Implementation Plan - Part 6: Implementation Steps (Phase 3)

## 5. Implementation Steps - Phase 3: Management & Utilities

### Step 9: Create Management Commands (Complexity: LOW)
**Duration**: 2-3 hours

Create essential management commands:

```python
# create_tenant.py
class Command(BaseCommand):
    def add_arguments(self, parser):
        parser.add_argument('--name', required=True)
        parser.add_argument('--slug', required=True)
        parser.add_argument('--admin-email', required=True)
    
    def handle(self, *args, **options):
        # Create tenant and admin user membership
        pass

# tenant_health_check.py  
class Command(BaseCommand):
    def handle(self, *args, **options):
        # Verify tenant data integrity
        pass
```

**Commands to Create**:
- `create_tenant` - Create new tenant with admin user
- `assign_user_to_tenant` - Add user to tenant with role
- `tenant_health_check` - Verify tenant data integrity
- `list_tenants` - Show all tenants and their status

**Deliverables**:
- Complete set of tenant management commands
- Help text and argument validation
- Error handling and logging

### Step 10: Add Basic PII Management (Complexity: MEDIUM)
**Duration**: 3-4 hours

Create `tenancy/pii.py`:

```python
class PIIExportRequest(models.Model):
    """Track user data export requests for compliance"""
    tenant = models.ForeignKey(Tenant, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    requested_at = models.DateTimeField(auto_now_add=True)
    status = models.CharField(max_length=50, default='pending')
    completed_at = models.DateTimeField(null=True, blank=True)

class PIIErasureRequest(models.Model):
    """Track user data deletion requests"""
    tenant = models.ForeignKey(Tenant, on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    requested_at = models.DateTimeField(auto_now_add=True)
    status = models.CharField(max_length=50, default='pending')
```

**Deliverables**:
- Basic PII request tracking models
- Management commands for data export/deletion
- Simple compliance reporting

### Step 11: Create Admin Interface (Complexity: MEDIUM)
**Duration**: 4-5 hours

Create `tenancy/admin.py`:

```python
@admin.register(Tenant)
class TenantAdmin(admin.ModelAdmin):
    list_display = ['name', 'slug', 'is_active', 'created_at']
    list_filter = ['is_active', 'created_at']
    search_fields = ['name', 'slug']
    readonly_fields = ['id', 'created_at', 'updated_at']

@admin.register(UserTenantMembership)
class UserTenantMembershipAdmin(admin.ModelAdmin):
    list_display = ['user', 'tenant', 'role', 'is_active', 'joined_at']
    list_filter = ['role', 'is_active', 'tenant']
    search_fields = ['user__email', 'tenant__name']
```

**Deliverables**:
- Django admin interface for tenant management
- User membership administration
- Basic reporting and filtering

### Step 12: Add Tenant Settings Framework (Complexity: MEDIUM)
**Duration**: 3-4 hours

Extend Tenant model settings field:

```python
# tenancy/settings_framework.py
class TenantSettings:
    """Type-safe tenant settings management"""
    
    def __init__(self, tenant):
        self.tenant = tenant
    
    def get(self, key, default=None):
        return self.tenant.settings.get(key, default)
    
    def set(self, key, value):
        self.tenant.settings[key] = value
        self.tenant.save(update_fields=['settings'])
    
    # Add property methods for common settings
    @property
    def timezone(self):
        return self.get('timezone', 'UTC')
    
    @timezone.setter
    def timezone(self, value):
        self.set('timezone', value)
```

**Deliverables**:
- Type-safe settings management framework
- Common tenant configuration options
- Settings validation and defaults