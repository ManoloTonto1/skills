# Mode: debug

Find the root cause of a bug with an agent team, prove it with runtime data, then fix it.

## 1. Collect the symptom

- Take the symptom from where it came from: a Sentry issue through the Sentry MCP (events, stack, breadcrumbs, release, first and last seen) plus related Sentry issues with the same cause; a GitHub issue (`gh issue view <n> --comments`); a pasted Next.js error overlay, server log or screenshot (treat the screenshot as the reproduction and the acceptance target); a client report. Drop anything Manuel attributes to a client's or user's own action.
- Pin down: the exact observed behaviour, the expected behaviour, where it happens (local, preview or production; which workspace and role), when it started, and the error text or stack trace if there is one.
- When Manuel says it worked before, diff the branch against main first.
- Before blaming code for a local connection error ("Failed to fetch", connection refused), check the infra: Docker up, `supabase status`, `curl :54321/auth/v1/health`, the auth logs, the config.toml providers.
- If there is no reliable way to reproduce the bug yet, finding one is the first task.
- If the error was never captured and can't be reproduced locally, the first deliverable is a capture-only PR: report the error with a route tag and the URL (`Sentry.captureException(error, { tags: { route }, extra: { url } })` where Sentry is installed) plus `console.error` in the catch, response unchanged, published per the protocol. Wait for it to recur, then fix in a follow-up that references the issue.

## 2. Hypotheses

Write down 2–4 competing hypotheses that each explain **every** observed symptom, and for each one the observation that would prove or kill it. Split them by layer or mechanism so they don't overlap.

## 3. Spawn the team

`techlead`, `advocate`, `qa`, plus `backend` and/or `frontend` for the layers involved, `data` when the bug involves wrong, missing or duplicated data or a migration, and `security` when it involves auth, access or exposed data. Give each engineer one or two hypotheses to own; `data` checks the real local data states and what production data the bug has already damaged. Tell them: **investigation phase: the only edits allowed are temporary debug instrumentation, and only in the files you've been assigned**.

- `advocate` tries to disprove every hypothesis, including the favourite, and messages the engineers directly. The theories fight it out between teammates; don't anchor on the first plausible one.
- `qa` records the test baseline now, reproduces the bug by hand in the real running app through the Playwright MCP (capturing console errors, failing network requests and screenshots as evidence), then turns the reproduction into an automated test: an E2E spec when the bug is visible in the UI, otherwise an integration test against the local stack.

## 4. Runtime instrumentation

Only you, the lead, pick the log file. Put it outside the repo so it can never be committed (e.g. `/tmp/<slug>-debug.log`) and tell every engineer the exact path.

Each engineer adds a temporary `debugLog(label, data)` helper in their assigned files that appends one timestamped JSON line per call to that file. Every label starts with the teammate's name and the route so the log is attributable, e.g. `backend:[pay/success] before-attach`, `frontend:[checkout] branch-early-return`. For browser-only code, log through a server action or route the engineer owns, or read the console through `qa`'s Playwright MCP. Instrument function entry and exit, branches, state before and after mutations, and caught errors, with the values that decide between the hypotheses.

Reproduce with the cheapest route that hits the real code path: `qa`'s failing test, a `curl` against the local app, or the E2E flow. For flows only Manuel can walk (an iDEAL bank redirect, a real payment), first put his own test account into the failing state in the local DB and Stripe test mode, then tell him exactly what to click and wait for "check the logs". Read the log, adjust, reset the state for the next run, repeat. Never touch real customers' records.

Engineers read the log, report which hypotheses the data kills, and adjust the instrumentation. Every claim cites log lines. Kill hypotheses with evidence; add new ones when the data demands it.

## 5. Agree on the root cause

The investigation is done when one root cause explains every symptom and every log line, `advocate` has no evidence-backed objection left, and `techlead` agrees the evidence holds. Search the repo for the same defect.

If Manuel asked to analyse first ("before doing anything", "what went wrong", "draft a plan", "come back with the report"), or gave no go-word, **stop and show him the root cause**: the evidence (log lines, failing test), why the other hypotheses were rejected, and the proposed fix (which files, which owner, the regression test). If he asked to fix it ("fix [ISSUE]", "fix it and add an e2e test", a /goal), go straight to step 6 and put the root cause in the report.

## 6. Fix

1. **Remove all instrumentation first.** Each engineer removes their own helper, fs import and call sites. Then `git diff | grep -nE '^\+.*(debugLog|appendFileSync|-debug\.log|console\.log)'` must return nothing.
2. `qa` writes the regression test (if the reproduction isn't one already), runs it, and shows it **failing for the right reason**.
3. The owner fixes the **root cause**, not the symptom, in the style of `rules.md` (no comments, no dead code). A fallback label, a relabel or a Sentry filter is not a fix. Fix the same defect everywhere in the repo. The fix also heals existing bad data (a backfill or reconciliation migration from `data`), or the report says exactly which data it leaves broken.
4. `qa` shows the regression test passing and re-verifies the fix by hand in the browser through the Playwright MCP, including the sibling paths and the edge cases around the bug. It verifies the real end state in the local DB and Stripe test mode (a green test is not proof), reruns the full baseline (green, zero skips, no noise) with the build, and gives `QA PASS` or `QA FAIL`.
5. Techlead gate on every changed file, including a check for leftover instrumentation, and the security gate where it applies.
6. `advocate` tries to break the fix.
7. An independent verification pass over the fix: the `code-review` skill, or a separate agent told to confirm the reported issue is fixed, with its findings fixed. Skip it when Manuel says to ship fast.

## 7. Clean up, report and publish

Delete the debug log file. Give Manuel the final report from the protocol, plus the root cause in plain words and the evidence chain. Shut the team down.

Publish per the protocol, exactly as far as his words go: commit with `Fixes <SENTRY-SHORT-ID>` in the body, then push, open the PR (Sentry short ID in the title) and merge. After deploy, resolve the Sentry issue (where the repo has Sentry) and the related ones with a comment naming the root cause and the PR, and note any issues ignored as noise with the reason.
