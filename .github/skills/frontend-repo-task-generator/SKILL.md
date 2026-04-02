---
name: frontend-repo-task-generator
description: >-
  Generates ordered, numbered task files for implementing features in the
  platform-frontend repo (React 18 + TypeScript + Redux Toolkit + Vite).
  Creates self-contained, dependency-aware implementation guides that
  an AI coding agent can follow file-by-file in separate chat sessions.
  Bakes in all frontend repo conventions so task files are immediately actionable.
  USE FOR: generate frontend tasks, frontend implementation plan, frontend task
  files, frontend handoff, React implementation docs, frontend feature tasks.
  DO NOT USE FOR: backend development (use api-patterns or new-module skill),
  database migrations (use migrations skill), AI service tasks (use ai-service-repo-task-generator skill).
---

# Frontend Task Generator — platform-frontend

Generate **ordered, numbered task files** for implementing features in the `platform-frontend` repo. Each task file is a self-contained guide with API contracts, component specs, Redux slices, routing, permissions, and UI behavior — ready for an AI coding agent to follow file-by-file.

## Target Repo: platform-frontend

| Aspect    | Detail                                                                                                          |
| --------- | --------------------------------------------------------------------------------------------------------------- |
| Framework | React 18 + TypeScript 5.x                                                                                       |
| Build     | Vite 6, tests via Vitest + React Testing Library                                                                |
| State     | Redux Toolkit (synchronous reducers only — no thunks/RTK Query)                                                 |
| Routing   | React Router v6, lazy loading via `lazyImport` + named exports                                                  |
| Styling   | Global SCSS + Bootstrap 5 (`react-bootstrap`)                                                                   |
| HTTP      | Axios shared instance, interceptor strips `response.data` and auto-toasts errors                                |
| Auth      | Auth0 / Microsoft Entra ID, JWT (`jwt-decode`), token expiry checked on navigation                              |
| Forms     | `react-hook-form`                                                                                               |
| i18n      | Custom implementation, single locale (`en-US`), flat `SCREAMING_SNAKE_CASE` keys                                |
| RBAC      | Permission strings (`ENTITY.ACTION`) via `useIsAbleTo` hook + `<ProtectedRoute>`                                |
| Key libs  | D3 (charts), DOMPurify, dayjs, PostHog, PowerBI embedded, react-beautiful-dnd, react-pdf-viewer, react-markdown |
| Imports   | Absolute via `baseUrl: "src"` in tsconfig                                                                       |
| Node      | >=20                                                                                                            |
| Lint      | ESLint + Prettier, Husky pre-commit (`lint-staged`)                                                             |

### Feature Module Pattern

Every feature lives under `src/features/<name>/` with this structure:

```
src/features/<name>/
  api/           # Axios calls (one function per endpoint)
  components/    # React components (one per file, named export)
  hooks/         # Custom hooks (useXxx pattern)
  routes/        # Route definitions + lazy-loaded page components
  types/         # TypeScript interfaces and enums
  index.ts       # Barrel export
```

### Frontend Conventions (Enforce in Every Task File)

- **Components:** Functional components with named exports (not default). One component per file.
- **API functions:** One file per resource in `api/`, return typed promises. Axios interceptor already strips `.data` — don't double-unwrap.
- **Redux slices:** Synchronous reducers only. 13 existing slices — add new slices sparingly; prefer local component state for feature-scoped data.
- **Routing:** Use `lazyImport(() => import("..."), "ComponentName")` for code splitting. Register in feature's `routes/` then wire into app router.
- **Permissions:** Gate with `useIsAbleTo("ENTITY.ACTION")` in components, `<ProtectedRoute requiredPermissions={[...]}/>` in routes. Permission strings match backend `BuiltInPermissions` enum values.
- **i18n keys:** `SCREAMING_SNAKE_CASE`, flat namespace. Add to locale file, reference via `t("KEY")`.
- **Forms:** Use `react-hook-form` with `useForm<FormType>()`. Validation via resolver or inline rules.
- **Dates:** Use `dayjs` — never raw `Date` or `moment`.
- **Sanitization:** Use `DOMPurify.sanitize()` for any user-generated HTML content.
- **Three user types:** COEUS (platform operator), Manufacturer (customer), Payer (participant). UI components must gate visibility based on user type and permissions.

## Reference Pattern

Output follows the proven pattern in `documents/accessiq-launch-planning/frontend/`. Study that folder structure — it is the gold standard.

## Clarifying Questions

Before generating, clarify with the user:

1. **Feature scope?** Which module/app area? (e.g., "AccessIQ Launch Planning", "RebateIQ Contract Management")
2. **Source material?** Business docs, technical design docs, mockups, or "derive from backend code"? Note: only reference HTML mockups from `mockups/` if the user explicitly provides them.
3. **Output folder?** Where to write the files (e.g., `documents/rebate/frontend/`)
4. **Sub-app?** ValueIQ (VBC), RebateIQ, or AccessIQ?

## Generation Process

### Step 1: Gather Backend Context

Read **all** relevant backend source files to build a complete API surface:

- **Controller(s):** Every endpoint — method, path, decorators, permissions, rate limits, query params
- **Schemas:** Request and response shapes — every field, type, optionality, validation
- **Models:** DB structure — relationships, hybrid properties, computed fields (informs what the API returns)
- **Constants:** Enums, status values, permission names
- **Business docs / mockups:** If provided, cross-reference with backend code for accuracy

Use the todo list to track which files you've read and what remains.

### Step 2: Plan the Task Breakdown

Create a dependency graph and implementation order:

1. **Foundation** — TypeScript types, enums, constants, API client functions
2. **Redux slice** (if needed) — state shape, reducers, selectors
3. **Core CRUD** — primary entity list/detail/create/edit pages
4. **Child/nested entities** — things that depend on core entities
5. **Cross-cutting** — AI integration, notifications, workflows, charts
6. **Dashboards/aggregations** — depend on everything else
7. **Integration audit** — final pass: routing, navigation, permissions, i18n

Group related small features into single task files. Keep large features in their own file.

### Step 3: Generate Task Files

```
{output_folder}/
  00-implementation-plan.md    # Master index with dependency graph
  01-{foundation}.md           # Types, enums, API functions
  02-{redux-slice}.md          # State management (if needed)
  ...
  NN-{feature}.md              # One per feature/group
  AGENT-INSTRUCTIONS.md        # Copy-paste prompts for each pass
```

### Step 4: Generate AGENT-INSTRUCTIONS.md

Copy-paste-ready prompts for each implementation pass. Each pass:

- Lists which task files to attach
- Provides a detailed prompt with key implementation points
- Includes a reminder of frontend conventions (permissions, i18n, forms, routing)
- Ends with "After implementing, list any assumptions you made"

Include these rules for every pass:

```
Rules for this pass:
- Use absolute imports (baseUrl: "src")
- Named exports only (no default exports)
- Permission strings must match backend BuiltInPermissions exactly
- Use react-hook-form for all forms
- Use dayjs for all date handling
- Use DOMPurify.sanitize() for any HTML content
- Axios interceptor strips .data — don't double-unwrap
- Add i18n keys as SCREAMING_SNAKE_CASE
- Gate UI by user type (COEUS/Manufacturer/Payer) and permissions
```

## Task File Format

Every numbered task file MUST include:

````markdown
# NN — {Feature Title}

**[Requires: 01, 02, ...]**
**Sub-app:** {ValueIQ | RebateIQ | AccessIQ}
**Access:** {Who can access — e.g., "Manufacturer + COEUS only. Payer sees nothing."}
**Permissions:** {e.g., `VBC_READ`, `VBC_WRITE` — exact backend enum values}
**Reload:** {Which reload counters / refetches to trigger after mutations}

---

## Concept

{Brief explanation of what this feature does and its lifecycle}

## API Endpoints

### {Action Name}

{HTTP method} /v1/{path}

**Headers:** X-Tenant-Id, X-Organization-Id, X-Tenant-App-Id, Authorization: Bearer {token}

{Request body as JSON with example values}

**Response:** (status {code})

```json
{exact response shape with example values}
```
````

**Query params:** {if applicable — name, type, default}

**Business rules:**

- {Rule 1}
- {Rule 2}

**Error cases:**

- {condition} → {status code} "{message}"

## TypeScript Types

```typescript
// types/{feature}.ts
export interface {EntityName} {
  // All fields with types, matching API response
}

export enum {EnumName} {
  // Values matching backend constants
}
```

## API Functions

```typescript
// api/{feature}.ts
export const get{Entities} = (params: {...}): Promise<{Type}> =>
  axios.get("/v1/...", { params });
```

## Components

### {ComponentName}

- **File:** `components/{ComponentName}.tsx`
- **Props:** {prop types}
- **Permissions:** {gate with useIsAbleTo("...")}
- **Behavior:** {what it renders, user interactions, state}

## Route Registration

```typescript
// routes/index.tsx
const { Page } = lazyImport(() => import("..."), "{Page}");
// Path: /app/{sub-app}/{feature}/...
```

## i18n Keys

```
{FEATURE}_{LABEL}: "Display text"
{FEATURE}_{BUTTON}: "Button text"
```

## UI Behavior

- {Loading states}
- {Empty states}
- {Error states}
- {User type gating — what COEUS/Manufacturer/Payer each see}
- {Form validation rules}
- {Navigation flows}

```

### Content Rules

- **API contracts are authoritative.** Extract exact paths, methods, query params, request bodies, and response shapes from the backend code. Do not guess.
- **Include every field.** For request/response schemas, list every field with its type, optionality, and any validation constraints.
- **Document error cases explicitly.** Every endpoint should list what returns 400, 403, 404, 409, etc.
- **Specify access control per endpoint.** Which user types (COEUS, Manufacturer, Payer) can call it? What happens if the wrong type calls it?
- **Map backend types to TypeScript.** `str` → `string`, `int` → `number`, `Decimal` → `number`, `datetime` → `string` (ISO 8601), `Optional[X]` → `X | null`, `List[X]` → `X[]`.
- **Include the Axios function signature.** Show exact API call with typed return — the consuming agent shouldn't have to guess the HTTP client pattern.
- **Show component file paths.** The consuming agent must know where to create each file.
- **Call out gotchas.** Enum quirks, sequential enforcement, Axios interceptor behavior, permission string format.
- **Use real example values** in JSON bodies — realistic healthcare/pharma data, not placeholders.

## Don'ts

- Don't assume the consuming agent knows anything about the backend
- Don't guess API contracts — extract from backend code
- Don't reference internal backend files (models, crud_service) — only expose the HTTP API surface
- Don't skip error cases — every endpoint needs 400/403/404/409 documentation
- Don't use placeholder data in JSON examples — use realistic healthcare/pharma values
- Don't create circular dependencies in the task ordering
- Don't mix unrelated features in one task file just to reduce file count
- Don't suggest default exports — this repo uses named exports only
- Don't suggest RTK Query or thunks — this repo uses synchronous Redux reducers only
- Don't suggest Jest — this repo uses Vitest

## Quality Checklist

Before finalizing, verify:

- [ ] Every backend endpoint for the feature has a corresponding task file entry
- [ ] Dependency graph has no circular dependencies
- [ ] Implementation order respects all dependencies
- [ ] `00-implementation-plan.md` indexes every file with its dependencies
- [ ] `AGENT-INSTRUCTIONS.md` covers every task file in at least one pass
- [ ] Permission strings match `BuiltInPermissions` enum values exactly
- [ ] Enum values match backend `constants.py` exactly
- [ ] Response schemas match what the backend actually returns (check `schema.py`)
- [ ] The three user types (COEUS, Manufacturer, Payer) are correctly mapped per endpoint
- [ ] TypeScript interfaces cover every field in the API response
- [ ] API function signatures use correct Axios patterns (no `.data` unwrap)
- [ ] Route paths follow existing app conventions
- [ ] i18n keys are `SCREAMING_SNAKE_CASE`
- [ ] Forms use `react-hook-form` (not controlled inputs)
```
