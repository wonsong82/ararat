# ADR-010: TRD as Living Document

**Status**: Accepted  
**Date**: 2025-02-25  
**Deciders**: Project Owner

## Context

Traditional technical documents become stale immediately after implementation begins. Developers stop reading them and reverse-engineer the codebase instead. AI agents waste context window reading the entire codebase when the information should be readily available in documentation.

## Decision

The TRD is a living document that evolves as the application is built. Every TRD section includes an "Implementation Notes" subsection that is updated immediately after code changes. Implementation Notes capture: module location, key files, actual endpoints, deviations from spec, edge cases discovered, and configuration. The TRD is the first place a developer (or AI agent) should look — not the code.

Additionally, the TRD is split into individual files (one per section) in a `docs/trd/` directory to enable targeted reading. Each file includes cross-references to related TRDs and ADRs.

## Alternatives Considered

1. **Static TRD, read code for implementation details**: Standard practice, but wastes developer time and AI context. Rejected.
2. **Auto-generated docs from code (JSDoc, Swagger)**: Good for API reference but doesn't capture design decisions, flows, or business logic. Rejected as sole solution — can complement TRD.
3. **Wiki-based documentation**: Harder to version control, often becomes disorganized. Rejected.
4. **Living TRD with Implementation Notes (chosen)**: Keeps spec and reality in sync, AI-friendly, version-controlled. Accepted.

## Consequences

**Positive**:
- Developers and AI agents find implementation details immediately
- No reverse-engineering needed
- TRD stays accurate as the system evolves
- Split files enable targeted reading (smaller context window for AI)
- Cross-references guide navigation

**Negative**:
- Requires discipline to update after every code change
- Implementation Notes must be kept in sync (can become stale if neglected)
- More files to manage (21 TRD files + ADRs vs 1 monolith)

**Affects**: All TRD sections, AGENTS.md, development workflow
