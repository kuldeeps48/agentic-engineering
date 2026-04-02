---
name: migrations
description: >-
  Create and manage SQL migrations for the COEBRA healthcare SaaS backend (Azure SQL Server). Handles tenant and platform schema changes, permission inserts, table creation, and tables.py sync.
  USE FOR: create migration, add column, alter table, create table, drop column, add index, add permission, new permission, schema change, database change, modify table, rename column, add foreign key, migration file, SQL migration, tenant table, platform table.
  DO NOT USE FOR: writing application code (use copilot-instructions.md), creating mockups (use mockup-guidelines skill), code review (use code-review skill).
---

# Migrations — COEBRA Platform Backend

> **CRITICAL:** Getting migrations wrong can silently break tenant provisioning, corrupt data, or cause production failures. Follow every step carefully.

---

## 0. Before You Start — Clarifying Questions

**Always confirm before writing a migration:**

1. **What is the change?** New table, new column, new index, new permission, alter existing column?
2. **Tenant-scoped or platform-scoped?** This determines whether you use `{schema_name}` placeholder or `PLATFORM.` prefix.
3. **Which app?** (ValueIQ/RebateIQ/AccessIQ) — affects permission role assignments.
4. **New permissions needed?** If so, which roles should get them?
5. **Does this affect an existing table?** If so, check the current DDL in `app/tenant/tables.py` or existing migrations.

---

## 1. Migration File Naming

### Convention: `YYYY_N_SEQ_REV.sql`

| Part   | Meaning                               | Example       |
| ------ | ------------------------------------- | ------------- |
| `YYYY` | Year                                  | `2026`        |
| `N`    | Major version                         | `2` (current) |
| `SEQ`  | Sequence number (increments globally) | `49`          |
| `REV`  | Revision (usually `1`)                | `1`           |

### Finding the Next Sequence Number

**Always check `_migrations/` for the highest current SEQ before creating a new file.**

```bash
ls -1 _migrations/*.sql | sort -t_ -k3 -n | tail -3
```

Increment the highest SEQ by 1. For example, if the latest is `2026_2_48_1.sql`, the next file is `2026_2_49_1.sql`.

### File Location

- Place new migrations in `_migrations/` (root level, NOT in `_migrations/2025_and_older/`)

---

## 2. Migration File Structure

Every migration file should follow this structure:

```sql
-- ========== PLATFORM MIGRATIONS ==========

-- [Platform-scoped changes here, if any]

-- ========== TENANT MIGRATIONS ==========
-- NOTE: Make sure app/tenant/tables.py is in sync

-- [Tenant-scoped changes here, if any]
```

If there are only platform changes, omit the tenant section (and vice versa). But always include the section header comments for clarity.

---

## 3. Platform vs. Tenant — Decision Guide

| Scope        | Schema Prefix              | When to Use                                                              | Examples                                                         |
| ------------ | -------------------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------- |
| **Platform** | `PLATFORM.TABLE_NAME`      | Shared across all tenants: users, roles, orgs, audit logs, system config | `PLATFORM.USERS`, `PLATFORM.ROLES`, `PLATFORM.ROLE_PERMISSION`   |
| **Tenant**   | `{schema_name}.TABLE_NAME` | Per-tenant data isolation: business entities, app-specific data          | `{schema_name}.VBC_PARTICIPANT`, `{schema_name}.REBATE_CONTRACT` |

**Key rule:** Tenant tables use `{schema_name}` placeholder — this gets replaced at runtime with the actual tenant schema name (e.g., `TENANT_ABC123`).

---

## 4. Common Migration Patterns

### 4.1 Create a New Tenant Table

```sql
CREATE TABLE {schema_name}.[NEW_TABLE_NAME] (
    ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    TENANT_APP_ID INT NOT NULL,
    NAME NVARCHAR(255) NOT NULL,
    STATUS NVARCHAR(50) NOT NULL DEFAULT 'Active',
    CREATED_BY NVARCHAR(100) NOT NULL,
    CREATED_AT DATETIME2 NOT NULL DEFAULT SYSDATETIME(),
    UPDATED_BY NVARCHAR(100) NULL,
    UPDATED_AT DATETIME2 NULL
);
CREATE INDEX IDX_NEW_TABLE_NAME_TENANT_APP_ID ON {schema_name}.[NEW_TABLE_NAME](TENANT_APP_ID);
CREATE INDEX IDX_NEW_TABLE_NAME_STATUS ON {schema_name}.[NEW_TABLE_NAME](STATUS);
```

**Conventions:**

- Table names: `UPPER_SNAKE_CASE`, wrapped in `[]`
- Primary key: `ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY`
- Always include: `CREATED_BY`, `CREATED_AT`, `UPDATED_BY`, `UPDATED_AT`
- Use `NVARCHAR` for text (Unicode support), `VARCHAR` only for ASCII-only fields like emails
- Use `DECIMAL(18,4)` for money/percentage values, never `FLOAT`
- Use `DATETIME2` for timestamps, not `DATETIME`
- Use `BIT` for booleans with `DEFAULT 0` or `DEFAULT 1`
- Index naming: `IDX_TABLE_NAME_COLUMN_NAME`

### 4.2 Create a New Platform Table

```sql
CREATE TABLE PLATFORM.[NEW_TABLE_NAME] (
    ID INT NOT NULL IDENTITY(1,1) PRIMARY KEY,
    -- columns...
    CREATED_AT DATETIME2 NOT NULL DEFAULT SYSDATETIME()
);
```

No `{schema_name}` placeholder — use `PLATFORM.` directly.

### 4.3 Add Column to Existing Table

```sql
-- Tenant table
ALTER TABLE {schema_name}.[EXISTING_TABLE] ADD NEW_COLUMN NVARCHAR(255) NULL;

-- Platform table
ALTER TABLE PLATFORM.[EXISTING_TABLE] ADD NEW_COLUMN NVARCHAR(255) NULL;
```

**Rules:**

- New columns on existing tables should almost always be `NULL` (existing rows won't have the value)
- If you need a `NOT NULL` column, add it as `NULL` first, backfill data, then alter to `NOT NULL`

### 4.4 Add Permissions

Permissions are inserted directly into `PLATFORM.ROLE_PERMISSION`. There is NO separate permissions table.

```sql
INSERT INTO PLATFORM.ROLE_PERMISSION (ROLE_ID, PERMISSION_NAME) VALUES
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Super Admin'), 'MODULE_NAME.VIEW'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Super Admin'), 'MODULE_NAME.CREATE'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Super Admin'), 'MODULE_NAME.EDIT'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Super Admin'), 'MODULE_NAME.DELETE'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Security Admin'), 'MODULE_NAME.VIEW'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Security Admin'), 'MODULE_NAME.CREATE'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Security Admin'), 'MODULE_NAME.EDIT'),
((SELECT ID FROM PLATFORM.ROLES WHERE NAME = 'Security Admin'), 'MODULE_NAME.DELETE');
```

**Permission naming:** `ENTITY_NAME.ACTION` with dot separator (e.g., `REBATE_GTN_MODELLING.VIEW`, `HEALTH_PLAN.EDIT`). A few legacy permissions exist without dots (e.g., `VBC_CONFIGURATION`, `VIEW_ALL_ORGANIZATIONS`) — do **not** follow that pattern for new permissions.

**Which roles?** Check with the user. Common patterns:

- **Super Admin** and **Security Admin** — almost always get all permissions
- **App-specific admin** (e.g., `Manufacturer RebateIQ Admin`) — gets permissions for that app
- Consult `app/role/constants.py` → `BuiltInSystemRoles` for the full role list

**After adding permissions:** Also add the new permission values to `BuiltInPermissions` enum in `app/role/constants.py`.

### 4.5 Add Index

```sql
CREATE INDEX IDX_TABLE_COLUMN ON {schema_name}.[TABLE_NAME](COLUMN_NAME);

-- With covering columns for query optimization
CREATE INDEX IDX_TABLE_COLUMN ON {schema_name}.[TABLE_NAME](COLUMN_NAME)
    INCLUDE (OTHER_COL1, OTHER_COL2);
```

### 4.6 Add Foreign Key

```sql
ALTER TABLE {schema_name}.[CHILD_TABLE]
ADD CONSTRAINT FK_CHILD_PARENT
FOREIGN KEY (PARENT_ID) REFERENCES {schema_name}.[PARENT_TABLE](ID);
```

For cascading deletes: `ON DELETE CASCADE`

### 4.8 Foreign Key Ordering in Migrations

When creating multiple tables with FK dependencies in a single migration, the **referenced (parent) table must be created before the referencing (child) table**. Plan the `CREATE TABLE` order based on the FK dependency graph.

```sql
-- CORRECT: IW_GAP created before IW_PAYER_REQUEST (which references it via GAP_ID)
CREATE TABLE {schema_name}.[IW_GAP] ( ... );
CREATE TABLE {schema_name}.[IW_PAYER_REQUEST] (
    ...
    GAP_ID INT NULL FOREIGN KEY REFERENCES {schema_name}.[IW_GAP](ID),
    ...
);

-- WRONG: Would fail because IW_GAP doesn't exist yet
CREATE TABLE {schema_name}.[IW_PAYER_REQUEST] (
    GAP_ID INT NULL FOREIGN KEY REFERENCES {schema_name}.[IW_GAP](ID)
);
CREATE TABLE {schema_name}.[IW_GAP] ( ... );
```

**Also applies to `tables.py`:** The table creation order in `get_tenant_tables_sql()` must respect the same FK dependency order.

### 4.7 Drop with Safety

```sql
DROP INDEX IF EXISTS IDX_OLD_INDEX ON {schema_name}.[TABLE_NAME];

IF EXISTS (SELECT * FROM sys.columns WHERE object_id = OBJECT_ID('{schema_name}.TABLE_NAME') AND name = 'OLD_COLUMN')
    ALTER TABLE {schema_name}.[TABLE_NAME] DROP COLUMN OLD_COLUMN;
```

---

## 5. The `tables.py` Sync Rule (CRITICAL)

When you create or modify a **tenant** table in a migration, you **MUST** also update `app/tenant/tables.py`.

### Why?

When a new tenant is provisioned, the system runs `get_tenant_tables_sql(schema_name)` from `tables.py` to create all tables in the new tenant schema. If `tables.py` doesn't include your new table, **new tenants will be missing it** — and this fails silently.

### What to Do

1. Open `app/tenant/tables.py`
2. Find the appropriate location (tables are roughly grouped by module)
3. Add your new table's `CREATE TABLE` and `CREATE INDEX` statements
4. **For column additions (ALTER TABLE in migration):** Do NOT add a separate `ALTER TABLE` in `tables.py`. Instead, **modify the original `CREATE TABLE` statement inline** to include the new column. `tables.py` represents the final schema for new tenants — it should read as clean `CREATE TABLE` DDL, not a series of migrations.
5. The SQL in `tables.py` uses `{schema_name}` f-string interpolation (same as migrations)

### Does NOT Apply To

- Platform tables (`PLATFORM.*`) — these are not per-tenant
- Index-only changes on existing tables (indexes are not always in `tables.py`, but new tables must be)

---

## 6. Post-Migration Checklist

After writing the migration, verify:

- [ ] File named correctly (`YYYY_N_SEQ_REV.sql`) with correct next sequence number
- [ ] Placed in `_migrations/` (not in `2025_and_older/`)
- [ ] Platform tables use `PLATFORM.TABLE_NAME`
- [ ] Tenant tables use `{schema_name}.TABLE_NAME`
- [ ] Section header comments present (`-- ========== PLATFORM MIGRATIONS ==========`)
- [ ] New tenant tables/columns also added to `app/tenant/tables.py`
- [ ] New permissions also added to `BuiltInPermissions` in `app/role/constants.py`
- [ ] New columns on existing tables default to `NULL`
- [ ] `DATETIME2` used (not `DATETIME`), `NVARCHAR` used for Unicode text
- [ ] `DECIMAL(18,4)` used for money/percentages (not `FLOAT`)
- [ ] Indexes created for foreign keys and frequently-queried columns

---

## 7. Don'ts

- **Don't edit applied migrations.** Always create a new file.
- **Don't skip the sequence number check.** Duplicate or gap sequences cause confusion.
- **Don't forget `tables.py`.** This is the #1 migration mistake — new tenants silently miss tables.
- **Don't use `FLOAT` for money.** Precision loss. Use `DECIMAL(18,4)`.
- **Don't add `NOT NULL` columns to existing tables without a default or backfill plan.**
- **Don't hardcode schema names.** Use `{schema_name}` for tenant, `PLATFORM.` for platform.
- **Don't create a separate PERMISSIONS table.** Permissions go directly into `ROLE_PERMISSION`.
