# AGENTS.md — Ararat

## PROJECT OVERVIEW

**Ararat** is a multi-tenant SaaS platform for managing Taekwondo gyms in the United States. Gyms are primarily run by Korean-American owners (관장님) serving diverse members (Korean, Hispanic, English-speaking families). The platform supports trilingual operation (English, Korean, Spanish).

### Applications

| App | Platform | Primary User | Description |
|-----|----------|--------------|-------------|
| **Parent/Member App** | Web (mobile-responsive) | Parents, adult members | Registration, activity feed, attendance history, payments, notifications |
| **Admin App** | Web | Gym owner, staff | Member management, billing, 심사 scheduling, reporting, newsletters |
| **Attendance Kiosk App** | iPad/Tablet | Children (self check-in) | Face recognition, QR code, and manual check-in at gym entrance |
| **Monitor App** | TV/Large Display | Staff, visitors | Live attendance board, schedule, announcements (optional) |

### Domain Terms

| Term | Meaning |
|------|---------|
| 관장님 (Gwanjangnim) | Gym owner/master — primary admin user |
| 도장 (Dojang) | Taekwondo gym/training hall |
| 심사 (Simsa) | Belt promotion examination |
| 띠 (Tti) | Belt — represents student rank |
| 수련생 (Suryeonsaeng) | Student/trainee |
| 사범 (Sabeom) | Instructor |
| 학부모 (Hakbumo) | Parent/guardian |
| 탈퇴 (Talhoe) | Membership withdrawal |
| 가정통신문 | Formal newsletter from gym to parents |
| 승품·단 | Official Kukkiwon belt promotion certification |

### Key Architecture Decisions

- **Multi-tenant**: Single deployment, data-isolated per gym
- **US market only**: Payments via Stripe (USD), SMS via Twilio, email via SendGrid
- **Trilingual**: i18n from Day 1 — English (default), Korean, Spanish
- **Face recognition**: On-device only (iPad). No biometric data in the cloud. COPPA/BIPA compliant.
- **Messenger integration**: Agnostic adapter pattern — KakaoTalk first, extensible to WhatsApp/LINE/etc.
- **API-first**: RESTful backend serving all four applications

---

## DOCUMENTATION SYSTEM

### Project Documents (`/docs`)

| Document | Purpose | Contains | Does NOT contain |
|----------|---------|----------|------------------|
| **PRD.md** | High-level product description & requirements | Business goals, user stories, feature scope, acceptance criteria | Technical implementation details |
| **TRD** | Technical details & implementation design — **living document** | `docs/trd/` directory with 21 section files, each with Implementation Notes. Index at `docs/trd/README.md`. ADRs in `docs/trd/adr/`. | Raw code dumps, implementation tracking/checklists |
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

On session start, check which documents exist:

```
1. Does docs/PRD.md exist?          → NO  → Start Phase: PRD
2. Does docs/trd/README.md exist?    → NO  → Start Phase: TRD
3. Does docs/imp/README.md exist?    → NO  → Start Phase: IMP
4. imp/README.md exists              → Read imp/README.md → Resume implementation
```

**Always announce the detected phase to the user:**
> "I see [PRD/TRD/IMP] exists but [next doc] is missing. We should work on [next doc] next."

#### New Project Flow

```
PRD → TRD → IMP → Implementation
```

1. **PRD Phase**: Gather requirements from the user. Create `docs/PRD.md` with business goals, user stories, feature scope, and acceptance criteria. PRD defines **what** and **why**.
2. **TRD Phase**: Translate PRD into technical specifications. Create `docs/trd/` directory with section files, index, and ADRs. TRD defines **how** — completely. After TRD, the PRD is never referenced during development.
3. **IMP Phase**: Create `docs/imp/` directory with phased implementation plan. Break TRD sections into development tasks, ordered by dependencies. Each task references its TRD section file. Index at `docs/imp/README.md`, one file per phase.
4. **Pre-Implementation Checkpoint**: Once PRD, TRD, and IMP are all finalized, ask the user: _"Project structure is defined. Should I tailor AGENTS.md to remove rules that don't apply to this project?"_ If confirmed, remove irrelevant sections and add a note: `<!-- Tailored on YYYY-MM-DD -->`.
5. **Implementation Phase**: Pick up tasks from `imp/README.md`, follow the Workflow below for each code change.

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
│  2. READ imp/README.md — find current phase & progress │
│  3. VERIFY — does IMP fully capture what TRD         │
│     describes? If GAP → ASK & CLARIFY first          │
│  4. Update TRD / IMP if changes are needed           │
│                                                      │
├──────────────────────────────────────────────────────┤
│          ⚠️  PLAN → CONFIRM → IMPLEMENT               │
│                                                      │
│  5. PRESENT PLAN to user — list the specific files    │
│     to change, what changes will be made, and why     │
│  6. WAIT for explicit user confirmation               │
│     • Do NOT start coding until user approves          │
│     • If user requests changes to plan, revise & re-ask│
│  7. IMPLEMENT code changes per approved plan           │
│                                                      │
├──────────────────────────────────────────────────────┤
│                 AFTER ANY CODE CHANGE                │
│                                                      │
│  8. UPDATE relevant IMP phase file — mark tasks done   │
│  9. UPDATE relevant TRD file — Implementation Notes   │
│  10. UPDATE PRD.md — if feature scope changed        │
│  11. UPDATE README  — if user-facing behavior changed │
│  12. CHECK all docs for consistency                   │
│                                                      │
├──────────────────────────────────────────────────────┤
│              ✅ VALIDATE → AUTO-COMMIT                 │
│                                                      │
│  13. RUN tests / linter / type-check                 │
│  14. VERIFY feature works as expected                 │
│  15. AUTO-COMMIT with descriptive message              │
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

### TRD Completeness Rule (CRITICAL)

The TRD is the **single source of truth** for implementation. During development, developers reference ONLY the TRD — never the PRD.

- **EVERY** business rule, flow, state machine, data model, and configurable behavior from the PRD must be captured as a technical specification in the TRD
- **EVERY** feature section in the TRD must contain enough detail to implement without cross-referencing any other document
- **NEVER** leave a requirement in PRD that is not reflected in TRD — if the PRD changes, update TRD first
- **ALWAYS** include: data model, validation rules, state transitions, error handling, edge cases, and configuration options for each feature
- If the TRD is missing information needed to implement a feature, **update the TRD first** before writing code
- The PRD defines **what and why**. The TRD defines **how** — completely.

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
- If the TRD's Implementation Notes are missing or stale, **update them before proceeding**

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

### Implementation Notes

> _This section will be updated as the feature is implemented._

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

1. **Section Map table** with these exact columns:

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

2. **ADR summary table**:

```markdown
| ADR | Decision | Affects |
|-----|----------|---------|
| [001](./adr/001-example.md) | Decision title | Section numbers affected |
```

3. **Glossary**: Domain-specific terms and acronyms

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
- [ ] **Task name** — `docs/trd/NN-section.md` — Brief scope description
```

#### imp/README.md Format (Index)

The index file is the entry point for IMP navigation. It MUST contain:

1. **Overall progress summary**:

```markdown
**Total Tasks**: NN | **Completed**: 0 | **Progress**: 0%
```

2. **Phase summary table**:

```markdown
| Phase | File | Status | Tasks | Progress |
|-------|------|--------|-------|----------|
| 0 | [Foundation](./phase-0-foundation.md) | Not Started | NN | 0% |
```

3. **Dependency notes** between phases

### Plan-and-Confirm Rule (CRITICAL)

- **NEVER** start writing or modifying code without presenting a plan first
- **ALWAYS** show the plan to the user: which files change, what changes, and why
- **ALWAYS** wait for explicit user approval ("yes" / "go ahead" / "approved") before touching code
- If the user rejects or modifies the plan, revise and re-present — do NOT proceed with the original
- Trivial typo or single-line fixes still require a brief plan ("I'll change X in file Y — ok?")

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
| After PRD created | Populate `## PROJECT OVERVIEW` — project description, applications, target users, domain terms, key constraints |
| After TRD created | Add key architecture decisions, tech stack summary, ADR highlights to PROJECT OVERVIEW |
| After IMP created | Update current phase note in NOTES section |
| During implementation | Update WHERE TO LOOK with actual paths, conventions, directory structure as they emerge |
| After features implemented | Refine PROJECT OVERVIEW with actual tech choices, deviations from original plan |
| After ADR created | Note relevant architectural constraints in PROJECT OVERVIEW |

**Rules:**
- Updates are **immediate** — not batched at end of session
- Only update sections relevant to the change — do not rewrite the entire file
- Preserve all existing rules and conventions — only add/update project-specific context
- If AGENTS.md conflicts with TRD, the TRD takes precedence — update AGENTS.md to match

---

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Product requirements | `docs/PRD.md` | What to build and why |
| Technical spec (index) | `docs/trd/README.md` | Start here — section map with dependencies |
| Implementation status (index) | `docs/imp/README.md` | Start here — phase summary and progress |
| IMP phase files | `docs/imp/phase-*.md` | Individual phase task lists |
| User/dev guide | `docs/README.md` | How to use the application |
| Reference materials | `docs/reference/` | Note.txt, Requirements.txt, Reference.txt |
| Agent knowledge | `AGENTS.md` | This file |
| TRD section files | `docs/trd/*.md` | Individual feature/system specs |
| Architecture decisions | `docs/trd/adr/` | ADR files with context, decision, consequences |


## NOTES

- Docs-first workflow: PRD → TRD → IMP → **Plan → User Confirm** → Code → Update Docs → Validate → Auto-Commit
- When picking up work, start by reading `imp/README.md` to find current state
- During implementation, reference `docs/trd/` only — never go back to PRD.md for technical details
- Use Mermaid for any diagrams (sequence, ER, flowchart)
- The plan-confirm step is non-negotiable — no exceptions, even for "quick" changes
- This project is currently in **IMP phase** — `docs/imp/` is being created
- One AGENTS.md at the project root is sufficient for most projects. For monorepos with truly independent apps (separate tech stacks, conventions, deployments), create an additional AGENTS.md in each app directory with app-specific overrides — the root AGENTS.md still holds shared rules
