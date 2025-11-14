# Django Multi-Tenant Implementation Plan - Part 2: Architecture & Design

## 3. Architecture & Design

### Core Components Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Middleware    │    │    Decorators    │    │    Managers     │
│                 │    │                  │    │                 │
│ TenantMiddleware├────┤@require_tenant   ├────┤TenantAwareManager│
└─────────────────┘    │@require_role     │    │TenantQuerySet   │
                       └──────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Models Layer                             │
│  ┌─────────┐  ┌──────────────────┐                             │
│  │ Tenant  │  │UserTenantMember  │                             │
│  │         │  │ship              │                             │
│  └─────────┘  └──────────────────┘                             │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           Existing Models + tenant FK                   │   │
│  │   Order, Invoice, Customer, etc.                       │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Data Model Design

**Tenant Isolation Strategy**: Row-Level Security (RLS)
- Each business model gets a `tenant` foreign key
- All queries automatically filtered by tenant
- No shared data between tenants

**User-Tenant Relationship**: Many-to-Many with Roles
- Users can belong to multiple tenants (future flexibility)
- Each membership has a role (Owner/Admin/Member/Viewer)
- Supports user switching between tenants

### Request Flow Design

```
1. Request arrives → TenantMiddleware
2. Lookup user's tenant membership
3. Set request.tenant and request.user_tenant_role
4. View checks @require_tenant_access decorator
5. Query uses .for_tenant(request.tenant)
6. Response returned with tenant-scoped data
```

### Role Hierarchy Design

```
OWNER (Level 4)
├─ Full admin access + billing
├─ Can manage all users and settings
├─ Can access all features
│
ADMIN (Level 3) 
├─ Can manage users and settings
├─ Cannot access billing
├─ Can access most features
│
MEMBER (Level 2)
├─ Standard user with read/write
├─ Cannot manage other users
├─ Can access core features
│
VIEWER (Level 1)
├─ Read-only access
├─ Cannot modify any data
└─ Can only view permitted content
```

### Database Schema Design

**Core Tenant Tables**:
- `tenancy_tenant` - Organization/company data
- `tenancy_usertenantmembership` - User roles per tenant

**Enhanced Existing Tables**:
- Add `tenant_id` foreign key to all business models
- Add indexes for `(tenant_id, created_at)` patterns
- Maintain existing primary keys and relationships