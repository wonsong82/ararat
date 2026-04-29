# AGENTS.md

## PROJECT OVERVIEW

> **This section is auto-populated.** After the PRD is created, the agent updates this section with project description, applications, target users, and domain terms. After the TRD is created, key architecture decisions and tech stack are added. This placeholder is replaced entirely — do not edit manually.

_(No project defined yet. Start by creating a PRD.)_

---

## DOCUMENTATION SYSTEM

### Project Documents (`/docs`)

| Document | Purpose | Contains | Does NOT contain |
|----------|---------|----------|------------------|
| **PRD.md** | High-level product description & requirements | `**Status**: Draft/Confirmed` field, business goals, user stories, feature scope, acceptance criteria | Technical implementation details |
| **TRD** | Technical details & implementation design — **living document** | `docs/trd/` directory with section files, each with Implementation Notes. Index at `docs/trd/README.md`. ADRs in `docs/trd/adr/`. | Raw code dumps, implementation tracking/checklists |
| **IMP** | Implementation plan & progress tracking — **split by phase** | `docs/imp/` directory with phase files. Index at `docs/imp/README.md`. | Product requirements, technical specs |
| **README.md** | End-user & developer usage guide | Setup, usage, configuration, API reference | Internal planning or tracking |


### App Documents (`/`)

| Document | Purpose | Contains | Does NOT contain |
|----------|---------|----------|------------------|
| **README.md** | End-user & developer usage guide | Setup, usage, configuration, API reference | Internal planning or tracking |


### Diagrams

All diagrams use **Mermaid** format embedded in markdown files. 

### Project Lifecycle (CRITICAL — Read First)

Every project follows this lifecycle. The agent MUST detect the current phase and guide accordingly.

#### Phase Detection

On session start, check which documents exist and their status:

```
1. Does docs/PRD.md exist?          → NO  → Start Phase: PRD
   → YES, Status: Draft → Continue PRD (present to user for review)
   → YES, Status: Confirmed → PRD complete
2. Does docs/trd/README.md exist?    → NO  → Start Phase: TRD
   → YES, Status: Draft → Continue TRD (present to user for review)
   → YES, Status: Confirmed → TRD complete
3. Does docs/imp/README.md exist?    → NO  → Start Phase: IMP
   → YES, Status: Draft → Continue IMP (present to user for review)
   → YES, Status: Confirmed → IMP complete
4. All documents Status: Confirmed → Read imp/README.md → Resume implementation
```

**Always announce the detected phase to the user:**
> "I see [document] exists with Status: [Draft/Confirmed]. [Action needed]."

Examples:
- _"PRD.md exists but is still Draft. Let me present it for your review."_
- _"PRD is Confirmed but TRD is missing. We should work on the TRD next."_
- _"All docs are Confirmed. Resuming implementation from imp/README.md."_

#### Document Status (CRITICAL)

Every phase document includes a `**Status**` field on its first line (after the title). This field gates phase transitions — the agent MUST NOT advance to the next phase until the current document is Confirmed.

| Status | Meaning | Next action |
|--------|---------|-------------|
| `Draft` | Agent has created or is iterating on the document | Present to user for review |
| `Confirmed` | User has reviewed and approved the document | Proceed to next phase |

**Rules:**
- Documents are created with `**Status**: Draft`
- Only the **user** can trigger a transition to `Confirmed` — the agent NEVER self-confirms
- If the user requests changes to a Draft document, iterate and keep `Draft` until user approves
- A `Confirmed` document can be reopened (set back to `Draft`) if the user requests significant changes

#### New Project Flow

```
PRD → Review → TRD → Review → IMP → Review → Implementation
```

1. **PRD Phase**: Gather requirements from the user. Create `docs/PRD.md` with `**Status**: Draft`. PRD defines **what** and **why** — business goals, user stories, feature scope, and acceptance criteria.
2. **PRD Review**: Present the PRD to the user. Iterate until the user confirms. Mark `**Status**: Confirmed`. **Do NOT start TRD until PRD is Confirmed.**
3. **TRD Phase**: Translate confirmed PRD into technical specifications. Create `docs/trd/` directory with `**Status**: Draft` in `README.md`. TRD defines **how** — completely. All technology choices must be finalized with ADRs before moving to IMP. No "Option A vs Option B" — only decisions.
4. **TRD Review**: Present the TRD to the user. Iterate until the user confirms. Mark `**Status**: Confirmed`. After TRD is Confirmed, the PRD is never referenced during development. **Do NOT start IMP until TRD is Confirmed.**
5. **IMP Phase**: Create `docs/imp/` directory with `**Status**: Draft` in `README.md`. Break TRD sections into development tasks, ordered by dependencies. Each task references its TRD section file.
6. **IMP Review**: Present the IMP to the user. Iterate until the user confirms. Mark `**Status**: Confirmed`. **Do NOT start implementation until IMP is Confirmed.**
7. **Pre-Implementation Checkpoint**: Once PRD, TRD, and IMP are all Confirmed, ask the user: _"Project structure is defined. Should I tailor AGENTS.md to remove rules that don't apply to this project?"_ If confirmed, remove irrelevant sections and add a note: `<!-- Tailored on YYYY-MM-DD -->`.
8. **Implementation Phase**: Pick up tasks from `imp/README.md`, follow the Workflow below for each code change.

#### Adding a New Feature (Existing Project)

```
Update PRD → New TRD section → New ADR (if needed) → Append to IMP → Implement
```

1. **Update PRD.md** — add the new feature's requirements and acceptance criteria
2. **Create new TRD section** — `docs/trd/NN-new-feature.md` with full technical spec, cross-references, and Implementation Notes placeholder
3. **Update `trd/README.md`** — add the new section to the Section Map table
4. **Create ADR** — if the feature involves a technology choice, architectural pattern, or scope decision
5. **Append to relevant IMP phase file** — add new tasks referencing the new TRD section, update `imp/README.md` progress
6. **Implement** — follow the Workflow below


### Workflow (MANDATORY — every code change)

```
┌──────────────────────────────────────────────────────┐
│                 BEFORE ANY CODE CHANGE               │
│                                                      │
│  1. READ trd/README.md — find relevant TRD sections  │
│  2. READ imp/README.md — find current phase & tasks  │
│  3. VERIFY Implementation Notes — spot-check that     │
│     module paths and key files still exist. If stale  │
│     → update Implementation Notes before proceeding   │
│  4. VERIFY — does IMP fully capture what TRD         │
│     describes? If GAP → ASK & CLARIFY first          │
│  5. Update TRD / IMP if changes are needed           │
│  6. MARK the relevant IMP task as in-progress [~]     │
│                                                      │
├──────────────────────────────────────────────────────┤
│          ⚠️  PLAN → CONFIRM → IMPLEMENT               │
│                                                      │
│  7. PRESENT PLAN to user — list the specific files    │
│     to change, what changes will be made, and why     │
│  8. WAIT for explicit user confirmation               │
│     • Do NOT start coding until user approves          │
│     • If user requests changes to plan, revise & re-ask│
│  9. IMPLEMENT code changes per approved plan           │
│                                                      │
├──────────────────────────────────────────────────────┤
│                 AFTER ANY CODE CHANGE                │
│                                                      │
│  10. UPDATE relevant IMP phase file — mark tasks [x]  │
│  11. UPDATE relevant TRD file — Implementation Notes  │
│  12. UPDATE PRD.md — if feature scope changed        │
│  13. UPDATE README  — if user-facing behavior changed │
│  14. CHECK all docs for consistency                   │
│                                                      │
├──────────────────────────────────────────────────────┤
│              ✅ VALIDATE → AUTO-COMMIT                 │
│                                                      │
│  15. RUN tests / linter / type-check                 │
│  16. VERIFY feature works as expected                 │
│  17. AUTO-COMMIT with descriptive message              │
│      • Format: "feat|fix|refactor|docs: <description>" │
│      • Commit ONLY when tests pass & feature validated │
│      • One commit per logical feature / fix             │
└──────────────────────────────────────────────────────┘
```

### Documentation Rules

- **NEVER** implement without consulting `trd/README.md` and `imp/README.md` first
- **NEVER** leave IMP out of sync after code changes
- **NEVER** dump raw code into TRD.md — describe logic in readable prose
- **ALWAYS** verify IMP captures everything in TRD before starting
- **ALWAYS** clarify gaps between TRD and IMP before implementing
- **ALWAYS** verify Implementation Notes (spot-check file paths exist) before trusting them
- When modifying an exposed service contract (endpoint shape, response format), note the breaking change in the commit message with affected consumers
- When a consumed service changes its API, update the Service Dependencies table in the relevant TRD section BEFORE implementing against the new shape

### TRD Completeness Rule (CRITICAL)

The TRD is the **single source of truth** for implementation. During development, developers reference ONLY the TRD — never the PRD.

#### Content Completeness

- **EVERY** business rule, flow, state machine, data model, and configurable behavior from the PRD must be captured as a technical specification in the TRD
- **EVERY** feature section in the TRD must contain enough detail to implement without cross-referencing any other document
- **NEVER** leave a requirement in PRD that is not reflected in TRD — if the PRD changes, update TRD first
- **ALWAYS** include: data model, validation rules, state transitions, error handling, edge cases, and configuration options for each feature
- If the TRD is missing information needed to implement a feature, **update the TRD first** before writing code

#### All-Layers Coverage (CRITICAL — prevents backend-only blindspot)

The TRD must specify **every application and layer** listed in the PRD — not just the backend API.

- **EVERY application** defined in the PRD (web apps, native apps, display apps, etc.) must have its own architectural spec in the TRD — folder structure, tech choices, screen inventory, and patterns
- **Backend API** is not the product — it is ONE layer. The TRD must also fully specify: frontend architecture, frontend screens/flows, native app architecture, and any other client applications
- **Each feature TRD** must describe BOTH the backend implementation AND the frontend/client experience — API endpoints alone are not a complete spec
- **Frontend architecture** requires the same rigor as backend: folder structure, routing strategy, state management pattern, API client pattern, auth flow (token storage, refresh, protected routes), component patterns, styling approach, form handling, error handling
- **Native apps** (iOS, Android) require: architecture pattern (MVC/MVVM/SwiftUI), local storage strategy, sync protocol, offline behavior, platform-specific APIs used
- **Cross-cutting frontend concerns** must be specified: shared component library (if multiple web apps), design tokens, responsive breakpoints, accessibility standards, i18n integration on the client side
### IMP Completeness Rule (CRITICAL)

The IMP is the **complete task breakdown** of the TRD. It must achieve perfect 1:1 coverage — every TRD spec has a corresponding task, and every task traces back to a TRD spec.

#### Coverage Requirements

- **EVERY** feature, flow, state machine, cron job, webhook handler, API endpoint, business rule, and integration described in the TRD must have a corresponding IMP task
- **EVERY** IMP task must reference the TRD section it implements (e.g., `docs/trd/07-registration.md`)
- **NO net-new tasks** — if an IMP task describes something not specified in the TRD, either add the spec to the TRD first or remove the task. IMP implements TRD, it does not extend it.
- **Subtle specs count**: Cron jobs, scheduled tasks, seed data, admin CRUD for config entities, auto-generated content (certificates, summaries), deadline reminders, retention policies, and dashboard views are all tasks — not implied by other tasks
- **Cross-reference verification**: Before finalizing IMP, perform a section-by-section cross-reference of every TRD file against IMP tasks. Flag any TRD content without a corresponding task.

#### All-Layers Task Coverage (CRITICAL — prevents backend-only IMP)

- **EVERY application** in the PRD must have IMP tasks — backend API tasks alone do NOT constitute a complete IMP
- For each feature, the IMP must include SEPARATE tasks for: backend API/service, frontend screens/components, and any native app work
- A task like "member registration" is INCOMPLETE — it must be split into: "member registration API" + "member registration frontend screens" (or equivalent granularity)
- **Frontend scaffolding** (app setup, routing, shared components, auth flow, API client) must have its own IMP tasks in Phase 0 alongside backend scaffolding
- **Verification**: Before finalizing IMP, count tasks per application. If any PRD-listed application has zero or near-zero tasks, the IMP is incomplete

#### Phase Splitting

- Phases are split based on **dependency order** (Phase N+1 depends on Phase N) and **task volume**
- **Target**: 15–25 tasks per phase. If a phase exceeds 25 tasks, split it into sub-phases (e.g., Phase 1a, Phase 1b) or reorganize into more granular phases
- **Phase 0** is always foundation/infrastructure — no feature logic, just scaffolding
- **Each phase must be independently deliverable** — completing a phase should produce a working (if incomplete) system
- When splitting, respect dependency chains — never put a dependent task in an earlier phase than its prerequisite

#### Task Granularity

- Each task should be completable in **1–3 focused work sessions** (roughly 1–4 hours each)
- Tasks that are too broad ("Build the entire payment system") must be broken into atomic subtasks
- Tasks that are too narrow ("Add a single field to a table") should be merged with related tasks
- A good task has a **clear done condition**: "X endpoint returns Y", "cron job runs daily and triggers Z", "admin can CRUD W"
- The PRD defines **what and why**. The TRD defines **how** — completely.

#### Technical Decisions (Must Be Finalized Before IMP)

- **EVERY** technology choice must be finalized in the TRD with a corresponding ADR — no open options ("X or Y") are allowed. The IMP cannot be created until all technical decisions are locked.
- **Folder structure**: The TRD must define the project's directory layout — monorepo structure, app directories, shared packages, config file locations. Developers must know where to put new files without guessing.
- **Architecture patterns**: The TRD must specify architectural patterns used — module boundaries, dependency injection, service layer patterns, repository patterns, middleware chains, error handling strategy. Not just "use NestJS" but HOW to use it.
- **Coding conventions**: The TRD must capture coding standards — naming conventions (files, classes, functions, variables), import ordering, module export style, DTO/entity patterns, response formatting. These go in a dedicated TRD section or in the system architecture section.
- **Detailed tech stack**: Beyond framework names, the TRD must specify key libraries and their roles — ORM (e.g., TypeORM vs Prisma vs Drizzle), validation library, logging library, testing framework, migration tool, API documentation tool. Each choice requires an ADR if alternatives exist.
- **API conventions**: Request/response shapes, authentication flow, error format, pagination strategy, versioning approach — all must be specified with concrete examples, not just described abstractly.
- **Frontend libraries**: UI component library, state management, form handling, styling system, charting/visualization, date/time handling — each requires an ADR if alternatives exist. "Use React" is not enough — specify HOW React is used.

### TRD as Living Document (CRITICAL)

The TRD is NOT a static blueprint — it is a **living system map** that evolves as the application is built. Its purpose is to save developers (and AI agents) from reverse-engineering the codebase.

- **EVERY** TRD feature section includes an **Implementation Notes** subsection
- **AFTER** implementing a feature, update its Implementation Notes with:
  - **Module location**: which directories/files implement this feature
  - **Key files**: the most important files a developer should read
  - **Actual endpoints**: real API paths (not just conventions)
  - **Deviations from spec**: any changes made during implementation and why
  - **Edge cases discovered**: issues found during development not in original spec
  - **Configuration**: actual env vars, config files, or settings involved
- Implementation Notes are updated **immediately** after code changes — not batched
- If you need to understand how a feature works, read the TRD first — not the code
- If the TRD’s Implementation Notes are missing or stale, **update them before proceeding**

#### Implementation Notes Guidelines

- Implementation Notes describe **current state** — not change history
- When updating, **replace** previous values rather than appending
  - ✅ `Module location: src/auth/` (current state)
  - ❌ `Module location: was src/users/, moved to src/auth/ on 2026-02-15` (change log)
- If a deviation is resolved (spec updated to match code), **remove it** from deviations
- Target: Implementation Notes should stay **under 20 lines** per feature section
- Change history belongs in **git commits**, not in Implementation Notes
- If a feature is heavily refactored, rewrite Implementation Notes from scratch based on current code — do not patch old notes

### TRD Structure & File Formats

The TRD is organized as a directory of individual files — NOT a single monolith. This enables targeted reading and keeps files manageable as Implementation Notes grow.

#### Directory Structure

```
docs/
└── trd/
    ├── README.md                    # Index — Section Map, ADR summary, glossary
    ├── NN-short-name.md             # One file per TRD section
    └── adr/
        ├── README.md                # ADR index with template
        └── NNN-short-name.md        # One file per architecture decision
```

#### File Naming

- **TRD sections**: `NN-short-name.md` — zero-padded 2-digit number + kebab-case name (e.g., `01-system-architecture.md`, `07-registration.md`, `14-parent-feed.md`)
- **ADR files**: `NNN-short-name.md` — zero-padded 3-digit number + kebab-case name (e.g., `001-us-market-only.md`, `010-trd-living-document.md`)
- Numbers are sequential and never reused — deprecated files keep their number

#### TRD Section File Format

Every TRD section file follows this exact format:

```markdown
# N. Section Title

**Related TRDs**: [02-multi-tenancy](./02-multi-tenancy.md), [03-data-model](./03-data-model.md)  
**Related ADRs**: [ADR-001](./adr/001-us-market-only.md)  
**Phase**: MVP (Phase 1)

---

[Section content: specs, flows, data models, diagrams, business rules...]

### Service Dependencies

#### Services This Feature Consumes
| Service | Repo | Endpoint | Method | Request Shape | Response Shape |
|---------|------|----------|--------|---------------|----------------|
| _None_ | | | | | |

#### Contracts This Feature Exposes
| Endpoint | Method | Consumer(s) | Request Shape | Response Shape |
|----------|--------|-------------|---------------|----------------|
| _None_ | | | | | |

### Implementation Notes

> Last verified: _Not yet implemented_

- **Module location**: _TBD_
- **Key files**: _TBD_
- **Actual endpoints**: _TBD_
- **Deviations from spec**: _None yet_
- **Edge cases discovered**: _None yet_
- **Configuration**: _TBD_
```

**Cross-reference rules**:
- **Related TRDs**: Link to files this section depends on or closely interacts with
- **Related ADRs**: Link to architecture decisions that constrain this section's implementation
- **Phase**: Which development phase this section belongs to
- Use `_None_` when there are no related ADRs
- Use `All sections` for cross-cutting concerns (performance, infrastructure)

#### trd/README.md Format (Index)

The index file is the entry point for all TRD navigation. It MUST contain:

1. **Document status** (first line after title):

```markdown
**Status**: Draft | Confirmed
```

2. **Section Map table** with these exact columns:

```markdown
| # | File | Covers | Key Entities | Depends On | Phase |
|---|------|--------|-------------|------------|-------|
| 1 | [System Architecture](./01-system-architecture.md) | High-level arch, services | All services | — | MVP |
```

   - **#**: Section number
   - **File**: Linked filename
   - **Covers**: What this section specifies (brief)
   - **Key Entities**: Data models / domain objects defined here
   - **Depends On**: Other section numbers this section references
   - **Phase**: Development phase (MVP, Phase 2, etc.)

3. **ADR summary table**:

```markdown
| ADR | Decision | Affects |
|-----|----------|---------|
| [001](./adr/001-example.md) | Decision title | Section numbers affected |
```

4. **Glossary**: Domain-specific terms and acronyms

### TRD Navigation Protocol (AI Agents)

The TRD is split into individual files for targeted reading. **Never read all TRD files** — use the index to find what you need.

1. **Start**: Read `docs/trd/README.md` — the Section Map table tells you which files cover what
2. **Identify**: Find the file(s) relevant to your task using the "Covers" and "Key Entities" columns
3. **Check dependencies**: The "Depends On" column lists related files you may also need to read
4. **Check ADRs**: The "Related ADRs" column links to architectural constraints that apply to your task
5. **Read only what you need**: Open only the relevant TRD files — not the entire directory
6. **After implementing**: Update the `### Implementation Notes` section in the relevant TRD file(s)

### ADR Convention

Architecture Decision Records live in `docs/trd/adr/`. Each ADR captures:
- **Context**: What prompted the decision
- **Decision**: What was decided
- **Alternatives Considered**: What else was on the table
- **Consequences**: Positive and negative impacts

**Create a new ADR when**:
- Choosing a technology, framework, or service
- Changing feature scope (adding or removing features)
- Making architectural pattern decisions
- Deciding compliance strategies

**Format**: `NNN-short-name.md` (e.g., `001-us-market-only.md`). See `docs/trd/adr/README.md` for template.

#### ADR File Format

```markdown
# ADR-NNN: Title

**Status**: Proposed | Accepted | Deprecated | Superseded  
**Date**: YYYY-MM-DD  
**Deciders**: [who made this decision]

## Context
[What prompted this decision]

## Decision
[What was decided]

## Alternatives Considered
[What else was on the table]

## Consequences
[Positive and negative impacts]

**Affects**: [list of TRD sections impacted]
```

- Never reuse an ADR number — deprecated ADRs keep their number
- If superseded, add `Superseded by ADR-NNN` to status

### IMP Structure & File Formats

The IMP is organized as a directory of phase files — NOT a single monolith. This keeps each phase focused and manageable for AI context loading.

#### Directory Structure

```
docs/
└── imp/
    ├── README.md                    # Index — overall progress, phase summary, dependency notes
    └── phase-N-short-name.md        # One file per implementation phase
```

#### File Naming

- **IMP phase files**: `phase-N-short-name.md` — single digit + kebab-case name (e.g., `phase-0-foundation.md`, `phase-1-mvp-core.md`, `phase-2-enhanced.md`)
- Numbers are sequential starting from 0

#### IMP Phase File Format

Every IMP phase file follows this exact format:

```markdown
# Phase N — Phase Name

**Status**: Not Started | In Progress | Completed  
**Tasks**: NN | **Completed**: 0 | **Progress**: 0%

---

> Brief description of this phase's scope and prerequisites.

### N.1 Feature/Subsystem Name

- [ ] **Task name** — `docs/trd/NN-section.md` — Brief scope description
- [~] **In-progress task** — `docs/trd/NN-section.md` — Currently being worked on
- [x] **Completed task** — `docs/trd/NN-section.md` — Done and verified
```

**Task states**: `[ ]` = not started, `[~]` = in progress (picked up by an agent), `[x]` = completed

**Rules**:
- Mark a task `[~]` **before** starting work on it (Workflow step 6)
- Mark a task `[x]` **after** it is implemented, tested, and committed (Workflow step 10)
- Only ONE task should be `[~]` at a time — finish before starting the next
- If a session ends with a `[~]` task, the next session should check its status via `git status` and the task's TRD Implementation Notes

#### imp/README.md Format (Index)

The index file is the entry point for IMP navigation. It MUST contain:

1. **Document status** (first line after title):

```markdown
**Status**: Draft | Confirmed
```

2. **Overall progress summary**:

```markdown
**Total Tasks**: NN | **Completed**: 0 | **Progress**: 0%
```

3. **Phase summary table**:

```markdown
| Phase | File | Status | Tasks | Progress |
|-------|------|--------|-------|----------|
| 0 | [Foundation](./phase-0-foundation.md) | Not Started | NN | 0% |
```

4. **Dependency notes** between phases

### Plan-and-Confirm Rule (CRITICAL)

- **NEVER** start writing or modifying code without presenting a plan first
- **ALWAYS** show the plan to the user: which files change, what changes, and why
- **ALWAYS** wait for explicit user approval (“yes” / “go ahead” / “approved”) before touching code
- If the user rejects or modifies the plan, revise and re-present — do NOT proceed with the original
- Trivial typo or single-line fixes still require a brief plan (“I’ll change X in file Y — ok?”)

### Auto-Commit Rule

- **ALWAYS** commit automatically after a feature/fix is implemented, tested, and validated
- Commit message format: `feat|fix|refactor|docs|chore: <concise description>`
- One commit per logical unit of work (one feature, one bug fix, one refactor)
- **NEVER** commit if tests fail or the feature is not validated
- **NEVER** commit partial/broken work — if something is incomplete, do NOT commit
- **NEVER** batch multiple unrelated changes into a single commit

---

## GENERAL PRACTICES

### Code Quality

- No type suppression (`any`, `@ts-ignore`, `@ts-expect-error`)
- No empty catch blocks — handle or propagate errors
- No default exports — named exports only
- No business logic in UI components — extract to `lib/` or `services/`

### Environment & Secrets

- Never commit `.env` files — use `.env.example` as template
- Never hardcode credentials, API keys, or secrets

### Docker-First Development

- **Docker-first** for all backend services, databases, and infrastructure — use `docker-compose.yml` at project root
- Native client apps (iOS, Android, desktop) run on the host but connect to Dockerized services
- If the entire stack CAN run in Docker, it SHOULD — consistency across dev environments is the priority
- **NEVER** require developers to install databases, message queues, or other infrastructure locally
- Include a `Makefile` or npm/pnpm scripts for common Docker operations (`make up`, `make down`, `make logs`)
- `.env.example` must include all environment variables needed for Docker Compose

### Testing Conventions

- **EVERY** new feature or bug fix must include tests
- **BEFORE** writing code, check if a testing framework is already set up — match existing patterns
- **Test file location**: co-locate test files with source files (`foo.ts` → `foo.test.ts`) unless the project uses a separate `tests/` directory — follow whichever convention already exists
- **Naming**: test files mirror source file names with `.test.` or `.spec.` suffix
- **What to test**:
  - Unit tests: pure functions, business logic, utilities, validators
  - Integration tests: API endpoints, database queries, service interactions
  - E2E tests: critical user flows (only when explicitly required)
- **Test first when fixing bugs**: write a failing test that reproduces the bug, then fix it
- **NEVER** delete or skip failing tests to make the suite pass — fix the code or the test
- If no testing framework exists yet, **ask the user** which framework to use before setting one up

### Code Organization

- **BEFORE** creating new files, check existing project structure and follow the same patterns
- **NEVER** invent new organizational patterns when existing ones are established — consistency beats cleverness
- If the project has no clear structure yet, propose one and get user confirmation before creating files
- Business logic belongs in dedicated service/lib files — not in route handlers, UI components, or controllers
- Keep files focused: one module/class/concern per file. If a file exceeds ~300 lines, consider splitting
- Shared types, constants, and utilities go in common/shared directories — not duplicated across features


### Conventions

Conventions are captured **in AGENTS.md** — not in external style guides, wiki pages, or code comments. This ensures AI agents always have conventions loaded when they start a session.

#### What to Capture

| Category | Examples |
|----------|----------|
| **Naming** | File naming (`kebab-case.ts`), class/function/variable naming, database table/column naming |
| **Architecture patterns** | Where business logic lives, module boundaries, dependency direction |
| **API style** | REST conventions, error response shape, pagination format, versioning |
| **Frontend patterns** | Component structure, state management approach, styling methodology |
| **Code style** | Import ordering, export style (named only), max file length, comment conventions |
| **Git** | Branch naming, commit message format, PR conventions |
| **Testing** | What to test, naming conventions, mocking strategy, coverage expectations |

#### Where to Capture

- **Simple project** (single tech stack): All conventions go in the root `AGENTS.md` under a `## CONVENTIONS` section
- **Multi-stack project** (e.g., `/api` in Java, `/app` in React): Shared conventions (git, PR, cross-cutting rules) go in root `AGENTS.md`. Stack-specific conventions go in each app's own `AGENTS.md` (e.g., `/api/AGENTS.md`, `/app/AGENTS.md`)
- See **Hierarchical AGENTS.md** below for the full multi-app structure

#### Convention Format

Be **prescriptive**, not descriptive. AI agents follow instructions literally.

```markdown
## CONVENTIONS

### Naming
- Files: `kebab-case.ts` (e.g., `user-profile.service.ts`)
- Classes: `PascalCase` (e.g., `UserProfileService`)
- Functions/variables: `camelCase`
- Database tables: `snake_case`, plural (e.g., `user_profiles`)
- API endpoints: `kebab-case`, plural nouns (e.g., `/api/v1/user-profiles`)

### Architecture
- Business logic in `src/services/` — never in controllers or route handlers
- One service per domain entity
- Controllers only handle HTTP concerns (parsing, validation, response formatting)

### API
- Error response: `{ "error": { "code": "SNAKE_UPPER", "message": "Human-readable" } }`
- Pagination: cursor-based, `{ "data": [...], "cursor": { "next": "..." } }`
```

> **This section is customized per project.** Replace the example conventions above with your project's actual conventions. Delete categories that don't apply.

### Incremental Verification

- **VERIFY after each logical change** — do not accumulate multiple changes before checking
- After editing a file: run linter/type-check on that file immediately
- After completing a feature unit: run relevant tests before moving to the next unit
- After all changes: run full test suite + build before committing
- **If a verification fails**: fix it immediately before proceeding — do not stack more changes on top of broken code
- This prevents cascading errors that are exponentially harder to debug

### AGENTS.md as Living Document

AGENTS.md is a **living document** that evolves alongside the project. The agent auto-updates it at each lifecycle milestone — never requiring manual edits for project context.

| Trigger | What to update in AGENTS.md |
|---|---|
| After PRD confirmed | Populate `## PROJECT OVERVIEW` — project description, applications, target users, domain terms, key constraints |
| After TRD confirmed | Add key architecture decisions, tech stack summary, ADR highlights to PROJECT OVERVIEW |
| After IMP confirmed | Update current phase note in NOTES section |
| During implementation | Update WHERE TO LOOK with actual paths, conventions, directory structure as they emerge |
| After features implemented | Refine PROJECT OVERVIEW with actual tech choices, deviations from original plan |
| After ADR created | Note relevant architectural constraints in PROJECT OVERVIEW |

**Rules:**
- Updates are **immediate** — not batched at end of session
- Only update sections relevant to the change — do not rewrite the entire file
- Preserve all existing rules and conventions — only add/update project-specific context
- If AGENTS.md conflicts with TRD, the TRD takes precedence — update AGENTS.md to match

### Prompt Log (`docs/PROMPTS.md`)

The agent maintains a running log of user prompts and short answers in `docs/PROMPTS.md`.

**When to log**: After every user prompt. Skip trivial confirmations ("yes", "go ahead").

**Format**:

```markdown
# Prompt Log

Exact user prompts (EXACTLY and ENTIRELY) and short answers.

---

**Date**: YYYY-MM-DD HH:MM AM/PM TZ

**User**: [exact user prompt, verbatim]

**Answer**: [1-2 sentence summary of what was done or decided]
```

**Rules:**
- Record the user's prompt **exactly as written** — do not paraphrase or clean up
- Keep answers **ultra-short** — 1-2 sentences max, capturing the outcome not the process
- Include **date and time** with timezone for every entry
- **Prepend** new entries below the header — most recent prompt always appears first. If you are adding multipe rows at once, make sure they are sorted accordingly.
- This file is **prepend-only** — previous entries are never modified or reordered

---

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Product requirements | `docs/PRD.md` | What to build and why |
| Technical spec (index) | `docs/trd/README.md` | Start here — section map with dependencies |
| Implementation status (index) | `docs/imp/README.md` | Start here — phase summary and progress |
| IMP phase files | `docs/imp/phase-*.md` | Individual phase task lists |
| User/dev guide | `docs/README.md` | How to use the application |
| Agent knowledge | `AGENTS.md` | This file |
| TRD section files | `docs/trd/*.md` | Individual feature/system specs |
| Architecture decisions | `docs/trd/adr/` | ADR files with context, decision, consequences |


## NOTES

- Docs-first workflow: PRD → TRD → IMP → **Plan → User Confirm** → Code → Update Docs → Validate → Auto-Commit
- When picking up work, start by reading `imp/README.md` to find current state
- During implementation, reference `docs/trd/` only — never go back to PRD.md for technical details
- Use Mermaid for any diagrams (sequence, ER, flowchart)
- The plan-confirm step is non-negotiable — no exceptions, even for "quick" changes
- One AGENTS.md at the project root is sufficient for most projects. For monorepos, see **Hierarchical AGENTS.md** below.

### Hierarchical AGENTS.md (Multi-App Projects)

One AGENTS.md at the project root is sufficient for **most projects**. For monorepos with truly independent apps (separate tech stacks, conventions, deployments), use a hierarchical structure:

```
project-root/
├── AGENTS.md              # Shared: project overview, lifecycle, workflow, shared conventions
├── api/
│   └── AGENTS.md          # API-specific: Java conventions, architecture patterns, testing approach
├── app/
│   └── AGENTS.md          # App-specific: React conventions, component patterns, styling approach
└── docs/                  # Shared docs — referenced by root AGENTS.md
    ├── PRD.md
    ├── trd/
    └── imp/
```

#### What Goes Where

| Content | Root AGENTS.md | App AGENTS.md |
|---------|----------------|---------------|
| Project overview & lifecycle | ✅ | ❌ |
| Documentation system (PRD/TRD/IMP) | ✅ | ❌ |
| Workflow & plan-confirm rules | ✅ | ❌ |
| Shared conventions (git, PR, cross-cutting) | ✅ | ❌ |
| Tech-stack-specific conventions | ❌ | ✅ |
| App-specific architecture patterns | ❌ | ✅ |
| App-specific folder structure | ❌ | ✅ |
| App-specific testing approach | ❌ | ✅ |
| WHERE TO LOOK (app-level paths) | ❌ | ✅ |

#### App AGENTS.md Format

App-level AGENTS.md files are **focused and short** — typically 30-80 lines. They cover only what differs from the root.

```markdown
# AGENTS.md — [App Name]

> This file contains app-specific conventions. For project-level workflow, documentation system, and shared rules, see the root [AGENTS.md](../AGENTS.md).

## CONVENTIONS

[App-specific conventions: naming, architecture, API style, etc.]

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| App entry point | `src/main.ts` | ... |
| Routes | `src/routes/` | ... |
```

> **Keep app AGENTS.md self-contained for its domain.** An AI agent working in `/api` should not need to read `/app/AGENTS.md`. Cross-app concerns belong in the root AGENTS.md.

#### AI Agent Behavior: Subfolder AGENTS.md (CRITICAL)

When working in a subdirectory that has its own `AGENTS.md`, the agent MUST read it:

```
1. ALWAYS read the root AGENTS.md on session start (project-level context)
2. BEFORE working in a subdirectory, check: does it have its own AGENTS.md?
   → YES → Read it. App-specific conventions OVERRIDE root conventions where they conflict.
   → NO  → Use root AGENTS.md conventions only.
3. When switching between apps (e.g., /api → /app), read the new app's AGENTS.md
4. If root and app AGENTS.md conflict, the app AGENTS.md wins for that app's code
```

This ensures agents always have the right conventions loaded — Java conventions when writing Java, React conventions when writing React — without being overwhelmed by irrelevant context.