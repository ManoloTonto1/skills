# Mode: design

Design a feature with an agent team. Nobody edits code in this mode.

## 1. Understand the ask

- A GitHub issue: read it with its comments. Otherwise take the feature idea from the input and the conversation.
- Read what already exists in `docs/` and `docs/planning/`, and load earlier decisions from claude-mem.
- Ask Manuel the product and domain questions the code can't answer as the design takes shape, step by step when he says so. Use short lettered options with a recommendation for real decisions. When he names an existing flow as the standard, follow it. Decide implementation questions yourself, and don't ask what the code or memory can answer.

## 2. Spawn the team

`techlead`, `advocate`, plus `backend`, `frontend` and `data` for the layers the feature touches, `security` when it exposes anything new, and `qa` for the test plan. Tell every teammate this is a **read-only design phase: no file edits**.

Assign one research task per engineer, for example:

- `backend`: find the existing packages, server actions, route handlers and integrations this builds on or would duplicate, including an earlier implementation in Manuel's other repos when he points at one. Propose the server actions and route handlers, the auth checks and the error cases, and the simplest reliable route (what gets added and what gets removed).
- `data`: find the existing tables, RLS policies, RPCs (or Payload collections) and the real states of the data. Propose the schema, the policies, the migration sequence (expand, then contract) and how existing rows are handled, with the data-loss risk.
- `security`: threat-model the feature: what becomes reachable, by whom, and the controls the design needs (auth, access control, storage privacy, webhook verification, personal data).
- `frontend`: find the existing screens, design-system primitives, components and server actions to reuse, and read DESIGN.md and PRODUCT.md. Shape the UI with the `impeccable` skill (`shape`) and, where the repo has it, the `apple-design` skill. Propose the UI flow, the states (loading, empty with a CTA, error), interactions and motion, and the contract it needs from the backend.
- `qa`: walk through the current app with the Playwright MCP to see where the feature fits and what could break, then propose how each acceptance criterion will be proven: which E2E specs per role, the edge cases they cover, the seed data they need, and the integration and unit tests behind them. Push back on criteria that can't be tested.
- `advocate`: attack the problem statement and each proposal as it arrives: simpler alternatives, existing features that already do this, production dependencies the change would break, existing users and records and what happens to a specific existing account, the deploy order, edge cases, partial failure, the manual cleanup a design would leave, and official docs for any third-party behaviour the design relies on.

Have the engineers and `advocate` message each other directly. The design converges through argument, not through you relaying.

## 3. Assemble the design

Merge the findings into one design doc written to the repo at `docs/planning/NN-<slug>.md` (the next free number; `docs/` if there is no planning folder). Writing it is local work and needs no approval.

1. Problem and goal
2. Scope and explicitly out of scope
3. Design: data model, backend flow, server actions and route handlers, frontend flow, each naming the existing code it reuses (`path`)
4. Error handling and boundary validation
5. Production safety: existing users and records affected, the migration sequence (additive, then app, then backfill, as separate PRs), deploy order, rollback
6. Test plan: which test proves each acceptance criterion
7. Rejected alternatives and why (including the advocate's objections that were settled)
8. Open questions

Then derive the **ticket breakdown**: small, independently shippable tickets, each with a user story line, the goal of the ticket, testable acceptance criteria, the files or areas involved, dependencies between tickets, and one PR per ticket. No ticket that only adds something "for later".

## 4. Gate

Send the design and the tickets to `techlead` (checks for dead abstractions, duplicated capabilities, missing validation, missing or wrongly mocked tests, stacked retries, data-loss risk in migrations, ownership overlaps), to `security` for the threat model, and to `advocate` for a final pass. Iterate until `techlead` signs off, `security` has no open findings, and `advocate` says `NO OBJECTIONS`, or an objection is written up in Open questions for Manuel to decide.

## 5. Stop for Manuel, then publish

Show Manuel the design doc and the tickets, then stop. Design always waits for his confirmation. Apply his edits and show the result again if he changes anything.

After he approves:

1. Link the doc from the repo's CLAUDE.md or `docs/planning/STATUS.md` where that is the repo's convention.
2. When he asks for tickets, create them as GitHub issues (`gh issue create`, one per ticket; the body holds the user story, goal, acceptance criteria and `Depends on #N`).
3. Report the doc path and the issue URLs.
4. If his approval says to start ("go", a /goal), hand the first ticket to build mode.

Then shut the team down.
