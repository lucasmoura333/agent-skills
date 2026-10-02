# Cross-harness memory layout

How to run several coding agents (Claude Code, opencode, Codex CLI, Antigravity, ...) on one machine with **one set of instructions, one skills directory, and one memory architecture** — while keeping each session's context small.

## Layers

| Layer | Canonical location | Shared with other harnesses via |
| --- | --- | --- |
| Pinned instructions | `~/.claude/CLAUDE.md` | symlinks: `~/.codex/AGENTS.md`, `~/.config/opencode/AGENTS.md`; per-harness extras in `~/AGENTS.md` |
| Skills | `~/.claude/skills/` | `~/.agents/skills` symlink; opencode auto-loads `~/.claude/skills/` and `~/.agents/skills/` |
| Working memory | `savepoints/` | the [savepoint](../skills/savepoint/SKILL.md) skill; written at the end of a block, consumed by the next session |
| Session archive | `<state-dir>/agent-sessions/` | `SessionEnd` hooks pipe every harness's transcript into one normalizer (sanitized, gzipped) |
| Long-term notes | external notes vault | periodic sync; structured facts optionally in a knowledge-graph MCP server |

```
       ┌────────────────────────────────────────────┐
       │        ~/.claude/CLAUDE.md  (pinned)       │
       └───────┬──────────────┬──────────────┬──────┘
               │ symlink      │ symlink      │
        ~/.codex/AGENTS.md   ~/.config/opencode/AGENTS.md   ~/AGENTS.md
               │              │              │
   Claude Code    opencode      Codex CLI     Antigravity
        │              │              │              │
        └──── SessionEnd hooks ───────┴──────────────┘
                       │
             sanitized archive (write-only)
```

## Why archives are write-only

Every `SessionEnd` hook exports the transcript into a normalized archive, but agents **never read it back by default**. Replaying old sessions would inject huge, stale context into new ones — the opposite of the [savepoint](../skills/savepoint/SKILL.md) strategy. The archive is for humans, auditing, and explicit lookups only.

## Setup

```bash
# one canonical instructions file, symlinked into every harness
ln -sf ~/.claude/CLAUDE.md ~/.codex/AGENTS.md
ln -sf ~/.claude/CLAUDE.md ~/.config/opencode/AGENTS.md

# one canonical skills directory
mkdir -p ~/.agents && ln -sfn ~/.claude/skills ~/.agents/skills
```

Session archival hooks (Claude Code `settings.json`, Codex `hooks.json`):

```json
{
  "hooks": {
    "SessionEnd": [
      { "hooks": [ { "type": "command", "command": "<your-normalizer> --agent <name>" } ] }
    ]
  }
}
```

A normalizer is a small script that reads the transcript path from the hook payload, sanitizes it, and appends one line per session to an `index.jsonl`. Keep it per-user and out of the repo: it contains machine-specific paths and rules.

## Would this work for you?

The pattern is portable; the specifics are not. Start with the two symlinks and the savepoint loop — they deliver most of the context savings with none of the plumbing. Add the archival pipeline when you actually need audit or search across sessions.
