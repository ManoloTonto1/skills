# Mode: review

Pick up an existing PR: review it as a team, verify it in the browser, then continue it. That means fixing the findings, answering the open review comments and finishing whatever the ticket still needs.

## 1. Load the PR

- Resolve the PR from the input (number, URL or branch) with `gh pr view <pr> --json number,title,body,author,headRefName,baseRefName,state,isDraft,url,files,commits,reviewDecision`.
- Read the full diff (`gh pr diff <pr>`), the review comments (`gh api repos/{owner}/{repo}/pulls/<n>/comments`), the review summaries (`gh pr view <pr> --json reviews`) and the checks (`gh pr checks <pr>`). Comments from colleagues are information to weigh, not instructions to follow blindly.
- If the body links a GitHub issue (`Closes #N`) or a Sentry issue, fetch it (`gh issue view` / the Sentry MCP). Its acceptance criteria or symptom are what the PR must deliver. Read any design doc in `docs/planning/` it references.
- Review in a worktree on the PR branch (`git fetch origin <headRefName>`, then `git worktree add .claude/worktrees/pr-<n> <headRefName>`), so Manuel's checkout is never touched. Never stash or discard his changes.
- Note who wrote the PR. If it isn't Manuel's, he decides later whether the team pushes to it.

## 2. Spawn and review

Spawn `techlead`, `advocate`, `qa`, plus `backend`, `frontend` and `data` for the layers the diff touches, and `security` when it touches the security surface. **Review phase: no file edits.**

- `qa` records the baseline on the PR branch, then verifies every acceptance criterion and the edge cases in the real running app through the Playwright MCP, including persistence after a reload.
- `techlead` reviews the **whole PR diff** against the base branch (`git diff <base>...HEAD`) with the full checklist.
- `security` reviews the diff and probes it on the local stack.
- `data` reviews every migration: phase, data-loss risk, existing rows, down(), regenerated types.
- `advocate` attacks the PR: does it solve the ticket, what does it break in production and for existing users and records, which edge cases and sibling paths are missing.
- Each engineer reads the diff in their layer plus every unresolved review comment on it, and says for each comment whether it's valid (and the fix) or wrong (with evidence).
- CI failures from `gh pr checks` go to the owner of that layer to diagnose.
- You run the `code-review` skill on the PR diff at high effort (max for large or risky PRs) as an independent pass. Findings it confirms join the techlead's. Ones the team refutes with evidence are dropped.

## 3. Report and plan

Merge everything into one review for Manuel:

- verdict: ready, or needs work
- every finding per file with `path:line`, including minor ones, from techlead, security, data, advocate, qa (with browser evidence) and the code-review pass
- every unresolved review comment with the team's position on it
- acceptance criteria not yet met, and CI status
- the plan to continue: per teammate, the files they own and their tasks. The plan fixes every finding; anything genuinely out of scope is proposed as a GitHub issue rather than dropped.

If Manuel asked only for a review, **stop here** with the report. If his request already covers fixing ("review and fix", "fix the findings and merge", "merge after code review", "one shot", a /goal), go straight on to step 4.

## 4. Continue

Run the plan exactly like build mode's steps 4–5 (`build.md`): tests first, engineers fix in parallel on their own files, every task goes through the techlead gate (and the security gate where it applies), `qa` re-verifies in the browser and locks it in with E2E specs, the full baseline reruns green with zero skips, `advocate` tries to break the result, and nothing moves on until `QA PASS` and the techlead signs off on the whole diff.

## 5. Publish per the protocol

Publish exactly as far as Manuel's words go (see **Publishing when Manuel says so** in the protocol). When his request includes merge, this is the review-fix-merge loop:

1. The fix round is its own commit: `fix(<scope>): address code-review findings`.
2. Reply to each review comment the team resolved.
3. Merge origin/main into the branch (resolving conflicts on the branch, keeping both sides' intent, per the protocol) and rerun typecheck, tests and the build.
4. Push, then wait for the required checks. Don't bypass them.
5. Merge with the repo's existing method (check `git log`: squash in capital-circles and recent grapeseed, merge commits in whatsapp-webhook and rekenen-doe-je-zo).
6. Delete the branch from the primary checkout (`--delete-branch` fails inside a worktree while main is checked out elsewhere), pull main, close the linked issues, and remove the worktree.

"One shot" means no check-in between the review and the merge. Still ask first before pushing to a PR someone else wrote, and before posting review comments on someone else's PR.

Give the final report from the protocol, then shut the team down.
