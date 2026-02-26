## DOCUMENTATION SYSTEM

### Project Documents (`/docs`)

| Document | Purpose | Contains | Does NOT contain |
|----------|---------|----------|------------------|
| **PRD.md** | High-level product description & requirements | Business goals, user stories, feature scope, acceptance criteria | Technical implementation details |
| **TRD** | Technical details & implementation design — **living document** | `docs/trd/` directory with section files, each with Implementation Notes. Index at `docs/trd/README.md`. ADRs in `docs/trd/adr/`. | Raw code dumps, implementation tracking/checklists |
| **IMP.md** | Implementation plan & progress tracking | Step-by-step plans, task checkboxes, status, phase tracking | Product requirements, technical specs |
| **README.md** | End-user & developer usage guide | Setup, usage, configuration, API reference | Internal planning or tracking |


### App Documents (`/`)

| Document | Purpose | Contains | Does NOT contain |
|----------|---------|----------|------------------|
| **README.md** | End-user & developer usage guide | Setup, usage, configuration, API reference | Internal planning or tracking |


### Diagrams

All diagrams use **Mermaid** format embedded in markdown files. 

### Workflow (MANDATORY — every code change)

```
┌──────────────────────────────────────────────────────┐
│                 BEFORE ANY CODE CHANGE               │
│                                                      │
│  1. READ trd/README.md — find relevant TRD sections  │
│  2. READ IMP.md  — understand current progress       │
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
│  8. UPDATE IMP.md  — mark tasks done, add new ones   │
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

- **NEVER** implement without consulting `trd/README.md` and IMP.md first
- **NEVER** leave IMP.md out of sync after code changes
- **NEVER** dump raw code into TRD.md — describe logic in readable prose
- **ALWAYS** verify IMP.md captures everything in TRD.md before starting
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
- If the TRD’s Implementation Notes are missing or stale, **update them before proceeding**

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

---

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Product requirements | `docs/PRD.md` | What to build and why |
| Technical spec (index) | `docs/trd/README.md` | Start here — section map with dependencies |
| Implementation status | `docs/IMP.md` | What's done, what's next |
| User/dev guide | `docs/README.md` | How to use the application |
| Agent knowledge | `AGENTS.md` | This file |
| TRD section files | `docs/trd/*.md` | Individual feature/system specs |
| Architecture decisions | `docs/trd/adr/` | ADR files with context, decision, consequences |


## NOTES

- Docs-first workflow: PRD → TRD → IMP → **Plan → User Confirm** → Code → Update Docs → Validate → Auto-Commit
- When picking up work, start by reading IMP.md to find current state
- During implementation, reference `docs/trd/` only — never go back to PRD.md for technical details
- Use Mermaid for any diagrams (sequence, ER, flowchart)
- The plan-confirm step is non-negotiable — no exceptions, even for "quick" changes