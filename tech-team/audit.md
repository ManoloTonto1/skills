# Mode: audit

Audit an area of the code or the production errors in Sentry, report, and stop. Nobody edits code in this mode.

## 1. Scope

- Name the area: Supabase usage, a screen or flow, security, performance, or production errors in Sentry.
- Load the matching skill or tool where the repo has it: `supabase` and `supabase-postgres-best-practices`, `impeccable` (`audit`), `apple-design`, the Next.js skills, or the Sentry MCP (unresolved issues in the project, recent first; org and project from the repo's Sentry config). If a skill isn't installed, say so and audit without it.
- Load prior context (claude-mem, `docs/planning/`, open GitHub issues) so known issues aren't reported as new.

## 2. Spawn

`techlead`, `advocate`, plus the engineers and `qa` for the layers involved. `security` owns the security angle (auth, access control and RLS, storage, secrets, personal data) and `data` owns the data angle (schema, migrations history, integrity, orphans and duplicates in the real data). Tell every teammate this is a **read-only audit: no file edits**.

- Each teammate takes one angle (for example security, data integrity, queries and performance, the auth flow, UI states) and returns findings with `path:line` or the Sentry short ID, plus the evidence.
- `advocate` tries to refute every finding; refuted ones are dropped.
- Every Sentry issue gets a verdict: real bug, third-party noise, transient infra, already fixed, or regressed.

## 3. Report and stop

One prioritized report for Manuel:

- Each finding: severity (high, medium, low), effort, location, evidence, proposed fix.
- For Sentry: short ID, title, event count, first and last seen, route or `file:line`, environment and release, user impact, verdict.
- What is already solid.
- A phased remediation order, security and data safety first.

**Stop and wait.** He picks what to fix, tells you what to ignore (for example what he attributes to a client's own action), or says "fix them all".

## 4. When he says fix

His fix instruction is the go-word, so hand off without another plan stop.

- Code findings: build mode steps 2 and 4–6, test first, one worktree and one PR per coherent fix set (one PR if he says so).
- Sentry bugs: debug mode per issue.
- Write `docs/planning/NN-<topic>.md` describing what changed per area with file references, and link it from the repo's CLAUDE.md.
- Mark noise issues as ignored in Sentry with the reason, and resolve fixed ones with the PR link after deploy, per the protocol.

## 5. Report

Give the final report from the protocol, then shut the team down.
