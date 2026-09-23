# Mode: build

Implement a ticket with an agent team. Plan first, test first, then build.

## 1. Read the ticket

- A GitHub issue: `gh issue view <n> --comments`, plus linked PRs. A Sentry issue: read it through the Sentry MCP.
- Read any design doc in `docs/planning/` and the ticket's card in `docs/planning/STATUS.md` where they exist.
- Otherwise use the description in the input and the conversation.
- Set up where the work happens, per **Worktrees and parallel tickets** in the protocol: a worktree (`<repo>/.claude/worktrees/<slug>`, branch `<type>/<slug>`, synced with origin/main) for parallel or backlog tickets or when his main checkout has unrelated changes, otherwise a `<type>/<slug>` branch, or main if he said to push straight to main. Tell Manuel where you are. Never stash or discard his local changes.

## 2. Spawn and plan

Spawn `techlead`, `advocate`, `qa`, plus `backend`, `frontend`, `data` and `security` as the ticket needs them (see **Roster** in the protocol). **Planning is read-only: no file edits yet.**

- `qa` records the test baseline now, before any edits (pass/fail/skip counts), and plans the failing tests: E2E for user-visible behaviour, unit for pure logic, integration when the change spans layers.
- Each engineer reads the relevant code and sends you a plan: the files they'll create or change, the call site of every new symbol, the error and validation cases, and the tests that prove each acceptance criterion. For UI: which impeccable command (`craft` for a new feature) and which apple-design principles. Any earlier implementation in Manuel's other repos he pointed at.
- `data` plans every schema change: the migrations and the expand/contract phase each belongs to, the states existing rows are in and how each is handled, the data-loss risk, RLS and grants, and the types to regenerate.
- `security` sends a threat model: what the change newly exposes, who could reach it, and the controls the plans must include.
- `advocate` attacks the plans: wrong problem, simpler approach, existing code that already does it, what in production depends on what's changing, existing users and records (already subscribed and paying, half-finished state, records from before the feature), retries, async third-party state, deploy order, edge cases, and manual cleanup the plan would still leave Manuel.
- `techlead` reviews the plans against the checklist.

Merge the plans into one: per teammate, the files they own and their tasks, in dependency order. Iterate until `techlead` signs off on the plan, `security` has no open threat-model findings, and `advocate` has no open objections backed by evidence.

## 3. Plan gate

Stop and show Manuel the plan (the approach, the file ownership, the task order, the test plan, the baseline, the migration sequence, and any repo-doc drift) when:

- he asked for a plan,
- nothing in his request says go, or
- the plan changes production data (a migration on populated tables, a destructive step).

When his request carried a go-word (/goal, "one shot", "go", "implement it", "I trust you"), post the plan as a short status line and continue to build.

If the plan has multiple deploy phases (an additive change first, removing the old path later), only the current phase gets built.

## 4. Build, test first

1. `qa` writes the failing tests first and shows them red for the right reason.
2. Create the tasks on the shared list with dependencies and let the engineers make the tests green in parallel on their own files. Cross-owner changes go by message to the file's owner.
3. Every task goes through the techlead gate, and the security gate where it applies (`READY` → findings → fixes → `SIGN-OFF` / `SECURITY CLEAR`). `data`'s migrations land before the app code that uses them.
4. Check progress as tasks complete. Redirect anyone who drifts from the plan. Bring changes that alter scope or product behaviour back to Manuel; decide implementation changes yourself.

As the code lands, `qa` tries to break it in the real running app through the Playwright MCP, locks every verified behaviour into E2E specs, adds integration and unit tests with real assertions, and routes failures to the owning engineer.

## 5. Verify

When every task has a techlead sign-off:

1. `qa` goes through every acceptance criterion and edge case in the browser once more. It confirms the change persisted (reload, then query the local DB or Storage), the console is clean at desktop and phone widths, and for UI the impeccable audit findings are fixed. It reruns the full baseline plus the new specs (everything green, zero skips, no error noise) and gives its verdict. On `QA FAIL`, the findings go back to their owners and through the techlead gate again, then `qa` re-verifies. Nothing moves on until `QA PASS`.
2. `security` probes the finished change on the local stack (signed out, as another user, as another workspace or tenant) and gives its final clearance. `data` confirms the migrations apply, roll back and apply again on the local stack, with types regenerated.
3. `advocate` tries to break the finished change and checks sibling code paths for the same issue.
4. `techlead` does one final review of the whole diff (`git diff` against the branch base), including debug leftovers and changes nobody asked for. It runs the build (`bun run build` or the repo's equivalent), scans the diff for secrets and personal data, and signs off on the whole.
5. When Manuel asked for a review or a merge, run the `code-review` skill on the whole diff at high effort (max for large or risky diffs) as an independent pass. Confirmed findings go back through their owners and the techlead gate.

## 6. Report and publish

Give Manuel the final report from the protocol and shut the team down.

Publish exactly as far as his words go (see **Publishing when Manuel says so** in the protocol):

1. Commit with a Conventional Commit. Where `.entire/` exists, check the Entire checkpoint was recorded.
2. Push and open the PR with `Closes #N`.
3. When he asked to merge, run the review-fix-merge loop from `review.md` step 5.
4. Close the resolved issues. After deploy, resolve fixed Sentry issues with the PR link. Remove the worktree.
5. Follow-ups he defers become GitHub issues when he asks.

Without a publish word, stop after the report.
