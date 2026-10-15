@AGENTS.md

## Claude Code specifics

- **Skills** (`.claude/skills/`): `plan-stage`, `orchestrate-wave`, `run-session`, `invariant-review`. For platform work also use the `harmonyos-app-lifecycle` skill (github.com/JohnnnyTang/harmonyos-dev-skills).
- **Parallel sessions:** dispatch with the Agent tool, one background agent per session brief with worktree isolation, all in one message. Never predict or fabricate a background agent's result.
