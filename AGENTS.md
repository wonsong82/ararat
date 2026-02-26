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
- If the TRD's Implementation Notes are missing or stale, **update them before proceeding**

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

---

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Product requirements | `docs/PRD.md` | What to build and why |
| Technical spec (index) | `docs/trd/README.md` | Start here — section map with dependencies |
| Implementation status | `docs/IMP.md` | What's done, what's next |
| User/dev guide | `docs/README.md` | How to use the application |
| Reference materials | `docs/reference/` | Note.txt, Requirements.txt, Reference.txt |
| Agent knowledge | `AGENTS.md` | This file |
| Agent template | `TEMPLATE_AGENTS.md` | Generic template for all projects |
| TRD section files | `docs/trd/*.md` | Individual feature/system specs |
| Architecture decisions | `docs/trd/adr/` | ADR files with context, decision, consequences |


## NOTES

- Docs-first workflow: PRD → TRD → IMP → **Plan → User Confirm** → Code → Update Docs → Validate → Auto-Commit
- When picking up work, start by reading IMP.md to find current state
- During implementation, reference `docs/trd/` only — never go back to PRD.md for technical details
- Use Mermaid for any diagrams (sequence, ER, flowchart)
- The plan-confirm step is non-negotiable — no exceptions, even for "quick" changes
- This project is currently in **IMP phase** — IMP.md is being created
