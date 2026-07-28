# LNTB Codex Context Files

## Documentation index

- `PHASE_1_PROGRESS.md` — current completion status and remaining release checks
- `PHASE_1_SCOPE.md` — authoritative Phase 1 boundaries
- `PHASE_1_MOBILE_IMPLEMENTATION.md` — current Flutter experience and contracts
- `API_CONTRACTS.md` — REST endpoints and payloads
- `AUTH_DEVICE_FLOW.md` — authentication, claim, sharing, and control flows
- `DATABASE_DESIGN.md` — lookup-driven relational schema
- `architecture/SYSTEM_ARCHITECTURE.md` — system responsibilities and boundaries

Recommended first Codex prompt:

```text
Read AGENTS.md and all Markdown files under docs/.

Do not modify code yet.

Summarize:
1. Phase 1 scope
2. Authentication behavior
3. Device-claim rules
4. Five-user sharing rule
5. Database conventions
6. API endpoints
7. Missing decisions

Then create an implementation plan.
```
