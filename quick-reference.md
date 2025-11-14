# Tenant Isolation Quick Reference

## For New Chat Windows - Quick Start

### 1. Copy this prompt to start implementation:

```
I need to implement Django tenant isolation following a proven plan.

Key requirements:
- Row-level tenant isolation with complete data separation
- Role-based access (Owner/Admin/Member/Viewer)
- Automatic tenant context via middleware
- Tenant-scoped queries (.for_tenant() method)

My setup:
- Django 4.x with custom User model
- PostgreSQL database
- [List your models that need tenant isolation]

Please start with Phase 1: Creating the tenancy app and core models.
```

### 2. Reference Architecture

**Core Models Structure:**
```python
class Tenant(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.CharField(max_length=255)
    slug = models.SlugField(max_length=100, unique=True)
    is_active = models.BooleanField(default=True)
    settings = models.JSONField(default=dict)

class UserTenantMembership(models.Model):
    user = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    tenant = models.ForeignKey(Tenant, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES)
    is_active = models.BooleanField(default=True)
```

**Implementation Phases:**
1. **Foundation**: Core models, managers, add tenant FKs to existing models
2. **Middleware**: TenantMiddleware, access decorators, update views
3. **Management**: Commands, admin, settings framework

### 3. Key Files to Reference

If you have access to the full plan, use these files in order:
- `01-overview-and-architecture.md` - Start here
- `04-implementation-phase1.md` - Foundation setup
- `05-implementation-phase2.md` - Middleware setup
- `06-implementation-phase3.md` - Management tools
- `09-code-examples-configuration.md` - Complete code examples

### 4. Critical Implementation Points

**Middleware Setup:**
```python
# settings.py
MIDDLEWARE = [
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'tenancy.middleware.TenantMiddleware',  # Add after auth
    # ... other middleware
]
```

**View Updates:**
```python
# Before
def my_view(request):
    items = MyModel.objects.filter(user=request.user)

# After
@login_required
@require_tenant_access
def my_view(request):
    items = MyModel.objects.for_tenant(request.tenant).filter(user=request.user)
```

**Model Updates:**
```python
# Add to existing models
class MyModel(models.Model):
    tenant = models.ForeignKey('tenancy.Tenant', on_delete=models.CASCADE)  # Add this
    # ... existing fields
    objects = TenantAwareManager()  # Add this
```

### 5. Testing Checklist

- [ ] Cross-tenant data access blocked
- [ ] All views require tenant context
- [ ] Role-based permissions working
- [ ] Migration preserves existing data
- [ ] Performance acceptable with indexes