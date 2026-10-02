# agent-skills

Small collection of [Agent Skills](https://github.com/anthropics/skills) for coding agents — Claude Code, [opencode](https://opencode.ai/docs/skills/), and any tool that loads `SKILL.md` files.

## Skills

| Skill | What it does |
| --- | --- |
| [**savepoint**](skills/savepoint/SKILL.md) | Writes a compact handoff note so you can end a long session and start fresh — cutting the context (and cost) re-sent on every message. |

## Install

Skills are plain folders with a `SKILL.md` inside. Copy or symlink the folder into the location your agent scans:

```bash
git clone https://github.com/lucasmoura333/agent-skills.git

# Claude Code
mkdir -p ~/.claude/skills && cp -r agent-skills/skills/savepoint ~/.claude/skills/

# opencode (global)
mkdir -p ~/.config/opencode/skills && cp -r agent-skills/skills/savepoint ~/.config/opencode/skills/
```

- opencode also auto-loads skills from `~/.claude/skills/` and `~/.agents/skills/`.
- Project-scoped installs go in `.opencode/skills/` (opencode) or `.claude/skills/` (Claude Code).
- Restart the agent after installing.

## How savepoint works

1. Work one block in one session.
2. At the end, ask for a savepoint — the agent writes `savepoints/YYYY-MM-DD-<slug>.md`.
3. Start a new session and resume from that file.

The note holds state, decisions, pending items, and sources of truth. The next session starts from a small context instead of replaying the entire history.

## Results

Measured on one heavy user's real opencode telemetry (Sep 24 – Oct 2, 2026; ~6,000 assistant messages; intervention: savepoint + fresh sessions per block):

| Metric | Before | After | Change |
| --- | ---: | ---: | ---: |
| Cost per assistant message | $0.00255 | $0.00131 | −49% |
| Avg context per message | 389,953 tok | 137,557 tok | −65% |
| Messages per session | 142 | 88 | −38% |
| Same-model control (deepseek-flash) | $0.00277/msg | $0.00079/msg | −71% |

Caveats: 48h post-intervention sample, mixed workloads and models — treat as directional, not a controlled benchmark.

## Docs

- [Cross-harness memory layout](docs/harness-memory.md) — running several agents with one instructions file, one skills directory, and a write-only session archive.

## License

MIT — see [LICENSE](LICENSE).
