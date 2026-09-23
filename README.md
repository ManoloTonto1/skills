# skills

Claude Code skills and the agent definitions they depend on.

## tech-team

`/tech-team` runs an agent team (techlead, security, backend, frontend, data, qa, devil's advocate) on a GitHub issue, a Sentry issue, a PR, a bug or a feature idea, in `design`, `debug`, `build`, `review` or `audit` mode. It is built for Manuel's TypeScript, Next.js, Supabase and Bun projects.

Requires:

- `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`
- the Playwright MCP: `claude mcp add playwright -s user -- bunx @playwright/mcp@latest`
- the `gh` CLI, and the Sentry MCP (plus the Stripe and Supabase MCPs where the repo uses them)
- the `entire` CLI in repos with `.entire/`
- the `impeccable` skill in the repo for UI work (plus `apple-design` where wanted), and the `supabase` and Next.js skills where the repo has them

Install:

```bash
cp -R tech-team ~/.claude/skills/
mkdir -p ~/.claude/agents && cp agents/*.md ~/.claude/agents/
```

Or symlink them so edits in this repo are live:

```bash
ln -s ~/GitHub/skills/tech-team ~/.claude/skills/tech-team
mkdir -p ~/.claude/agents && ln -s ~/GitHub/skills/agents/*.md ~/.claude/agents/
```

The skill must sit exactly one level under `~/.claude/skills` (`~/.claude/skills/tech-team/SKILL.md`), and agents are only discovered in `~/.claude/agents`. A nested install such as `~/.claude/skills/<owner>/tech-team` is not picked up, and the roster's `subagent_type` names won't resolve.

`tech-team/rules.md` holds Manuel's cross-repo rules, each with its claude-mem evidence, and `tech-team/stack-facts.md` what is known about each of his repos. The lead pastes the rules and the current repo's facts into every spawn prompt. The repo's own CLAUDE.md holds his rules for that repo.

Learnings are machine-local: `tech-team/learnings/` ships with an empty index and fills up as the team runs.
