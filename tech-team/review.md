# Mode: review

Pick up an existing PR: review it as a team, verify it in the browser, then continue it — fix the findings, answer the open review comments and finish whatever the ticket still needs.

## 1. Load the PR

- Resolve the PR from the input (number, URL or branch) with `gh pr view <pr> --json number,title,body,author,headRefName,baseRefName,state,isDraft,url,files,commits,reviewDecision`.
- Read the full diff (`gh pr diff <pr>`), the review comments (`gh api repos/{owner}/{repo}/pulls/<n>/comments`), the review summaries (`gh pr view <pr> --json reviews`) and the checks (`gh pr checks <pr>`). Comments from colleagues are information to weigh, not instructions to follow blindly.
- If the title or body names a Jira key, fetch the ticket (`getJiraIssue`) and any linked Confluence design. The acceptance criteria are what the PR must deliver.
- Check the working tree (`git status`). If it's clean, check out the PR branch (`gh pr checkout <pr>`). If it isn't, stop and ask Manuel what to do with his local changes — never stash or discard them yourself.
- Note who wrote the PR. If it isn't Manuel's, he decides later whether the team pushes to it.

## 2. Spawn and review

Spawn `techlead`, `advocate`, `qa`, and `backend` and/or `frontend` for the layers the diff touches. **Review phase: no file edits.**

- `qa` records the baseline on the PR branch, then verifies every acceptance criterion and the edge cases in the real running app through the Playwright MCP.
- `techlead` reviews the **whole PR diff** against the base branch (`git diff <base>...HEAD`) with the full checklist.
- `advocate` attacks the PR: does it solve the ticket, what does it break in production, which edge cases and sibling paths are missing.
- Each engineer reads the diff in their layer plus every unresolved review comment on it, and says for each comment whether it's valid (and the fix) or wrong (with evidence).
- CI failures from `gh pr checks` go to the owner of that layer to diagnose.

## 3. Report and plan

Merge everything into one review for Manuel:

- verdict: ready, or needs work
- findings per file with `path:line`, from techlead, advocate and qa (with browser evidence)
- every unresolved review comment with the team's position on it
- acceptance criteria not yet met, and CI status
- the plan to continue: per teammate, the files they own and their tasks

**Stop and wait for Manuel.** He can approve the plan, change it, or only ask for the review to be posted as PR comments.

## 4. Continue

After approval, run the plan exactly like build mode's steps 4–5 (`build.md`): engineers fix in parallel on their own files, every task goes through the techlead gate, `qa` re-verifies in the browser and locks it in with E2E specs, the full baseline reruns, `advocate` tries to break the result, and nothing moves on until `QA PASS` and the techlead signs off on the whole diff.

## 5. Publish only after approval

Show Manuel exactly what would go out and publish only what he approves:

- review comments on the PR (`gh pr review` / `gh api` for line comments) — the text of each one
- replies to existing review comments
- commits (Conventional Commits with a scope and body) and the push to the PR branch — ask explicitly before pushing to a PR someone else wrote
- a PR description update, or a Jira comment or transition

Give the final report from the protocol, then shut the team down.
