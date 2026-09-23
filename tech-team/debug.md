# Mode: debug

Find the root cause of a bug with an agent team, prove it with runtime data, then fix it.


## 1. Collect the symptom

- If the input contains a Jira key, fetch it through the Atlassian MCP (`getJiraIssue`) with its comments and attachments.
- Pin down: the exact observed behaviour, the expected behaviour, where it happens (environment, tenant/client, user role), when it started, and the error text or stack trace if there is one.
- If there is no reliable way to reproduce the bug yet, finding one is the first task.

## 2. Hypotheses

Write down 2–4 competing hypotheses that each explain **every** observed symptom, and for each one the observation that would prove or kill it. Split them by layer or mechanism so they don't overlap.

## 3. Spawn the team

`techlead`, `advocate`, `qa`, plus `backend` and/or `frontend` for the layers involved. Give each engineer one or two hypotheses to own. Tell them: **investigation phase — the only edits allowed are temporary debug instrumentation, and only in the files you've been assigned**.

- `advocate` tries to disprove every hypothesis, including the favourite, and messages the engineers directly. The theories fight it out between teammates; don't anchor on the first plausible one.
- `qa` records the test baseline now, reproduces the bug by hand in the real running app through the Playwright MCP (capturing console errors, failing network requests and screenshots as evidence), then turns the reproduction into an automated test — an E2E spec when the bug is visible in the UI, otherwise an integration test.

## 4. Runtime instrumentation (debug-mode)

This is Manuel's installed `runtime-debugging` skill (claudecode-debug-mode), adapted for the team. **Only you, the lead, run the server.** Start it with its working directory **outside the repo**, so the log can never be committed:

```bash
SERVER=$(ls -d ~/.claude/plugins/cache/pzep1-claudecode-debug-mode/claude-debug-plugin/*/scripts/debug-server.js | tail -1)
DEBUG_DIR="<your scratchpad directory>/claude-debug"
mkdir -p "$DEBUG_DIR" && cd "$DEBUG_DIR" && node "$SERVER" 3333
```

Run it in the background, then check it with `curl -s http://localhost:3333/health`. The log is `$DEBUG_DIR/.claude-debug/debug.log`; tell every engineer that path.

Every label starts with the teammate's name so the log is attributable, e.g. `backend:HandleAsync-entry`, `frontend:branch-early-return`, `backend:state-before-save`. Instrument function entry and exit, branches, state before and after mutations, and caught errors — with the values that decide between the hypotheses.

TypeScript / browser / Node:
```ts
fetch("http://localhost:3333/debug", { method: "POST", headers: { "Content-Type": "application/json" }, body: JSON.stringify({ label: "frontend:label", data: { key: value } }) }).catch(() => {});
```

C# (.NET API, EventHandler, Functions):
```csharp
_ = System.Net.Http.Json.HttpClientJsonExtensions.PostAsJsonAsync(new System.Net.Http.HttpClient(), "http://localhost:3333/debug", new { label = "backend:label", data = new { key = value } });
```

If the process runs inside Docker, use `http://host.docker.internal:3333/debug`.

Reproduce with the cheapest route that hits the real code path: `qa`'s failing test, a `curl` against the local API, or the E2E flow. Ask Manuel to reproduce it by hand only when none of those can reach it; then tell him exactly what to do and wait for him to say he's done.

Engineers read the log, report which hypotheses the data kills, and adjust the instrumentation. Every claim cites log lines. Kill hypotheses with evidence; add new ones when the data demands it.

## 5. Agree on the root cause

The investigation is done when one root cause explains every symptom and every log line, `advocate` has no evidence-backed objection left, and `techlead` agrees the evidence holds. Check sibling code paths for the same defect.

**Stop and show Manuel the root cause**: the evidence (log lines, failing test), why the other hypotheses were rejected, and the proposed fix — which files, which owner, the regression test. Wait for his approval before anyone fixes anything.

## 6. Fix

1. **Remove all instrumentation first.** Each engineer removes their own debug calls. Then `git diff | grep -n "localhost:3333\|host.docker.internal:3333"` must return nothing.
2. `qa` writes the regression test (if the reproduction isn't one already), runs it, and shows it **failing for the right reason**.
3. The owner fixes the **root cause**, not the symptom — in the style of Manuel's rules, no comments, no dead code — and fixes the same defect in the sibling paths.
4. `qa` shows the regression test passing, re-verifies the fix by hand in the browser through the Playwright MCP, including the sibling paths and the edge cases around the bug, reruns the full baseline, and gives `QA PASS` or `QA FAIL`.
5. Techlead gate on every changed file, including a check for leftover instrumentation.
6. `advocate` tries to break the fix.

## 7. Clean up and report

Stop the debug server (`pkill -f debug-server.js`) and delete `$DEBUG_DIR`. Give Manuel the final report from the protocol, plus the root cause and the evidence chain. Shut the team down.

Commit, push, open a PR, or comment on or transition the Jira ticket **only when Manuel asks**, showing him the exact content first.
