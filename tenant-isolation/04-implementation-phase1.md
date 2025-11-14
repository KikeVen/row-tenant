# Django Multi-Tenant Implementation Plan - Part 4: Implementation Steps (Phase 1)

## 5. Implementation Steps - Phase 1: Foundation

### Step 1: Create Tenancy App (Complexity: LOW)
**Duration**: 1-2 hours

```bash
# Create new Django app
python manage.py startapp tenancy

# Add to INSTALLED_APPS
# settings.py
INSTALLED_APPS = [
    # ... existing apps
    'tenancy',
]
```

**Deliverables**:
- Empty tenancy app structure
- Basic app configuration

### Step 2: Create Core Models (Complexity: MEDIUM)
**Duration**: 4-6 hours

Create `tenancy/models.py` with:

```python
# Core tenant model
class Tenant(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.CharField(max_length=255)
    slug = models.SlugField(max_length=100, unique=True)
    is_active = models.BooleanField(default=True)
    settings = models.JSONField(default=dict)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

# User-tenant relationship with roles
class UserTenantMembership(models.Model):
    ROLE_CHOICES = [
        ('owner', 'Owner'),
        ('admin', 'Administrator'),
        ('member', 'Member'),
        ('viewer', 'Viewer'),
    ]
    
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    tenant = models.ForeignKey(Tenant, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES)
    is_active = models.BooleanField(default=True)
    joined_at = models.DateTimeField(auto_now_add=True)
```

**Deliverables**:
- Tenant and UserTenantMembership models
- Proper meta configurations and indexes
- String representations and basic methods

### Step 3: Create Tenant-Aware Managers (Complexity: MEDIUM)
**Duration**: 3-4 hours

Create `tenancy/managers.py`:

```python
class TenantAwareQuerySet(models.QuerySet):
    def for_tenant(self, tenant):
        return self.filter(tenant=tenant)
    
    def for_user(self, user):
        # Get user's tenant IDs and filter
        pass

class TenantAwareManager(models.Manager):
    def get_queryset(self):
        return TenantAwareQuerySet(self.model, using=self._db)
    
    def for_tenant(self, tenant):
        return self.get_queryset().for_tenant(tenant)
```

**Deliverables**:
- Custom QuerySet with tenant filtering methods
- Manager that provides .for_tenant() convenience
- Documentation for usage patterns

### Step 4: Add Tenant Fields to Existing Models (Complexity: HIGH)
**Duration**: 6-8 hours

For each existing business model:

```python
# Before
class Order(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    # ... existing fields

# After  
class Order(models.Model):
    tenant = models.ForeignKey('tenancy.Tenant', on_delete=models.CASCADE)
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    # ... existing fields
    
    objects = TenantAwareManager()  # Add custom manager
```

**Models to Update**: List all business models that need tenant isolation
- Orders, Invoices, Customers, Products, etc.
- Any model containing user-specific data
- Configuration and settings models

**Deliverables**:
- Migration files for all model changes
- Updated model definitions with tenant FK
- Custom managers added to all tenant-aware models