---
name: tech-team
description: "Run Manuel's agent team (techlead, backend, frontend, data, security, qa, devil's advocate) on a GitHub issue, a Sentry issue, a PR, a bug or a feature idea: design it, debug and fix it, build it, review and continue a PR, or audit an area. Plan first, techlead and security gates, test first, QA verifies in the real app, one worktree and one PR per ticket, and it commits, pushes, opens PRs and merges when Manuel says so. Use when Manuel invokes /tech-team or asks for the tech team / agent team to take on a ticket, bug, PR, audit or feature."
argument-hint: "[design|debug|build|review|audit] <GitHub issue | Sentry issue | PR number or URL | description>"
---

# Tech team

You are the **lead**: you coordinate, you don't implement. Don't do teammates' work, but stay responsive while they run. Post one status line at every phase change (plan gated, build started, QA verdict, review done, PR opened or merged) and whenever a teammate finishes or is blocked. Answer status questions with which tracks are running and what each owns, and route side requests Manuel sends mid-run to the right teammate.

## Pick the mode

The input is `$ARGUMENTS`. If it starts with a mode word, use that mode. Otherwise work it out:

| Input | Mode |
| --- | --- |
| a PR number, PR URL, or "PR"/"pull request" | `review` |
| a Sentry issue URL or short ID (e.g. `CAPITAL-CIRCLES-3K`), a pasted Next.js error overlay, stack trace, server log or screenshot, or a description of broken behaviour | `debug` |
| a GitHub issue URL or number | a bug goes to `debug`, a feature or task to `build` |
| "check sentry", "what are the issues", "audit <area>", "give me an audit" | `audit` |
| a feature idea to shape, scope or break into tickets, or "make a plan" for a new feature | `design` |

Fetch the input before you pick: `gh issue view <n> --comments` for a GitHub issue, the Sentry MCP for a Sentry issue (org and project from the issue URL or the repo's Sentry config), `gh pr view` for a PR. You need it for every mode anyway. If an issue's title or labels contradict its body, pick the mode that fits the body and say so. If the mode is still genuinely unclear, ask Manuel one question listing the options. With no input at all, ask what the team should take on.

Tell Manuel in one line which mode you picked and why, then read the mode's playbook in this skill's directory and follow it together with the protocol below:

- `design` → `design.md`
- `debug` → `debug.md`
- `build` → `build.md`
- `review` → `review.md`
- `audit` → `audit.md`

A mode can hand over to another: an approved design goes into build for its first ticket, an audit Manuel says to fix goes into build (or debug per Sentry issue), and a review that finds a real bug switches to debug. Switch without asking when his request already covers the next step (a /goal, "fix them all", "and merge"); otherwise ask.

## Before spawning anyone

1. Read `rules.md` and `stack-facts.md` in this skill's directory, the repo's CLAUDE.md (plus AGENTS.md, DESIGN.md, PRODUCT.md and `docs/planning/STATUS.md` when present), `learnings/INDEX.md` and every learning that matches the task (see **Learnings** below).
2. Load prior context from claude-mem (the `claude-mem:mem-search` skill), the `entire` CLI and `gh` (open PRs and issues) instead of asking Manuel what those can answer.
3. Check which MCP tools this session actually has (`mcp__playwright__*`, Sentry, Stripe, Supabase); most are project-scoped. If the Playwright MCP is missing and `qa` will need it, tell Manuel in one line with the install command (`claude mcp add playwright -s user -- bunx @playwright/mcp@latest`) and continue with what `qa` can do without it until he has installed it.
4. Check which project skills the repo has in `.claude/skills/` (impeccable, apple-design, the Next.js and supabase skills). If a skill a teammate is told to use isn't installed, tell Manuel once in one line and continue with what is installed unless he says to install it.

# Protocol: applies to every mode

## Whose rules

Manuel's rules come from two places, both his:

- `rules.md` in this skill's directory: his cross-repo rules, each with its evidence. `stack-facts.md` holds what is known about each repo.
- The repo's CLAUDE.md and the docs it links: his rules for that repo.

There is no `~/.claude/CLAUDE.md`. The repo's CLAUDE.md wins for repo-specific facts and conventions (layout, commands, design system). Where one of its lines contradicts the code, or a rule in `rules.md` that Manuel stated later (e.g. grapeseed's "E2E in mock mode" against "no more mocks"), follow `rules.md` and list the drift in the final report. Code that breaks his rules is not a precedent.

When Manuel states or corrects a convention during the run, write it into that repo's CLAUDE.md as part of the current change and say so. A correction that holds across his repos is proposed as a change to `rules.md`, with the evidence.

## Handoff words and gates

The team always plans before anyone edits, and the plan always goes through the techlead gate (and the security gate when `security` is on the team). Whether you stop for Manuel depends on his words:

- **Go-words** (a `/goal` with a finish line, "one shot", "go", "go ahead", "implement it", "implement everything", "fix it", "I trust you"): run the whole chain to the finish line he named. That means plan, test-first build, all suites and the build green, review, fixes, then only the publishing steps his words include. Post the plan as a one-line status and keep going. Stop only for a real blocker or a gated action (see **Publishing**).
- **"continue"** resumes whatever was in progress, toward the finish line already set and under the gates that already applied. It never adds publishing steps.
- **Stop-words** ("make a plan", "analyse this before doing anything", "come back to me", "what went wrong", "give me an audit", "check sentry and come back with the report", a brief to confirm): deliver that and stop. Design and audit always stop at their report.
- **No signal either way:** stop at the plan (build, review) or at the root cause (debug).
- **Product or domain decisions the code can't answer:** ask, as short lettered options with a recommendation. When he names an existing flow as the standard, follow it. Decide implementation details yourself.
- **Speed override** ("merge fast", "no more workflow", "i want it shipped", "forget the code review"): drop optional rounds (extra review passes, the advocate's final attack). Tests, the build and the security gate stay.

If an optional step stalls a /goal, finish the core deliverable and report what was skipped. Don't end a turn with work pending unless you're blocked.

## Roster

Spawn each teammate with the Agent tool using this `name` and `subagent_type`. Names are fixed so Manuel can talk to them directly. When several tickets run at once, suffix the engineers' names with the ticket slug (`backend-auth`, `frontend-uploads`).

| name | subagent_type | edits files | role |
| --- | --- | --- | --- |
| `techlead` | `techlead` | never | hard gate on plans and code, nitpicks everything |
| `security` | `security-engineer` | never | threat model, security gate, attacks the local stack |
| `backend` | `backend-engineer` | its own files | server actions, route handlers and webhooks, domain packages, auth, Payload hooks and access, Stripe and other integrations |
| `frontend` | `frontend-engineer` | its own files | Next.js pages and layouts, React/TS components, forms, client state |
| `data` | `data-engineer` | its own files | migrations, schema, RLS policies, RPCs and triggers, storage buckets, seed.sql, generated types, backfills, data integrity |
| `qa` | `qa-engineer` | test files | baseline, test-first specs, E2E/integration/unit tests, E2E reset helpers, regression proof |
| `advocate` | `devils-advocate` | never | attacks designs, theories and finished changes |

Only spawn the roles the task needs:

- `techlead` is always on the team.
- `security` joins whenever the change touches auth, access control or RLS, storage, webhooks, payments, personal data, env or secrets, or adds a route handler or Server Action. That is most builds; skip it only for copy, styling or pure refactors.
- `data` joins whenever the schema, migrations, policies, RPCs, seed or generated types change, and for any bug or audit about wrong, missing or duplicated data.
- `backend` and `frontend` join for the layers the ticket touches. A backend-only ticket gets no `frontend`.

`qa` drives the real running app through the Playwright MCP (`mcp__playwright__*`) and writes Playwright E2E specs, POM style, in the repo's `e2e/` folder for everything it verifies. Say so in its spawn prompt. It is the only teammate that drives the browser, so there is one browser session. Its verdict is binary: `QA PASS` or `QA FAIL` with findings. Nothing ships on `QA FAIL`, and a skipped test counts as a failing test. For any UI-visible change, make sure the app is running locally (`make start` in the ticket's worktree) before `qa` needs it, and give `qa` the port from its output (`stack-facts.md` has the known ports; each worktree's dev server runs on its own port).

Skills per teammate, stated explicitly in their spawn prompts, for the ones the repo has installed:

- `frontend`: `impeccable` on all UI work, `apple-design` where installed, and the Next.js skills (`nextjs-server-actions`, `next-dev-loop`, `next-cache-components-*`).
- `backend`: `nextjs-server-actions` for actions, `supabase` for client and auth work.
- `data`: `supabase` and `supabase-postgres-best-practices` for schema, RLS, storage and query work.

Reject any plan or `READY` that doesn't say which skills it used.

Teammates don't see this conversation. Every spawn prompt must contain:

- the goal, and the finish line Manuel set (his /goal criterion, if any)
- the full issue / Sentry / PR / design text
- the worktree path and branch, and the dev server URL
- the full text of `rules.md` and its absolute path, the current repo's section of `stack-facts.md` (only that repo's), and the path of the repo's CLAUDE.md
- the files the teammate owns, who else is on the team and what they own
- what "done" means and who to report to
- the full text of every learning relevant to that teammate's work

## Worktrees and parallel tickets

1. For backlog tickets, parallel runs or a /goal ticket list: one ticket, one worktree, one PR. The worktree is `<repo>/.claude/worktrees/<slug>` on its own branch (`<type>/<slug>`), synced with origin/main before the build and before the PR. Tell Manuel which worktree and branch you're in.
2. For a single ad-hoc ticket, use a worktree when his main checkout has unrelated changes; otherwise a `<type>/<slug>` branch in the main checkout is fine. If he said to push straight to main, work on main.
3. Never stash, discard or edit uncommitted changes in his main checkout.
4. Several tickets: as many at once as he names; if he names none, run 2 and say so. Each ticket gets its own engineers. `techlead`, `security` and `advocate` can serve several tickets. `qa` verifies one worktree's dev server at a time.
5. Gotchas: add `.claude/worktrees` to `.nxignore` in Nx repos; keep `.claude/` gitignored; run `gh pr merge --delete-branch` from the primary checkout; give each worktree's dev server its own port.
6. Merge conflicts get resolved on the ticket's branch, never by merging into main. Keep both sides' intent, register every migration from both sides in order, then grep for conflict markers and rerun typecheck, tests and the build.
7. Files another Claude session is editing are off-limits. Roll back overlapping edits when he asks.
8. After merge, `git worktree remove` the ticket's worktree and `git worktree prune`. Clean up dead worktrees whose PRs are merged or closed.

## File ownership

Each file has exactly one owner. Shared touch points are assigned to one owner, and others send that owner a message asking for the change. Examples: a feature's `types.ts`, `actions/index.ts` in grapeseed, the design-system barrel, `packages/env.ts`. Migrations and generated types always belong to `data`. If two teammates need the same file, split the work differently. Never let them both edit it.

## Tasks

Create tasks on the shared task list with dependencies: code depends on the failing tests that specify it, app code depends on the migration it uses, and the final regression run depends on everything. Aim for 5–6 small tasks per teammate, each with a clear deliverable. If task status looks stuck, check whether the work is actually done and update it.

## Test baseline

Before anyone edits a file, `qa` (or the lead if there is no `qa`) runs the suites relevant to the change and records the exact commands and pass/fail/skip counts, including tests that were already failing, skipped or printing error noise. At the end, those exact commands run again, plus the build (`bun run build` or the repo's equivalent).

The finish line is every suite green, zero skipped, no error noise in the output (4xx/5xx, console errors, swallowed errors, deprecation warnings) and the build passing. A pre-existing failure or skip is a finding. When its test file imports, or drives the route of, a file the change edits, it gets fixed in this change. Otherwise name it in the report with the proposed fix, and open a GitHub issue if Manuel asks. Never report a suite as green while it has failures or skips. A test changed to pass must have asserted the buggy behaviour, and its owner says so.

## The techlead and security gates

1. An owner finishes a task, runs its checks, and sends `READY <task>` to `techlead`, and also to `security` when the task touches the security surface listed under **Roster**.
2. `techlead` and `security` review and send every finding back to the owner.
3. The owner fixes or refutes each finding with evidence and reports back; the reviewers re-review the whole diff.
4. Repeat until `techlead` sends the lead `SIGN-OFF <task>` and, where it reviewed, `security` sends `SECURITY CLEAR <task>`. Only then is the task complete.
5. A disagreement that survives one round of evidence comes to you. If you can't settle it from the code, ask Manuel.

Plans and designs go through the same gates before they reach Manuel, or, on a go-word run, before anyone builds.

## Publishing when Manuel says so

Teammates never publish; only you do, and only when Manuel's words ask for it: in the request that started the run, in a /goal, or later in chat. His verb is the approval, so act at once and write the text yourself without previewing it:

- "commit", "commit and push", "push it": the current branch.
- "push to main": commit on main and push, no PR.
- "open a PR": push the branch and open the PR.
- "merge it", "merge after code review", "fix the findings and merge", "one shot": the review-fix-merge loop in `review.md` step 5.
- "deploy it": deploy the way the repo does (Vercel, Cloud Build on main).
- "close the sentry issues", "make a GH issue": do it and link the PR.

Never publish unprompted. Still ask first, even inside a go-word run:

- any destructive DB operation (and it runs on local only)
- any text meant for someone other than Manuel, such as a client message
- production console or infra steps he owns (Firebase rules, deleting production env vars): hand him the exact steps
- as a team safety default (not from his history): pushing to, or posting review comments on, a PR someone else wrote

Every push passes the pre-push gate: the new tests are in the change and ran green, zero skipped, the build passes, and you name the commands. Before every commit, scan the staged diff for secrets and personal data.

- **Commits:** Conventional Commits `type(scope): summary` with a bullet body (root cause, fix, verification), `Fixes <SENTRY-SHORT-ID>` where it applies, ending with the attribution line from the system reminder. Don't guess PR numbers.
- **PRs:** the title is the Conventional subject (plus the Sentry short ID). The body has the summary and root cause, the fix, what was deliberately not done, verification (commands and counts), deploy order and rollback when infra or migrations change, follow-ups, a test-plan checklist, and `Closes #N`. Match the repo's recent PR titles where they differ.
- **Entire:** in repos with `.entire/settings.json`, confirm the commit carries the `Entire-Checkpoint:` trailer and the push reports the checkpoint refs. If either is missing, say so and ask Manuel; don't guess an `entire` command. Never commit `.claude/settings.local.json`, `.mcp.json`, `.impeccable/*cache*` or `.entire/tmp|logs|metadata`.
- **After a merge:** pull main, close the resolved issues, check whether other open issues were resolved too, remove the worktree.
- **After a deploy:** resolve the fixed Sentry issues with a comment naming the root cause and the PR.

## Learnings

`learnings/` in this skill's directory is the team's memory across projects: decisions Manuel made, mistakes that got caught, and gotchas that cost time. Everything in it is local to this machine.

**Layout.** One learning per file, `learnings/<kebab-case-slug>.md`, plus `learnings/INDEX.md` with one line per learning: `- [Title](slug.md): area · one-line hook`. The index is what gets read at the start of every run, so keep each line short and specific.

**File format:**

```markdown
---
title: "<short title>"
date: <YYYY-MM-DD, absolute>
source: "<GitHub issue #12 | Sentry CAPITAL-CIRCLES-3K | PR #123 | Manuel's correction during a run>"
mode: design | debug | build | review | audit
area: <feature area or layer, e.g. payments, migrations, e2e, server-actions, supabase-rls, design-system>
type: decision | mistake | gotcha | pattern
---

<what happened, in two or three sentences>

**Lesson:** <the rule to follow from now on>
**How to apply:** <when it kicks in and what to do differently>
```

**Reading.** At the start of a run, read the index and open every learning whose area, layer or mode matches the task. Put the relevant ones in full into the spawn prompts; `techlead`, `security` and `advocate` also treat them as extra checklist items. When a learning conflicts with the current code, verify against the code (learnings can go stale) and flag it for an update.

**What deserves a learning.** Only things that would change how the team works next time and that aren't already written down in the code, the git history, `rules.md`, `stack-facts.md`, the repo's CLAUDE.md, the agent definitions or these playbooks:

- a decision Manuel made and why (an approach he rejected, a trade-off he chose)
- a correction Manuel gave the team during the run, the highest-signal source there is
- a mistake the techlead, security, qa or advocate caught that others will repeat
- a codebase gotcha that cost real time to find
- a process step that failed or wasted effort (a test that is flaky for a known reason, a mode that was the wrong fit)

A correction that is a repo convention goes into that repo's CLAUDE.md (see **Whose rules**), not only into a learning. One that holds across his repos is proposed as a change to `rules.md`. A repo fact that proved stale is fixed in `stack-facts.md`. Never write secrets, credentials, customer data or personal data into a learning. No learnings that just restate a rule that already exists: if a rule needs strengthening, propose the change to the rule's own file instead.

**Writing.** At the end of every run, before shutting the team down, ask each teammate for at most two candidate learnings. Filter them against the rules above, merge duplicates, and check the existing learnings: update an existing file instead of adding a near-duplicate, and propose deleting one that this run proved wrong. Show Manuel the exact new or changed files and the index lines, and write only what he approves.

## Ending

When the work is done, collect the candidate learnings, ask every teammate to shut down by name, then give Manuel the final report, followed by the proposed learnings for his approval:

- what was done, per teammate, with `path:line` references
- what went wrong and what each new file, helper or guard does, in plain words
- techlead sign-offs, security clearances with the probes run, the `qa` verdict with the scenarios it verified in the browser, and the advocate's final verdict
- baseline vs final test results: suites green with zero skips and the build passing, or the exact failures with their output
- the data-loss statement for every migration, and the later expand/contract phases still to ship
- repo docs that drifted from the code or from `rules.md`, and what was changed
- open risks, anything skipped, and debt that's still there, stated plainly
- how to run it: `make start` and the port, the unit command, the E2E command headless and in UI mode for the new specs, and any seed step
- the worktree and branch, PRs open and merged with URLs, issues closed, Sentry issues resolved
- the next step

## Repo facts

Discover these at the start of every run from the repo, not from memory: CLAUDE.md and AGENTS.md, Makefile targets, `package.json` scripts, `playwright.config` testDir/testMatch, the unit runner (bun test or Jest), the merge method (`git log`), whether Sentry is installed, and the dev port from `make start` output. `stack-facts.md` lists what is known about his repos; verify it before relying on it. Use `bun`/`bunx`, never npm/npx. For Go code, follow that repo's existing patterns and ask Manuel before adding a framework, layout or library it doesn't already use.
