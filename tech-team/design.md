# Mode: design

Design a feature with an agent team. Nobody edits code in this mode.


## 1. Understand the ask

- If the input contains a Jira key, fetch the issue through the Atlassian MCP (`getJiraIssue`), including the description, acceptance criteria, comments and linked issues.
- Otherwise, take the feature idea from the input and the conversation.
- If the goal, the users or the definition of success is actually unclear, ask Manuel before spawning anyone. Don't ask what the code can answer.

## 2. Spawn the team

`techlead`, `advocate`, plus `backend` and/or `frontend` for the layers the feature touches, and `qa` for the test plan. Tell every teammate this is a **read-only design phase: no file edits**.

Assign one research task per engineer, for example:

- `backend`: find the existing capabilities, entities, handlers and events this feature builds on or would duplicate; propose the data model, the endpoints/handlers and the error cases.
- `frontend`: find the existing screens, components, hooks and query keys to reuse; shape the UI with the `impeccable` skill (`shape`) and the `apple-design` skill; propose the UI flow, the states (loading, empty, error), interactions and motion, and the API contract it needs.
- `qa`: walk through the current app with the Playwright MCP to see where the feature fits and what could break, then propose how each acceptance criterion will be proven — which E2E specs per role, the edge cases they cover, and the integration and unit tests behind them. Push back on criteria that can't be tested.
- `advocate`: attack the problem statement and each proposal as it arrives — simpler alternatives, existing features that already do this, production dependencies the change would break, the deploy order, edge cases, partial failure.

Have the engineers and `advocate` message each other directly. The design converges through argument, not through you relaying.

## 3. Assemble the design

Merge the findings into one design doc in your scratchpad (markdown):

1. Problem and goal
2. Scope and explicitly out of scope
3. Design: data model, backend flow, API contract, frontend flow — each naming the existing code it reuses (`path`)
4. Error handling and boundary validation
5. Production safety: what's affected in prod, deploy order, expand/contract steps if needed
6. Test plan: which test proves each acceptance criterion
7. Rejected alternatives and why (including the advocate's objections that were settled)
8. Open questions

Then derive the **story breakdown**: small, independently shippable stories, each with a user story line, acceptance criteria that can be tested, the files or areas involved, and dependencies between stories. No story that only adds something "for later".

## 4. Gate

Send the design and the stories to `techlead` (checks for dead abstractions, duplicated capabilities, missing validation, missing or mocked-wrong tests, stacked retries, ownership overlaps) and to `advocate` for a final pass. Iterate until `techlead` signs off and `advocate` says `NO OBJECTIONS`, or an objection is written up in Open questions for Manuel to decide.

## 5. Approval, then publish

Show Manuel the full design doc and every story exactly as they will be published. Ask where they go: which Confluence space and parent page (`getConfluenceSpaces`), which Jira project and epic (`getVisibleJiraProjects`). Look up the options for him instead of making him type IDs.

Publish nothing until he approves. Apply his edits and show the result again if he changes anything. After approval:

1. Create the Confluence page (`createConfluencePage`).
2. Create the Jira stories (`createJiraIssue`), linking each story to the Confluence page and adding the dependencies between stories (`createIssueLink`).
3. Report the page URL and the story keys.

Then shut the team down.
