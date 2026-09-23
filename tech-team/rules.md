# How Manuel codes and works

His cross-repo rules, mined from claude-mem across his TypeScript, Next.js, Supabase and Bun projects (grapeseed/Bucare, capital-circles, DRE-Eugenia-todo-app, whatsapp-webhook, gentle-frame-milano, rekenen-doe-je-zo). Falcon, OnlineResults, .NET, Jira and Confluence patterns are out of scope and never a precedent. [P612]

- **Precedence.** The repo's CLAUDE.md (plus AGENTS.md, DESIGN.md, PRODUCT.md and the docs it links) wins for repo-specific facts and conventions: layout, commands, design system. This file wins over a repo doc line that contradicts a rule Manuel stated later; report the drift.
- **Repo facts.** What is known about each repo is in `stack-facts.md` next to this file. Rules here that name one repo apply only there.
- **Evidence.** Tags in brackets point at claude-mem: `P` is a prompt, `#` an observation. Search them with the `claude-mem:mem-search` skill before retiring or changing a rule.
- **Growing it.** A new cross-repo rule Manuel states during a run is proposed here with its evidence. A repo-only rule goes into that repo's CLAUDE.md (A3).

## A. Where his rules live

- **A1** In his repos the repo CLAUDE.md is his rulebook for that repo, not a mere map. There is no `~/.claude/CLAUDE.md`; this file carries his cross-repo rules. [P40, P105, P236, #1984, #1985]
- **A2** When a repo doc line contradicts a later explicit correction (grapeseed's "E2E in mock mode" vs "no more mocks"), follow the correction and report the drift. [#2078, #3462, P569, P515, P532]
- **A3** When he states or corrects a convention during a run, write it into that repo's CLAUDE.md in the same change and say so. [P40, P105, P236, #1365, #1481]

## F. Handoff and gates

- **F1** Go-words run the whole chain without stopping: `/goal <finish line>`, "one shot", "go", "go ahead", "implement it/all", "fix it", "I trust you". That means plan, test-first build, suites and build green, review, fixes, then only the publishing steps his words include. "continue" resumes whatever was in progress under the gates that already applied to it. [P38, P523, P524, P477, P504, P537, P561, P599, P339]
- **F2** Stop and report when he asks for a plan, analysis, audit, report or brief ("before even doing anything", "come back to me", "draft a plan", "I'll confirm the brief"). [P45, P359, P452, P533, P518, P182]
- **F3** Ask the product and domain questions the code can't answer; for a real decision give short lettered options with a recommendation (he answers "B"). Take a design step by step when he says so. When he names an existing flow as the standard, follow it. Decide implementation details yourself. [P14, P19, P33, P104, P170, P210, P120]
- **F4** Keep going to the finish line; don't hand back mid-task. If an optional step stalls a /goal, finish the core deliverable and say what was skipped. [P188, P481, P525-P532]
- **F5** Speed override: "merge fast", "no more workflow", "i want it shipped", "forget the code review" drop optional ceremony. Tests and build stay green. [P342, P532, P318]
- **F6** Stay in the scope he sets. Build nothing "for later". In an unreleased product, delete old code outright: no deprecation shims or notes. [P75, P457, P532, P595, #3551]

## O. Orchestration

- **O1** Parallelize independent work by default and say how many tracks run and what each owns. Post one status line at every phase change (plan gated, build started, QA verdict, review done, PR opened or merged) and whenever a track finishes or is blocked. Stay responsive to side requests while work runs. [P38, P48, P119, P154, P181, P187, P189, P102, P282, P478]
- **O2** Run as many tickets at once as he names; if he names none, run 2 and say so. He often raises it. [P504, P506]
- **O3** For backlog tickets, parallel runs or a /goal ticket list: one worktree (`<repo>/.claude/worktrees/<slug>`) and one PR per ticket. Say which tree you're in, sync with origin/main before the build and before the PR, and remove dead worktrees after merge. For a single ad-hoc fix he often wants a direct push to main: follow his verb (G3). [P505, P513, P318, P586, P602, #3582-#3589; counter: P391, P549, P587]
- **O4** Worktree gotchas: add `.claude/worktrees` to `.nxignore` in Nx repos; keep `.claude/` gitignored; run `gh pr merge --delete-branch` from the primary checkout (it fails inside a worktree while main is checked out elsewhere); each worktree's dev server gets its own port. [#2044, #2989, #2994, #2101, #3291, #3474]
- **O5** Files another Claude session is editing are off-limits; roll back overlapping edits when he asks. [P186, #1594, #850]

## G. Git and publishing

- **G1** His verb is the approval: "commit and push", "push to main", "open a PR", "merge it", "merge the pr after code review", "deploy it", "close the sentry issues", "make a GH issue". Do it at once and write the commit message, PR title and body yourself, with no preview. Never publish unprompted. [P26, P29, P157, P201, P222, P235, P309, P330, P338, P374, P467, P508, P522, P560, P587, #2042, #1021]
- **G2** Still ask first, even inside a go-word run: destructive DB operations (local only, P287); text for anyone other than him, such as a client message (P322); production console or infra steps he owns, such as Firebase rules or deleting production env vars, for which you hand him exact steps (P343, P327, P530, P280).
- **G3** Target the branch he names. "push to main": commit on main and push, no PR. "open a PR": branch plus PR. Plain "commit and push": the current branch. [P391, P587, #3286, #1099, P361, P274]
- **G4** One concern per PR: one PR per ticket; dependent follow-ups stacked and retargeted to main after the base merges; tests in the same PR as the feature; tooling and skills changes in their own chore PR; risky migrations split (additive, then app, then backfill). He sometimes asks for "one big pr with tests" for a multi-phase plan. [P505, P218, P247, P369, P182, P184, P500, P584, #885, #886; counter: P537]
- **G5** When his request includes merge: run `/code-review` (high; max for large or risky PRs), fix every confirmed finding, rerun typecheck, tests and build, wait for the required checks, merge, delete the branch, pull main. "One shot" means no check-in between review and merge. In whatsapp-webhook the fix round was its own `fix(<scope>): address code-review findings` commit and origin/main was merged into the branch before merging. Don't bypass required checks. [P178, P181, P338, P465, P508, P519, P523, P524, #2009, #2046, #2141-#2147]
- **G6** Pre-push gate: no push, PR or merge unless the new tests are in the change, ran green with zero skips, and the build passes. Say which commands ran. [P369, P370, P374, P516, P555]
- **G7** In repos with `.entire/settings.json`, the committed hooks create an Entire checkpoint per commit and `git push` sends the checkpoint refs. After committing, confirm the commit carries the `Entire-Checkpoint:` trailer and the push reports checkpoint refs. If either is missing, the hooks aren't active: say so and ask Manuel; don't guess an `entire` command. Commit the shared `.claude/settings.json` and `.entire/settings.json`; never commit `.claude/settings.local.json`, `.mcp.json`, `.impeccable/*cache*` or `.entire/tmp|logs|metadata`. [P534, P535, P552, P560, P587, #3287, #3289, #3293, #3504, #2432]
- **G8** Conventional Commits `type(scope): imperative summary`, with a bullet body (root cause, fix, verification), `Fixes <SENTRY-SHORT-ID>` where it applies, and the attribution trailer. PR title: the Conventional subject, plus the Sentry short ID. PR body: summary and root cause, the fix, what was deliberately not done, verification (commands and counts), deploy order and rollback when infra or migrations change, follow-ups, a test-plan checklist, `Closes #N`. Don't guess PR numbers. Match the repo's recent titles where they differ. [#927, #876, #1952, #2046, #2102, #3424, #1527, #1781, #2021, #3578]
- **G9** Branches are `<type>/<kebab-summary>` (fix, feat, chore, hotfix, docs, test); worktree branches may keep `worktree-<name>`. [#1019, #1779, #2189, #1525, #2425, #2544, #3582]
- **G10** Before every commit, scan the staged diff for secrets and personal data (tokens, `.env*`, customer names, phone numbers, ID documents). Keep `.env*` gitignored and `.env.example` current. [#2073, #2226, #2358, P277, P293]
- **G11** Resolve merge conflicts on the feature branch or its worktree, never by merging into main. Keep both sides' intent ("combine, not choose"): the branch's version of files it owns, merged view and query files so main's newer features survive, every migration from both sides registered in order. Then grep for conflict markers and rerun typecheck, tests and build. [P212, P213, #986, #3496, #3497]

## T. Trackers and planning

- **T1** GitHub issues and Sentry are the trackers. Work arrives as a GitHub issue, a Sentry link or short ID, a PR, a pasted error overlay, log or screenshot, a client message, or free text. [P324, P456, P459, P480, P215, P221, P319, P353]
- **T2** Close the loop: `Closes #N` in the PR; after merge, close resolved issues and check whether other open issues were resolved too; after deploy, resolve fixed Sentry issues with a comment naming the root cause and the PR; deferred follow-ups become GitHub issues instead of widening the PR. [P289, P459, P479, P502, P280, P296, #1534, #1535, #2109]
- **T3** Plan in the repo: design and decision docs in `docs/` (audit and fix docs in `docs/planning/NN-<topic>.md`, linked from CLAUDE.md), written before implementing; work broken into user stories with a goal per ticket; read and update `docs/planning/STATUS.md` where it exists. [P2, P5, P45, P146, P152, P244, P561, #629, #637, #3352, #3353]
- **T4** Load prior context before asking him: claude-mem, the `entire` CLI, `docs/planning`, and `gh` PR and issue history. [P20, P27, P63, P121, P243, P318, P456, P195, P205]

## Q. Testing

- **Q1** Test first: write the failing test (bug repro or feature spec), show it red for the right reason, implement to green, then run the full suite. [P89, P216, P504, P537, P543, #1011, #1014, #3262]
- **Q2** In the same PR as the feature or fix: Playwright E2E for user-visible behaviour, unit tests for pure logic, integration tests when the change spans layers. A Sentry fix gets a regression test, E2E when it's visible. [P236, P247, P284, P353, P369, P453, #1644, #2495]
- **Q3** Zero skips: a skipped test is a failing test. No `test.skip(!hasX())` guards; seed what the spec needs and reset state through the DB. [P348, P555, P558, #1644, #2438, #3304, #3311]
- **Q4** Run every test and read the whole output and exit code. Noise is a failure: 4xx/5xx, console errors, unhandled rejections, "non-fatal" errors caught in setup or teardown, and deprecation warnings in dev, build or CLI output. Fix the cause (for a deprecation: the package upgrade or documented config migration, cited); don't silence it. [P369, P370, P371, P372, P345, P240, P227, P229, #2483, #2559-#2564, #1104-#1106]
- **Q5** Assert the real outcome end to end (state persisted, tier switched, record or email created) plus the failure path. A test that only sees a dialog open proves nothing. Hard `expect` only: no `if (await x.isVisible())`, no `.catch(() => false)`. [P557, P568, P485, P394, #3305, #3306, #3316, #2611]
- **Q6** No mock code in the app past PoC: no mock or demo modes, `NEXT_PUBLIC_USE_MOCKS`, `withMockToggle`, fixtures standing in for the DB, or `*Real`/`*Mock` names. E2E and manual checks run against the real local stack (local Supabase or Postgres with migrations and seed) and Stripe test mode. Unit tests may inject in-memory dependencies through a context parameter and stub framework modules that can't load in the runner (`next/cache`, `next/headers`). [P490, P492, P497, P510, P515, P532, P569, #3369, #3370]
- **Q7** Playwright for E2E, in POM form: page objects are classes (extending the repo's `BasePage` where one exists) with Locator fields and intent-level methods under `e2e/pages/`; specs in the repo's `e2e/` folder; helpers in `e2e/helpers/`; no raw selectors in specs. Unit tests are colocated `foo.spec.ts`/`foo.test.ts`, never in `__tests__/`. [P284, P264, P486, P464, #1223, #1637, #2437, #3467, #3469]
- **Q8** Keep runners apart: Playwright reads only `e2e/`, and bun test or Jest never loads E2E specs. [#1642, #1146, P486]
- **Q9** Drive the running app with the Playwright MCP while writing and validating specs, checking console and network. For a bug fix, also run an independent verification pass (code-review or a review agent) that confirms the reported issue is fixed. [P458, P554, P583, P455, P290, P356, #3473]
- **Q10** Put test data into the needed state yourself: Supabase SDK or pg on the local DB, Stripe test mode via SDK or MCP. For checks he walks himself, set up his own test account (never real customers), then verify the end state in the DB and Stripe. [P377, P393, P395, P305, P558, #2571, #2587, #2626, #3309]
- **Q11** Deterministic E2E data: a local-only seed script that refuses a non-local database URL and redacts it in the error; known seeded IDs; per-test resets in `beforeEach`; credentials from the seed's gitignored `.env.e2e`. A suite that mutates shared DB state runs in serial mode and each test seeds its own preconditions. No retries to paper over races; a helper-level retry is allowed only after checking the real state (the session exists) and is named in the report. [P557, P558, #3262, #3282, #3307, #3309, #3314, #3316, #2441, #2443; counter: #2478]
- **Q12** Done means the build passes (`bun run build` or the repo's equivalent) and all suites are green. Search the repo for the same defect before calling it fixed. [P516, #1566]
- **Q13** A pre-existing failure is named in the report with a proposed fix, never carried silently. It is in the area of the change when its test file imports, or drives the route of, a file the change edits: fix those in the PR. List the rest with a proposed fix, and open a GitHub issue when Manuel asks. [#2039, #2043, #2045, #1644]

## D. Debugging and Sentry

- **D1** In repos with Sentry, it is the intake, through the Sentry MCP (org and project from the issue URL or the repo's Sentry config). "check sentry" or "what are the issues": a prioritized report, and he picks. Each entry: short ID, title, events, first and last seen, route or file:line, environment and release, impact, a verdict (real bug, third-party noise, transient infra, already fixed, regressed) and the proposed fix. "fix [ISSUE]": fix it and the related issues. Drop what he attributes to a client's own action. [P215, P221, P274, P281, P538, P540, P353, #3234, #1556, #1583]
- **D2** If the failure can't be seen, capture it first: a small observability-only PR (report the error with a route tag and URL, plus `console.error`, response unchanged), let it recur, then fix in a follow-up that references the issue. [P304, P312, #1934, #1936, #1942, #1951, #2003]
- **D3** Instrumentation loop: add a temporary `debugLog` helper that appends timestamped JSON lines to a log file outside the repo (`/tmp/<slug>-debug.log`) with a prefix per teammate or route; prepare the state; reproduce it yourself if you can, otherwise tell him exactly what to click and wait; read the log when he says "check the logs"; iterate; remove every trace (helper, fs import, call sites) and grep before committing. [P383-P386, P389, P390, P391, P396, P397, #2588, #2598, #2599, #2600]
- **D4** Fix the root cause, not the display: no fallback labels ("System"), no filter-only fixes, no manual cleanup left to him. Explain what went wrong in plain words, and trace the specific reported case through the code when asked. [P572, #3374, #3379, P394, P206, P225, P359, P365]
- **D5** Back claims about third-party behaviour with official docs or the installed library source, cited. Prefer the documented solution. Docs are necessary but not proof: a docs-based iDEAL fix that passed mocked tests broke production. [P72, P360, P382, P227, P329, P331, P495, #1739, #1747, #2399]
- **D6** For async third-party state (Stripe `generated_sepa_debit`), poll a bounded number of times and keep the webhook as the guaranteed path, both through one shared helper. [P387, P388, #2593, #2594, #2636, #2642]
- **D7** Local connection errors: check infra first (Docker, `supabase status`, `:54321/auth/v1/health`, auth logs, config.toml providers). [P596, P528, P530, #3549, #3550, #3580]
- **D8** Sentry noise: fix your own part first, then mark pure noise as ignored in Sentry with the reason. Never hide a broad class of errors. [#1103, P289]

## E. Errors and observability

- **E1** Route every fallible call through the repo's `tryCatch`, which returns `{ data, error }`. Destructure, check `error` first with an early return, then use `data`; never destructure only `data`. Bare try/catch only around third-party libraries that throw. Match an established module pattern where one exists (grapeseed server actions throw `ActionError`). [P82, P219, P380, #438, #1040, #1985, #2213, #2581]
- **E2** No silent errors. Every error branch propagates, or logs with context and reaches the repo's error reporter (Sentry where installed, with a route tag and the URL) plus `console.error`. An empty catch, `catch (_error)`, an unused destructured `error`, or a generic 500 with no log is a defect, in test helpers too. [P82, P219, P380, #1934, #1942, #2561]
- **E3** Ignore only a known-harmless case, matched by exact code (Stripe `invalid_payment_method_attachment`), and rethrow the rest. Never match on substrings. [#2577, #2579, #2580, P382]
- **E4** A missing record is not an exception: non-throwing lookups (Payload `find` with `where` and `limit: 1` over `findByID`; Supabase `.maybeSingle()`) return `T | null` and the caller calls `notFound()`. An upstream 404 becomes null, not a 500 or an error report. [#1059, #1060, #1069, #1506, #1507, #1534, #3271]
- **E5** Remove temporary debug logging before commit or push. [P391, P397, #2600, #2259]
- **E6** Retry only transient network failures on idempotent calls (`fetchWithRetry`, 2 retries with backoff). HTTP errors go to the caller; non-idempotent side effects are never auto-retried. [#1504, #1535, #1877, P388]
- **E7** Check every success and failure field of an external response before telling the user it worked. Parse error bodies defensively (`response.text().catch(() => "")`). [#2020, #2018, #1995, #2046]

## C. Code style and structure

- **C1** No comments: no narration, explanatory JSDoc, TODOs, tombstones ("X removed, use Y") or commented-out code. Explanations go in the report and the commit or PR body. [P314, P595, #3531, #3551]
- **C2** No dead code. Every symbol added has a caller in the change. After a migration or refactor, delete unused exports, files, legacy components, stale imports and constants; grep each one. [#3570, #3567, #3344, #3384, #1990, #3357]
- **C3** Guard clauses and early returns. A refactor changes no behaviour and keeps every test green. [P398, #2637, #2642, #2644, #2645]
- **C4** Domain logic lives in `packages/<domain>/` (e.g. `packages/stripe`, `packages/payments`, `packages/uploads`). Routes, pages, webhooks and `index.ts` only parse, authorize, call one package function and respond. [P100, P398, #488, #826, #2638, #1985, #3179, #1504]
- **C5** No underscore-prefixed folders (`_components`, `_steps`, `_lib`). grapeseed also bans underscore files and keeps `app/` routes-only; capital-circles keeps its `_actions.ts` file convention. shadcn primitives stay in `components/ui` or the design system's `src/ui`. [P39, P40, P105, P263, P272, #2528, #1572]
- **C6** Name things by what they do. No version suffix on exports (the version lives in the module path, `events/<aggregate>/v1.ts`) and no mode words (`fooReal`, `fooMock`). [P270, P271, P497]
- **C7** Reuse before writing: the repo's utils (e.g. `checkRole` from `@/access`), the component another flow already uses, the flow he declared the standard, and his earlier implementation in another repo when he points at one (`~/GitHub`, github.com/ManoloTonto1). Keep one source of truth; merge duplicated logic into one domain-package function. [P299, P135, P120, P115, P589, P542, #1938, #2637, #2013, #3490]
- **C8** Use the simplest idiom: spread the validated input instead of copying optional fields one at a time (`Record<string, unknown>` plus `if (x !== undefined)`), and call the platform or SDK directly instead of a hand-written wrapper or loop that does the same job. [P498]
- **C9** Named domain types go in each domain's `types.ts`, derived from the generated Supabase `Database` or Payload types. No `Awaited<ReturnType<...>>` chains, no inline `as User | null` casts, no bare `any`; `import type` for types. [#1985, #2216, #2062, #3213, P299]
- **C10** Magic values and user messages live in constants modules (`constants.ts` per package, `messages/errors.ts`). Delete unused constants. [#1985, #1859, #1990, #3479, #3213]
- **C11** Run independent reads in `Promise.all` and check each result. [#2022, #1988, #2574]
- **C12** Read env through the repo's Zod `Env` object where one exists (whatsapp-webhook `packages/env.ts`); new vars go in the schema and `.env.example`. [#1985, #2088, #2276, #3569]
- **C13** Don't change UI or behaviour he didn't ask to change. Anything he pinned or approved stays identical: `git diff main -- <file>` is empty, and a Playwright screenshot of it on main and on the branch at 1440x900 and 390x844 shows no difference (global CSS can still move it). On a regression, diff the branch against main first. [P137, P132, P339, P155, P172, P120, #2335, #2356]
- **C14** Take the simplest route that is reliable. Fold small external services and queues into the repo, call an API directly instead of automating a browser, and remove infrastructure that is no longer used (dependency, route, platform config, env vars) instead of keeping it around. A plan names the moving parts it adds and removes. [P291, P274, #1867]

## N. Next.js

- **N1** Mutations are Server Actions in `'use server'` files. Never `'use client'` on an action file. The file layout is per repo (`stack-facts.md`). [P566, P567, #3359, #3362, #3425]
- **N2** Pages are thin and server-first: load data through a package function returning `T | null`, call `notFound()` on null, render the feature component. No client hooks for data the server can load. [#1534, #3271, #3155, #3362]
- **N3** View state lives in URL search params (`?tab=`, `?step=`, `?selected=`), via nuqs where the repo already has it, with `scroll: false` and no param for the default tab. [P42, #3334, #3335, #2335, #1077]
- **N4** Data read by layouts and `generateMetadata` (site config, globals) goes through a cached helper (`lib/payload-cache.ts` in Payload repos, with `depth: 0`, `select` and a revalidation path) that returns null on failure and reports the error, so the layout renders defaults and a CMS or DB outage never 500s every page. Never an uncached `findGlobal` in a layout. [#2490, #2491, #2497, #2263, #3271, P353, P355]
- **N5** Use the repo's installed skills for the work they cover and name the one used: `nextjs-server-actions`, `next-dev-loop`, `next-cache-components-*`, `supabase`, `supabase-postgres-best-practices`, `impeccable`, `apple-design`. [P567, P575, P580, P593, P355, P354, P495, P559]
- **N6** Upgrade Next.js with the official codemods and upgrade guide, with a plan first when the upgrade is large. Keep every `@payloadcms/*` package on the same version as `payload`. [P9, P49, P346, P294]
- **N7** Next.js 16 request interception lives in `proxy.ts`. Never add a `middleware.ts` next to it; the "middleware file convention is deprecated" warning or a middleware/proxy conflict is a defect. [#1152, #3566, P603]

## S. Supabase and data safety

- **S1** Never lose data: new columns on populated tables are nullable with a NULL default; never edit an applied migration; generate migrations with the tool (Supabase CLI, Payload `migrate:create`), never by hand; dry-run and check `migrate:status`; ship additive schema, then app code, then backfill, NOT NULL or drop, as separate PRs in order; migrations are idempotent with a working down(); a schema change ships with its migration; state the data-loss risk. [P164, P166, P167, P182-P185, P194, P204, P205, P297, #777, #866, #885, #886]
- **S2** Destructive DB operations need his explicit approval and run on local only. Destructive scripts refuse non-local hosts and never delete admin or editor accounts. [P287, #1645, #3265, #3282]
- **S3** Storage buckets, RLS policies and functions ship as SQL migrations applied with the Supabase CLI (use the `supabase` skill), never through Studio. [P580, P581, #3439, #3453]
- **S4** Personal and sensitive files are never public: private storage, accessed server-side only (short-lived signed URLs or the Admin SDK). [P293, P340, #2285, #2304, #3451]
- **S5** In repos that query Supabase directly (grapeseed, DRE): RLS on every public table; SECURITY DEFINER RPCs check membership; EXECUTE revoked on internal functions; explicit column selects; secrets in config.toml via `env()`. In Payload repos (capital-circles, gentle-frame-milano), access control lives in Payload's `access/` functions. [P593, P599, #3561, #3542, #3578]
- **S6** Supabase clients come only from the repo's shared factories: the cookie-bound anon server client by default so RLS applies, a browser client in the browser, the service-role client only for explicit admin work. No inline `createServerClient` in routes. [#3539, #3565, #3568, #3562, #2536]
- **S7** Regenerate types after schema changes (`make gen-types`, Payload `bun run types`) and commit them with the change. [#3290, #752]
- **S8** Stress-test against existing users and data: people already subscribed and paying, half-finished registrations, retries, records from before the feature, duplicate subscriptions. Adopt or reconcile orphans rather than deleting referenced rows. Walk him through a specific existing account along the code path when asked. [P217, P364, P365, P375, P376, P387, P394, P159, #1057, #2613]
- **S9** Auth: gate every non-public route server-side so the app can't be reached until the user is logged in. When a session expires, send the user to sign-in with a clear "session expired" message, not a generic error. One auth helper serves pages and actions, and it tells "unauthenticated" apart from "profile or customer record missing". [P514, P541]

## P. Stripe and integrations

- **P1** Stripe logic lives in `packages/stripe`, shared by the success route and the webhook. Webhooks are idempotent (idempotency keys like `sub_from_pi_${id}`, compare-and-swap columns like `emails_sent_at IS NULL`). Map every subscription status, not only `active`. [P398, #2642, #859, #1126, #828, #2530]
- **P2** iDEAL recurring: charge the first period for real (PaymentIntent with `setup_future_usage: 'off_session'`, or Checkout); resolve `latest_charge.payment_method_details.ideal.generated_sepa_debit`; attach it and set it as default on the subscription and `invoice_settings`; poll in the success route and mirror in the webhook. Check the Stripe docs first. [P382, P387, P388, #2603, #1127, #1738]
- **P3** Payload: `payload/collections`, `payload/hooks`, `access/` (`checkRole`, `adminOnly`); `required` matches DB nullability; API and MCP keys are least privilege. [P294, #1860, #776, #3240, #2425]
- **P4** whatsapp-webhook: Firebase through the Admin SDK only (`preferRest: true` on Bun) with deny-all client rules; sanitize Firestore reads; irreversible submits sit behind an off-by-default break-glass flag and are never auto-retried; user messages are in Spanish. [#2330, #2337, #2333, #1886, #1877, P343]

## U. UI and design

- **U1** Every UI task uses `impeccable`: `craft` to build, `shape` for a redesign or brief, `audit`, `polish` and `harden` to check. All impeccable findings fixed is part of done. Use `apple-design` as well where the repo has it (grapeseed). If a skill isn't installed, work from DESIGN.md and PRODUCT.md and say the skill was unavailable. [P16-P20, P22, P36, P58, P91, P97, P339, P476, P477, P507, P517, P559, P574, P575, P589]
- **U2** Read PRODUCT.md, DESIGN.md and `.impeccable/design.json` when present and follow their named rules. Each project has its own design language. For a redesign without them, create them first. [P339, #2196, #2315, #2318, P349, #1611]
- **U3** Build from the project's design system. If a primitive is missing, add it to the design-system package. Migrate local duplicates to the shared component and delete them; grep the design system's ui folder first. [#3381, #3384, #3382, #1611, #2354]
- **U4** The UI explains itself: a disabled button has a tooltip saying what's missing; every empty list or tab shows an EmptyState with a specific title, an instructive description and one primary CTA per purpose; admin screens say what an action does and block destructive ones clearly; tables are sortable; conversations never dead-end. [P494, P574, P577, P578, P54, P55, P17, P333, P311, #3390, #2153]
- **U5** No em dashes in anything he or his users read: UI, bot messages, emails, docs, PR bodies, client messages, these skill files. No decorative icons. [P77, P322, P98]
- **U6** A UI action is done only when the record exists in the DB or Storage and survives a reload. The backing bucket, migration and action ship with the UI. [P568, P575, P580, #3434, #3495]
- **U7** Read a component's props before passing them. After any UI change the browser console is free of React warnings and errors. [P582, #3461, #3464]
- **U8** Verify at desktop 1440x900 and phone 390x844, with a screenshot of each changed state and no horizontal overflow. Delete temporary preview routes. [P583, #2267, #2390, #2314]
- **U9** Tailwind utilities, tokens and cva variants. No inline `style={{}}` or arbitrary values for color, spacing, layout or static type. Never put an interactive element inside another (`<Link><Button>`, `<a><button>`): when a button must navigate, use `<Button asChild><Link/></Button>`; inside a card that is already a link, style the call to action as a non-interactive span with `buttonVariants`. Fix every sibling instance. [#1611, #2349, #2350, #1098, #1563]
- **U10** JS motion respects reduced motion (`MotionConfig reducedMotion="user"`) and follows the project's motion tokens. [#2361, #2381, #3332]
- **U11** Copy is in the product's language (Spanish for the WhatsApp bot); keep Dutch domain terms as he writes them. Where the repo has an i18n setup (grapeseed: i18next, English first), UI text goes through it, never hard-coded. [P306, P307, P14, P147, P493, #2078, #2168, #3373]

## R. Reporting

- **R1** End every report with: exact commands (`make start` and the port, the unit command, E2E headless and in UI mode for the new specs, any seed step); open and merged PRs with URLs, closed issues, resolved Sentry issues; the worktree and branch; the next step. [P373, P544, P545, P551, P556, P260, P466, P499, P531]
- **R2** Explain every new concept, helper, file or guard in plain words, and restate the goal when a thread gets long. [P546, P547, P548, P226, P303, #1019]

## K. Tooling

- **K1** Bun for every project, package and test command (`bun`, `bunx`, `bun add`, `bun run`), never npm, npx or node. When he pastes an npx command, run the bunx equivalent. If a repo has only a non-Bun lockfile, follow it and say so. [P487, P462, P460, #1985]
- **K2** The root Makefile is the entry point: `make start`/`make dev` brings up the local stack (plus `stripe listen` where there are payments) and the dev server, and must work on a clean checkout; `make test`, `test-unit`, `test-e2e`, `supabase-reset` and `gen-types` where present. Add a missing target rather than documenting a manual step. [P238, P484, P489, P470, P542, #3242, #3258, #3290, #2201]

## X. Go

- **X1** Memory holds no Go work. For Go code, follow that repo's existing patterns and ask before adding a framework, layout or library it doesn't already use.
