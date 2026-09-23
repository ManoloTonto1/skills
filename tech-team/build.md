# Mode: build

Implement a ticket with an agent team. Plan first, then build.


## 1. Read the ticket

- If the input contains a Jira key, fetch it through the Atlassian MCP (`getJiraIssue`): description, acceptance criteria, comments, linked issues and the linked Confluence design page if there is one (`getConfluencePage`).
- Otherwise use the description in the input and the conversation.
- Check the working tree and branch (`git status`, `git branch --show-current`). If it's on `main` or has unrelated changes, tell Manuel before anyone edits anything.

## 2. Spawn and plan

Spawn `techlead`, `advocate`, `qa`, and `backend` and/or `frontend` for the layers the ticket touches. **Planning is read-only: no file edits yet.**

- `qa` records the test baseline now, before any edits.
- Each engineer reads the relevant code and sends you a plan: the files they'll create or change, the call site of every new symbol, the error and validation cases, and the tests that prove each acceptance criterion.
- `advocate` attacks the plans: wrong problem, simpler approach, existing code that already does it, what in production depends on what's changing, deploy order, edge cases.
- `techlead` reviews the plans against the checklist.

Merge the plans into one: per teammate, the files they own and their tasks, in dependency order. Iterate until `techlead` signs off on the plan and `advocate` has no open objections backed by evidence.

## 3. Manuel approves the plan

Show Manuel the plan: the approach, the file ownership, the task order, the test plan, the baseline, and any conflicts between his rules and the repo's conventions. **Stop and wait for his approval.** If the plan has multiple deploy phases (an additive change first, removing the old path later), only the current phase gets built.

## 4. Build

Create the tasks on the shared list with dependencies and let the engineers work in parallel on their own files. Cross-owner changes go by message to the file's owner. Every task goes through the techlead gate (`READY` → findings → fixes → `SIGN-OFF`). Check progress as tasks complete; redirect anyone who drifts from the approved plan, and bring real plan changes back to Manuel.

As the code lands, `qa` tries to break it in the real running app through the Playwright MCP, locks every verified behaviour into E2E specs, adds integration and unit tests with real assertions, and routes failures to the owning engineer.

## 5. Verify

When every task has a techlead sign-off:

1. `qa` goes through every acceptance criterion and edge case in the browser once more, reruns the full baseline plus the new E2E specs, and gives its verdict. On `QA FAIL`, the findings go back to their owners and through the techlead gate again, then `qa` re-verifies. Nothing moves on until `QA PASS`.
2. `advocate` tries to break the finished change and checks sibling code paths for the same issue.
3. `techlead` does one final review of the whole diff (`git diff` against the branch base) — including debug leftovers and changes nobody asked for — and signs off on the whole.

## 6. Report and publish

Give Manuel the final report from the protocol. Shut the team down.

Commit, push, open a PR, comment on or transition the Jira ticket **only when Manuel asks**, showing him the exact commit message, PR title/body or comment first.
