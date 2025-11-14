# Django Multi-Tenant Implementation Prompt

Copy and paste this into a new chat window to implement tenant isolation:

---

I need to implement tenant isolation in my existing Django project. Please help me add multi-tenant functionality with the following requirements:

## Core Features Needed

- Row-level tenant isolation (complete data separation between organizations)
- Role-based access control (Owner/Admin/Member/Viewer permissions)
- Automatic tenant detection via middleware
- Tenant-scoped queries and managers

## My Project Setup

- Django 4.x+ project with custom User model
- PostgreSQL database
- Existing authentication system (django-allauth)
- [Describe your current models that need tenant isolation]

## Implementation Plan

### Phase 1: Foundation

1. Create `tenancy` app with core models (Tenant, UserTenantMembership)
2. Create tenant-aware managers and querysets
3. Add tenant foreign keys to existing models
4. Create migrations for existing data

### Phase 2: Middleware & Access Control

1. Create TenantMiddleware to set request.tenant context
2. Create access control decorators (@require_tenant_access, @require_role)
3. Update all views with tenant scoping
4. Create data migration commands

### Phase 3: Management & Utilities

1. Create management commands (create_tenant, assign_user_to_tenant)
2. Add Django admin interface for tenant management
3. Create tenant settings framework
4. Add basic PII management

## Key Models Needed

```python
class Tenant(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4)
    name = models.CharField(max_length=255)
    slug = models.SlugField(max_length=100, unique=True)
    is_active = models.BooleanField(default=True)
    settings = models.JSONField(default=dict)
    created_at = models.DateTimeField(auto_now_add=True)

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
```

## Request Flow Needed

1. Request arrives → TenantMiddleware
2. Lookup user's tenant membership
3. Set request.tenant and request.user_tenant_role
4. View checks @require_tenant_access decorator
5. Query uses .for_tenant(request.tenant)
6. Response returned with tenant-scoped data

## Critical Requirements

- Complete data isolation between tenants
- Backward compatibility with existing code
- Performance optimization with proper indexes
- Migration strategy for existing data
- Comprehensive testing for cross-tenant access prevention

Please help me implement this step by step, starting with Phase 1. Ask me about my current models and provide the specific code I need to create.

---
