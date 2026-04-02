---
name: new-module
description: >-
  Scaffold a new backend module for the Platform healthcare SaaS platform. Creates controller, crud_service, models, schema, and constants files with correct patterns.
  USE FOR: create module, new module, scaffold module, add module, new feature, new endpoint, add API, create API, new controller, new route, scaffold feature, add feature module, create backend feature.
  DO NOT USE FOR: modifying existing modules, database migrations (use migrations skill), code review (use code-review skill), creating mockups (use mockup-guidelines skill).
---

# New Module — Platform Backend

> **Purpose:** Scaffold a new backend module with all required files following established project patterns. Every new module needs 5-6 files created correctly and wired into the app.

---

## 0. Before You Start — Clarifying Questions

**Always confirm before scaffolding:**

1. **Module name?** Use lowercase_snake_case (e.g., `payer_rebate_management`, `vbc_marketplace`)
2. **Which app?** ValueIQ (`VBC`), RebateIQ (`REBATE`), AccessIQ (`INTELLIGENT_WORKSPACE`), or platform-level?
3. **What entities/tables?** List the main data objects this module will manage.
4. **Tenant-scoped or platform-scoped?** Most feature modules are tenant-scoped.
5. **What operations?** CRUD? List/detail? Custom business logic?
6. **Permissions needed?** What permission names and which roles?
7. **Does it need AI integration?** If yes, check `enable_ai` in app config.

---

## 1. Module Directory Structure

Every module lives under `app/<module_name>/` with these files:

```
app/<module_name>/
├── __init__.py          # Empty file (required for Python package)
├── controller.py        # Routes / API endpoints
├── crud_service.py      # Database operations (repository pattern)
├── models.py            # SQLAlchemy ORM models
├── schema.py            # Pydantic request/response schemas
├── constants.py         # Enums, constants (optional, create if needed)
└── service.py           # Cross-module business logic (optional)
```

---

## 2. File Templates

### 2.1 `__init__.py`

```python

```

Empty file. Just needs to exist.

### 2.2 `controller.py`

```python
from typing import Annotated

from common.authentication import authenticated
from common.authorization import has_permissions
from common.decorators.is_app import is_app
from common.logger import PlatformLogger
from common.rate_limit import limiter
from common.security import (
    get_context_vars,
    get_current_user,
    get_current_user_organization_id,
)
from http import HTTPStatus
from fastapi import APIRouter, Depends, HTTPException, Request, Response
from fastapi.encoders import jsonable_encoder
from fastapi_versioning import version
from role.constants import BuiltInPermissions
from tenant.constants import PlatformApps

from <module_name> import crud_service
from <module_name>.schema import (
    Create<Entity>Body,
    <Entity>Schema,
    Update<Entity>Body,
)

logger = PlatformLogger(__name__)

<module_name>_router = APIRouter(
    prefix="/<url-prefix>",
    tags=["<Display Name>"],
)


@<module_name>_router.get("", dependencies=[Depends(authenticated)])
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_VIEW.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def get_all(request: Request):
    """List all <entities>."""
    _, db = get_context_vars()
    current_user = get_current_user()
    items = crud_service.get_all()
    return items


@<module_name>_router.get("/{item_id}", dependencies=[Depends(authenticated)])
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_VIEW.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def get_by_id(request: Request, item_id: int):
    """Get a single <entity> by ID."""
    item = crud_service.get_by_id(item_id)
    if not item:
        raise HTTPException(status_code=404, detail="<Entity> not found")
    return item


@<module_name>_router.post("", dependencies=[Depends(authenticated)])
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_CREATE.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def create(request: Request, response: Response, body: Create<Entity>Body):
    """Create a new <entity>."""
    current_user = get_current_user()
    item = crud_service.create_<entity>(body, current_user.email)
    return item


@<module_name>_router.put("/{item_id}", dependencies=[Depends(authenticated)])
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_EDIT.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def update(request: Request, item_id: int, body: Update<Entity>Body):
    """Update an existing <entity>."""
    current_user = get_current_user()
    item = crud_service.update_<entity>(item_id, body, current_user.email)
    if not item:
        raise HTTPException(status_code=404, detail="<Entity> not found")
    return item


@<module_name>_router.delete(
    "/{item_id}",
    dependencies=[Depends(authenticated)],
    status_code=HTTPStatus.NO_CONTENT,
)
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_DELETE.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def delete(request: Request, item_id: int):
    """Delete an <entity>."""
    crud_service.delete_<entity>(item_id)
    # No return — 204 is set in the decorator
```

**Key rules:**

- Router variable: `<module_name>_router` (this is what gets imported in `main.py`)
- Route prefix: lowercase, hyphen-separated URL path (e.g., `/rebate-gtn-modelling`)
- All handlers are **sync `def`** (NOT `async def`)
- `request: Request` is always the first parameter (required by rate limiter)
- Decorator order: route → `@version(1)` → `@has_permissions` → `@is_app` → `@limiter.limit`
- `dependencies=[Depends(authenticated)]` goes on the route decorator, not as a separate decorator

#### Three-User Authorization Template

For modules that need caller-type branching (Admin / Manufacturer / Payer), add this pattern inside route handlers:

```python
from common.utils import is_customer_user
from fastapi import Header
from typing import Annotated

@router.get("", dependencies=[Depends(authenticated)])
@version(1)
@has_permissions(BuiltInPermissions.<MODULE>_VIEW.value)
@is_app([PlatformApps.<APP>])
@limiter.limit("30/minute")
def get_items(
    request: Request,
    response: Response,
    x_tenant_id: Annotated[str, Header()],
    x_tenant_app_id: Annotated[int, Header()],
    participant_id: int = Query(...),  # Required for payer, optional for manufacturer/Admin
):
    current_user = get_current_user()
    org_id = get_current_user_organization_id()
    _, db = get_context_vars()

    tenant_app = get_tenant_app_by_id(x_tenant_app_id)

    if current_user.is_admin_user:
        # Admin: full access, no ownership check
        pass
    elif is_customer_user(current_user, tenant_app.tenant, org_id):
        # Manufacturer: access manufacturer-scoped data only
        pass
    else:
        # Payer: validate ownership
        participant = get_participant_by_id(participant_id)
        if participant.organization_id != org_id:
            raise HTTPException(status_code=403, detail="User does not have access to this participant")

    return crud_service.get_items(x_tenant_app_id, participant_id)
```

**Reference:** `app/intelligent_workspace/launch_planning/controller.py`

#### `tenant_app_id` Body Validation

For POST/PATCH routes with a body containing `tenant_app_id`, validate it matches the header:

```python
def create_item(
    request: Request,
    body: CreateItemBody,
    x_tenant_app_id: Annotated[int, Header()],
):
    if body.tenant_app_id != x_tenant_app_id:
        raise HTTPException(status_code=400, detail="tenant_app_id mismatch")
    ...
```

### 2.3 `crud_service.py`

```python
from datetime import datetime

from common.security import get_context_vars, get_current_user

from <module_name>.models import <Entity>


def get_all(tenant_app_id: int = None):
    _, db = get_context_vars()
    query = db.query(<Entity>)
    if tenant_app_id:
        query = query.filter(<Entity>.tenant_app_id == tenant_app_id)
    return query.all()


def get_by_id(item_id: int):
    _, db = get_context_vars()
    return db.query(<Entity>).filter(<Entity>.id == item_id).first()


def create_<entity>(data, created_by: str, in_transaction: bool = False):
    _, db = get_context_vars()
    item = <Entity>(
        tenant_app_id=data.tenant_app_id,
        name=data.name,
        created_by=created_by,
    )
    db.add(item)
    db.flush()  # Always flush to get auto-generated ID
    if not in_transaction:
        db.commit()
        db.refresh(item)
    return item


def update_<entity>(item_id: int, data, updated_by: str, in_transaction: bool = False):
    _, db = get_context_vars()
    item = db.query(<Entity>).filter(<Entity>.id == item_id).first()
    if not item:
        return None
    item.name = data.name
    item.updated_by = updated_by
    item.updated_at = datetime.now()
    db.flush()
    if not in_transaction:
        db.commit()
        db.refresh(item)
    return item


def delete_<entity>(item_id: int, in_transaction: bool = False):
    _, db = get_context_vars()
    item = db.query(<Entity>).filter(<Entity>.id == item_id).first()
    if item:
        db.delete(item)
        db.flush()
        if not in_transaction:
            db.commit()
```

**Key rules:**

- Free functions, NOT a class
- `get_context_vars()` and `get_current_user()` called inside functions — NEVER passed as parameters
- Every write function has `in_transaction: bool = False` parameter
- Always `flush()` first, then conditionally `commit()`
- `refresh()` after commit to get DB-generated values

### 2.4 `models.py`

```python
from datetime import datetime
from typing import Optional

from common.database import Base
from sqlalchemy import DATETIME, Integer, String, text
from sqlalchemy.orm import Mapped, mapped_column


class <Entity>(Base):
    __tablename__ = "<TABLE_NAME>"
    __table_args__ = {"schema": None}  # CRITICAL: schema injected at runtime

    id: Mapped[int] = mapped_column(
        Integer, primary_key=True, autoincrement=True, index=True
    )
    tenant_app_id: Mapped[int] = mapped_column(Integer, nullable=False)
    name: Mapped[str] = mapped_column(String(255), nullable=False)
    status: Mapped[str] = mapped_column(String(50), nullable=False, default="Active")
    created_by: Mapped[str] = mapped_column(String(100), nullable=False)
    created_at: Mapped[datetime] = mapped_column(
        DATETIME, server_default=text("SYSDATETIME()"), nullable=False
    )
    updated_by: Mapped[Optional[str]] = mapped_column(String(100), nullable=True)
    updated_at: Mapped[Optional[datetime]] = mapped_column(DATETIME, nullable=True)
```

**Key rules:**

- Inherit from `Base` (from `common.database`)
- `__table_args__ = {"schema": None}` is **mandatory** for tenant tables — without it, multi-tenancy breaks
- Table name: `UPPER_SNAKE_CASE` string
- Use SQLAlchemy 2.0 style: `Mapped[T]` + `mapped_column()`
- Timestamps: `server_default=text("SYSDATETIME()")` (SQL Server function)
- ForeignKey references use bare table name without schema: `ForeignKey("PARENT_TABLE.id")`
- Use `Decimal` type for money columns: `Mapped[Decimal] = mapped_column(Numeric(18, 4))`

#### Relationships & Constraints

For models with relationships, use the following patterns:

```python
from sqlalchemy import ForeignKey, UniqueConstraint
from sqlalchemy.ext.hybrid import hybrid_property
from sqlalchemy.orm import relationship


class ParentEntity(Base):
    __tablename__ = "PARENT_TABLE"
    __table_args__ = (
        UniqueConstraint("tenant_app_id", "participant_id"),  # Tuple syntax for constraints + schema
        {"schema": None},
    )

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True, index=True)
    participant_id: Mapped[int] = mapped_column(
        Integer, ForeignKey("INTELLIGENT_WORKSPACE_PARTICIPANT.id"), nullable=False
    )

    # Relationships — lazy="select" by default to avoid unnecessary JOINs in list queries
    children = relationship("ChildEntity", back_populates="parent", lazy="select")
    participant = relationship("IntelligentWorkspaceParticipant", lazy="joined")  # Always eager-load

    @hybrid_property
    def computed_field(self) -> Optional[str]:
        """Derived field for list-view schemas. Requires children to be loaded (joinedload in query)."""
        if not self.children:
            return None
        return some_derivation(self.children)


class ChildEntity(Base):
    __tablename__ = "CHILD_TABLE"
    __table_args__ = {"schema": None}

    id: Mapped[int] = mapped_column(Integer, primary_key=True, autoincrement=True, index=True)
    parent_id: Mapped[int] = mapped_column(
        Integer, ForeignKey(ParentEntity.id), nullable=False
    )
    parent = relationship("ParentEntity", back_populates="children")
```

**Query patterns with relationships:**

```python
from sqlalchemy.orm import joinedload

# List view — only load what's needed for summary schemas
def get_all(tenant_app_id):
    _, db = get_context_vars()
    return db.query(ParentEntity).options(
        joinedload(ParentEntity.children)  # Needed for hybrid_property
    ).filter(ParentEntity.tenant_app_id == tenant_app_id).all()

# Detail view — load all relationships
def get_by_id(item_id):
    _, db = get_context_vars()
    return db.query(ParentEntity).options(
        joinedload(ParentEntity.children),
        joinedload(ParentEntity.other_relationship),
    ).filter(ParentEntity.id == item_id).first()
```

**Reference:** `app/intelligent_workspace/launch_planning/models.py`, `app/intelligent_workspace/launch_planning/crud_service.py`

### 2.5 `schema.py`

```python
from datetime import datetime
from typing import Optional

from common.base_schema import BaseSchema
from pydantic import Field


class <Entity>Schema(BaseSchema):
    """Response schema for <Entity>."""
    id: int
    tenant_app_id: int
    name: str
    status: str
    created_by: str
    created_at: datetime
    updated_by: Optional[str] = None
    updated_at: Optional[datetime] = None


class Create<Entity>Body(BaseSchema):
    """Request body for creating an <Entity>."""
    tenant_app_id: int = Field(gt=0)
    name: str = Field(min_length=1, max_length=255)


class Update<Entity>Body(BaseSchema):
    """Request body for updating an <Entity>."""
    name: str = Field(min_length=1, max_length=255)
```

**Key rules:**

- Inherit from `BaseSchema` (from `common.base_schema`), NEVER from `pydantic.BaseModel`
- `BaseSchema` has `use_enum_values=True` — enum fields become strings on parse. Compare with `.value`
- `BaseSchema` has `from_attributes=True` — schemas can be created from SQLAlchemy objects
- Use `Field()` for validation on request body schemas
- Use `datetime` type for timestamps (not `str`)
- Use `Decimal` for money fields (not `float`)
- Use `Optional[T] = None` for nullable fields

### 2.6 `constants.py` (Optional)

```python
from enum import Enum


class <Entity>Status(Enum):
    ACTIVE = "Active"
    INACTIVE = "Inactive"
    DRAFT = "Draft"
```

Create only if the module has enums or constants. Otherwise, skip this file.

---

## 3. Wiring Into the App

After creating the module files, you must register the router.

### Top-Level Module → Register in `app/main.py`

```python
# Add import:
from <module_name>.controller import <module_name>_router

# Add router registration:
app.include_router(<module_name>_router)
```

### Sub-Module → Register on Parent Router (NOT `main.py`)

Sub-modules live inside an existing module directory and register their router on the **parent module's router**:

```
app/intelligent_workspace/
  controller.py                    # Parent: intelligent_workspace_router
  launch_planning/                 # Sub-module
    __init__.py
    controller.py                  # launch_planning_router (APIRouter with prefix)
    crud_service.py
    models.py
    schema.py
    constants.py
```

```python
# In parent controller.py — add import and include:
from intelligent_workspace.launch_planning.controller import launch_planning_router

intelligent_workspace_router.include_router(launch_planning_router)
```

Resulting URL paths nest automatically: `/v1/intelligent-workspace/launch-planning/...`

Sub-module routers use their own prefix and tags:

```python
# In sub-module controller.py:
launch_planning_router = APIRouter(
    prefix="/launch-planning",
    tags=["Intelligent Workspace", "Launch Planning"],
)
```

**Reference:** `app/intelligent_workspace/controller.py` (parent), `app/intelligent_workspace/launch_planning/controller.py` (sub-module)

---

## 4. Permissions Setup

If the module needs permissions:

### Step 1: Add to `BuiltInPermissions` Enum

In `app/role/constants.py`, add the new permission values:

```python
class BuiltInPermissions(Enum):
    # ... existing permissions ...
    <MODULE>_VIEW = "<MODULE_NAME>.VIEW"
    <MODULE>_CREATE = "<MODULE_NAME>.CREATE"
    <MODULE>_EDIT = "<MODULE_NAME>.EDIT"
    <MODULE>_DELETE = "<MODULE_NAME>.DELETE"
```

### Step 2: Create a Migration

Use the **migrations** skill to create a migration that inserts the permissions into `PLATFORM.ROLE_PERMISSION` for the appropriate roles.

---

## 5. Database Table Setup

If the module has new tenant tables:

### Step 1: Create a Migration

Use the **migrations** skill to create the table DDL.

### Step 2: Update `tables.py`

Add the table DDL to `app/tenant/tables.py` so new tenant provisioning includes it.

### Step 3: Create the SQLAlchemy Model

Follow the `models.py` template in Section 2.4.

---

## 6. Audit Logging (When Needed)

For create, update, and delete operations on important entities, add audit logging:

```python
from audit_log.audit_service import create_audit_log
from audit_log.constants import AuditEntityAction, AuditEntityName
from fastapi.encoders import jsonable_encoder

create_audit_log(
    user_email=current_user.email,
    user_organization_id=get_current_user_organization_id(),
    entity_name=AuditEntityName.<ENTITY>,
    entity_action=AuditEntityAction.ADDED,
    entity_data=jsonable_encoder(item),
    api_request_data=request,
)
```

If the entity name doesn't exist in `AuditEntityName`, add it to `app/audit_log/constants.py`.

---

## 7. Post-Scaffold Checklist

After creating all files, verify:

- [ ] `__init__.py` exists in the module directory
- [ ] `controller.py` — router variable named `<module>_router`, correct prefix and tags
- [ ] `controller.py` — all routes have `dependencies=[Depends(authenticated)]`
- [ ] `controller.py` — decorator order: route → `@version(1)` → `@has_permissions` → `@is_app` → `@limiter`
- [ ] `controller.py` — `authenticated` imported from `common.authentication` (NOT `common.security`)
- [ ] `controller.py` — all handlers are sync `def` (NOT `async def`)
- [ ] `crud_service.py` — uses `get_context_vars()` inside functions, never as parameter
- [ ] `crud_service.py` — write functions have `in_transaction` parameter
- [ ] `crud_service.py` — `flush()` before conditional `commit()`
- [ ] `service.py` — NO direct DB access (`db.query`, `db.add`, `get_context_vars`) — calls `crud_service` functions only
- [ ] `models.py` — inherits from `Base`
- [ ] `models.py` — `__table_args__ = {"schema": None}` present on all tenant models
- [ ] `models.py` — uses `Mapped[]` + `mapped_column()` (SQLAlchemy 2.0 style)
- [ ] `schema.py` — inherits from `BaseSchema` (NOT `pydantic.BaseModel`)
- [ ] `main.py` — router imported and `include_router()` added
- [ ] Permissions added to `BuiltInPermissions` enum in `app/role/constants.py`
- [ ] Migration created for new tables and permissions (use migrations skill)
- [ ] `app/tenant/tables.py` updated for new tenant tables

---

## 8. Don'ts

- **Don't use `async def` for route handlers.** The app uses sync SQLAlchemy — sync handlers run in FastAPI's threadpool.
- **Don't import `authenticated` from `common.security`.** Import from `common.authentication`.
- **Don't inherit schemas from `pydantic.BaseModel`.** Use `BaseSchema`.
- **Don't pass `db` or `current_user` as function parameters.** Use `get_context_vars()` and `get_current_user()`.
- **Don't forget `__table_args__ = {"schema": None}`.** Multi-tenancy will silently break.
- **Don't use `float` for money.** Use `Decimal` / `Numeric(18, 4)`.
- **Don't store mutable state in module-level globals.** Multi-worker deployment means each worker has separate memory.
- **Don't skip the migration.** New tables need both a migration file AND a `tables.py` update.
