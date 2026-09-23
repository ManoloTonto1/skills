# skills

Claude Code skills and the agent definitions they depend on.

## tech-team

`/tech-team` runs an agent team (techlead, backend, frontend, qa, devil's advocate) on a Jira ticket, a PR, a bug or a feature idea, in `design`, `debug`, `build` or `review` mode.

Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`, the Playwright MCP (`claude mcp add playwright -s user -- npx @playwright/mcp@latest`), and the `impeccable`, `apple-design` and `runtime-debugging` skills.

Install:

```bash
cp -R tech-team ~/.claude/skills/
cp agents/*.md ~/.claude/agents/
```

Learnings are machine-local: `tech-team/learnings/` ships with an empty index and fills up as the team runs.
