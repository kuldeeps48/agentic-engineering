---
name: ai-service-repo-task-generator
description: >-
  Generates ordered, numbered task files for implementing features in the
  platform-ai (platform-ai) repo — a Python FastAPI microservice for AI/ML
  processing (RAG chat, document analysis, OCR, vector search, launch planning).
  Creates self-contained, dependency-aware implementation guides that
  an AI coding agent can follow file-by-file in separate chat sessions.
  Bakes in all AI service repo conventions so task files are immediately actionable.
  USE FOR: generate AI service tasks, AI implementation plan, AI task files,
  AI service handoff, platform-ai docs, platform-ai feature tasks, AI microservice tasks.
  DO NOT USE FOR: backend development (use api-patterns or new-module skill),
  database migrations (use migrations skill), frontend tasks (use frontend-repo-task-generator skill).
---

# AI Service Task Generator — platform-ai (platform-ai)

Generate **ordered, numbered task files** for implementing features in the `platform-ai` repo. Each task file is a self-contained guide with API contracts, database operations, AI/ML pipeline specs, event queue integration, and multi-tenant handling — ready for an AI coding agent to follow file-by-file.

## Target Repo: platform-ai (platform-ai)

| Aspect     | Detail                                                                                                            |
| ---------- | ----------------------------------------------------------------------------------------------------------------- |
| Type       | Internal microservice — consumed by platform-backend via REST API, not client-facing                                |
| Runtime    | Python 3.12, FastAPI, Uvicorn                                                                                     |
| AI/ML      | Azure OpenAI (GPT-5-nano), sentence-transformers (`all-MiniLM-L6-v2`, 384-dim), spaCy (NER/data masking), NLTK    |
| Databases  | Azure SQL Server (SQLAlchemy Core, pyodbc) for multi-tenant app data; PostgreSQL + pgvector for vector embeddings |
| OCR        | Tesseract + PyMuPDF with threaded processing                                                                      |
| Infra      | Dockerized, Azure Linux App Service, Terraform (sbx/dev/qa/beta/prod)                                             |
| Processing | Event-queue-driven — `AI_EVENT_QUEUE` table, 3-second polling loop, dispatch by `FEATURE_NAME`                    |
| Scheduling | APScheduler for daily jobs (e.g., AI dashboard insights)                                                          |
| Streaming  | SSE (Server-Sent Events) for RAG chat responses                                                                   |

### Architecture Overview

```
platform-backend → inserts row into AI_EVENT_QUEUE → platform-ai polls every 3s → dispatches by FEATURE_NAME
platform-backend → direct REST calls to platform-ai for chat, summaries, etc.
```

### Multi-Tenancy

- **Azure SQL:** Per-tenant schemas resolved at runtime (same pattern as backend). All access via SQLAlchemy Core (no ORM) with dynamic `Table` objects and factory functions.
- **PostgreSQL + pgvector:** Shared database for vector embeddings. Tenant isolation via `tenant_id` column filtering.

### Core Pipelines

| Pipeline            | Flow                                                                                                            |
| ------------------- | --------------------------------------------------------------------------------------------------------------- |
| Document Processing | OCR (Tesseract + PyMuPDF) → text extraction → vectorization (sentence-transformers, 384-dim) → pgvector storage |
| RAG Chat            | Query → pgvector L2 distance search → context assembly → Azure OpenAI GPT → SSE streaming response              |
| Document Analysis   | Extract text → Azure OpenAI → summarization / key terms / key sections / policy comparison                      |
| Launch Planning     | Gap detection → top-10 P&T answers → executive summary (AccessIQ-specific)                                      |
| Dashboard Insights  | APScheduler daily → cluster chat history across tenants → generate insights                                     |

### Feature Domains (FEATURE_NAME enum values)

- `IW_DOCUMENT_MANAGEMENT` — Intelligent Workspace document processing
- `REBATE_CONTRACT_MANAGEMENT` — Rebate contract analysis
- `REBATE_CONTRACT_ADDENDUM` — Rebate addendum processing
- `REVIEW_HEALTH_PLAN_POLICY` — Health plan policy review
- `REVIEW_PRIMARY_POLICY` — Primary policy review
- `GOLDEN_POLICY` — Golden policy comparison/generation

### Repo Structure

```
platform-ai/
  ai_services/          # AI logic organized by feature
    chat/               # RAG chat (streaming SSE)
    summary/            # Document summarization
    key_term/           # Key term extraction
    ocr/                # OCR pipeline (Tesseract + PyMuPDF)
    review/             # Policy review & comparison
    launch_planning/    # AccessIQ gap detection, P&T, exec summary
    ...
  database_operations/  # One file per table, each function manages its own DB session
  models.py             # All table definitions + tenant-scoped factory functions
  database_utils/       # Connection/engine/session setup
  _terraform/           # Per-environment Terraform configs
  _migrations/          # PostgreSQL migration scripts
```

### AI Service Conventions (Enforce in Every Task File)

- **No ORM.** All Azure SQL access uses SQLAlchemy Core with dynamic `Table` objects from factory functions. Never use declarative models.
- **One function per DB operation.** Each function in `database_operations/` manages its own session — no shared transaction context.
- **Event queue pattern.** New features triggered by the backend MUST integrate with `AI_EVENT_QUEUE` polling. Document the `FEATURE_NAME` enum value and expected queue payload.
- **Tenant schema resolution.** Azure SQL queries must resolve tenant schema at runtime. Show the factory function pattern in task files.
- **Vector operations use pgvector.** Embeddings are 384-dim (all-MiniLM-L6-v2). Use L2 distance for similarity search.
- **Streaming responses.** Chat endpoints use SSE — document the event format and streaming behavior.
- **Data masking.** spaCy NER is used to mask PII before sending to Azure OpenAI. New features handling user data must go through the masking pipeline.
- **Azure OpenAI calls.** Use the shared client/config. Document expected prompts, system messages, temperature, and token limits.
- **Feature gating.** The backend checks `enable_ai` in tenant app config before calling the AI service. The AI service itself does not check — but document the gating for completeness.

## Reference Pattern

Output follows the proven pattern in `documents/accessiq-launch-planning/frontend/` (adapted for AI service). Study that folder structure for the general format.

## Clarifying Questions

Before generating, clarify with the user:

1. **Feature scope?** Which feature domain? (e.g., "Launch Planning gap detection", "Rebate contract analysis")
2. **Source material?** Business docs, technical design docs, or "derive from backend code"? Check backend's `ai_service/` module for the integration surface.
3. **Output folder?** Where to write the files (e.g., `documents/rebate/ai-service/`)
4. **Event-driven or direct API?** Is this triggered via `AI_EVENT_QUEUE` polling, direct REST call, or both?
5. **New feature domain?** Does this require a new `FEATURE_NAME` enum value?

## Generation Process

### Step 1: Gather Backend Context

Read the backend's AI integration surface:

- **`app/ai_service/`** — Service functions that call the AI microservice (shows the REST API contract)
- **`app/ai_service/constants.py`** — `AIEventQueueFeatureName` enum, `is_ai_enabled_for_app()`
- **Controllers** that trigger AI features — shows what data the backend sends
- **Schemas** — request/response shapes between backend and AI service
- **`AI_EVENT_QUEUE` model** — queue payload structure
- **Business docs / mockups** — if provided, cross-reference for accuracy

Use the todo list to track which files you've read and what remains.

### Step 2: Plan the Task Breakdown

Create a dependency graph and implementation order:

1. **Foundation** — models.py updates (new tables/factory functions), database_operations files, FEATURE_NAME enum
2. **Data pipeline** — OCR/extraction/vectorization (if document processing is involved)
3. **Core AI logic** — prompt engineering, Azure OpenAI integration, response parsing
4. **API endpoints** — FastAPI routes consumed by the backend
5. **Event queue handler** — dispatcher integration for async processing
6. **Scheduled jobs** — APScheduler tasks (if applicable)
7. **Integration audit** — verify end-to-end flow from backend trigger to AI response

### Step 3: Generate Task Files

```
{output_folder}/
  00-implementation-plan.md    # Master index with dependency graph
  01-{foundation}.md           # Models, DB operations, config
  02-{pipeline}.md             # Data processing pipeline
  ...
  NN-{feature}.md              # One per feature/group
  AGENT-INSTRUCTIONS.md        # Copy-paste prompts for each pass
```

### Step 4: Generate AGENT-INSTRUCTIONS.md

Copy-paste-ready prompts for each implementation pass. Each pass:

- Lists which task files to attach
- Provides a detailed prompt with key implementation points
- Includes a reminder of AI service conventions
- Ends with "After implementing, list any assumptions you made"

Include these rules for every pass:

```
Rules for this pass:
- Use SQLAlchemy Core only (no ORM/declarative models)
- Each DB function manages its own session
- Resolve tenant schema at runtime via factory functions
- Vector embeddings are 384-dim (all-MiniLM-L6-v2), use L2 distance
- Mask PII with spaCy NER before sending to Azure OpenAI
- Use shared Azure OpenAI client/config
- SSE for streaming chat responses
- FEATURE_NAME must match AIEventQueueFeatureName enum in platform-backend
- Document expected AI_EVENT_QUEUE payload structure
```

## Task File Format

Every numbered task file MUST include:

````markdown
# NN — {Feature Title}

**[Requires: 01, 02, ...]**
**Feature Domain:** {FEATURE_NAME enum value}
**Trigger:** {Event queue | Direct REST | Scheduled job}
**Called by:** {Which backend endpoint/service calls this}

---

## Concept

{Brief explanation of what this AI feature does, its pipeline, and expected outcomes}

## Backend Integration Surface

### How the Backend Calls This

{HTTP method} /api/{path} (or: inserts into AI_EVENT_QUEUE with FEATURE_NAME={value})

**Request payload:**

```json
{exact request shape with example values}
```
````

**Response:** (status {code})

```json
{exact response shape with example values}
```

**Event queue payload (if applicable):**

```json
{
  "FEATURE_NAME": "{value}",
  "TENANT_ID": "...",
  "PAYLOAD": { ... }
}
```

## Database Operations

### {Table/Operation Name}

**Database:** {Azure SQL (tenant-scoped) | PostgreSQL (pgvector)}
**File:** `database_operations/{file}.py`

```python
# Function signature and key logic
def get_something(tenant_id: str, entity_id: int) -> dict:
    ...
```

**Schema:** {table columns, types, constraints}

## AI Pipeline

### Step 1: {Data Preparation}

- {Input processing, text extraction, PII masking}

### Step 2: {AI Processing}

- **Model:** {Azure OpenAI model name}
- **System prompt:** {system message template}
- **User prompt:** {user message template with variable placeholders}
- **Temperature:** {value}
- **Max tokens:** {value}

### Step 3: {Response Processing}

- {Post-processing, parsing, storage}

## API Endpoint (if direct REST)

```python
@router.post("/{path}")
async def handler(...):
    ...
```

**Streaming:** {Yes/No — if yes, document SSE event format}

## Event Queue Handler (if event-driven)

**FEATURE_NAME:** `{value}`
**Dispatcher function:** `{function name in ai_services/}`
**Processing steps:** {ordered list}
**Success behavior:** {what happens on completion — status update, callback, etc.}
**Failure behavior:** {error handling, retry logic, status update}

## Error Handling

- {AI timeout} → {behavior}
- {Empty/invalid response} → {behavior}
- {PII masking failure} → {behavior}
- {Vector store errors} → {behavior}

## Gotchas

- {Specific pitfalls for this feature}

```

### Content Rules

- **Backend integration is authoritative.** Read the backend's `ai_service/` module to understand exactly what it sends and expects back.
- **Document the full pipeline.** From trigger (event queue or REST) through data prep, AI processing, response handling, and storage.
- **Include prompt templates.** Show system prompts, user prompts, and variable placeholders. These are critical for AI features.
- **Specify model parameters.** Temperature, max tokens, model name — these affect output quality.
- **Document PII masking.** Which fields go through spaCy NER masking before Azure OpenAI calls.
- **Show DB operation signatures.** The consuming agent must know the exact function patterns for SQLAlchemy Core.
- **Map FEATURE_NAME values exactly.** Must match `AIEventQueueFeatureName` enum in the backend.
- **Call out gotchas.** Token limits, streaming edge cases, tenant isolation in pgvector, OCR quality issues.
- **Use real example values** — realistic healthcare/pharma data, not placeholders.

## Don'ts

- Don't assume the consuming agent knows anything about the backend
- Don't guess API contracts — extract from backend code
- Don't reference backend internal code (controllers, crud_service) — only expose the integration surface (what the backend sends to the AI service)
- Don't skip error handling — AI features fail in unique ways (timeouts, empty responses, token limits)
- Don't use placeholder data in examples — use realistic healthcare/pharma values
- Don't create circular dependencies in the task ordering
- Don't suggest SQLAlchemy ORM — this repo uses Core only
- Don't forget PII masking — all user-facing text must go through spaCy NER before Azure OpenAI
- Don't hardcode tenant schemas — always resolve at runtime

## Quality Checklist

Before finalizing, verify:

- [ ] Every backend→AI service integration point has a corresponding task file entry
- [ ] Dependency graph has no circular dependencies
- [ ] Implementation order respects all dependencies
- [ ] `00-implementation-plan.md` indexes every file with its dependencies
- [ ] `AGENT-INSTRUCTIONS.md` covers every task file in at least one pass
- [ ] `FEATURE_NAME` values match `AIEventQueueFeatureName` enum exactly
- [ ] Event queue payload structures match what the backend inserts
- [ ] AI pipeline steps are complete (data prep → AI call → response processing → storage)
- [ ] Prompt templates include all variable placeholders
- [ ] PII masking is documented for all paths that send data to Azure OpenAI
- [ ] Database operations use SQLAlchemy Core (no ORM)
- [ ] Tenant schema resolution is documented for Azure SQL operations
- [ ] Error handling covers AI-specific failure modes (timeouts, empty responses, token limits)
- [ ] Streaming behavior is documented for SSE endpoints
```
