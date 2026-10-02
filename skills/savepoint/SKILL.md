---
name: savepoint
description: Write a compact savepoint of the current work block and recommend starting a new session, so context stays small and token cost stays down. Use when a work block ends, when switching topics, or when the user asks for a savepoint.
license: MIT
metadata:
  version: "1.0.0"
---

# Savepoint

A savepoint is a short handoff note that captures the state of one work block. It is the **source of truth for resuming** — the next session must not depend on the previous session's chat history.

**Why it saves money:** agent cost scales with the context re-sent on every message, and a long session re-sends its entire history every turn. A savepoint plus a fresh session resets that context; the note carries the state forward for a tiny fraction of the tokens.

## The loop

1. Work one block in one session.
2. Write a savepoint (this skill).
3. Start a new session (`/new`) and resume from the file.

## Where to write

Default: `savepoints/YYYY-MM-DD-<slug>.md` in the current working directory.
If the project documents another location (`AGENTS.md`, `CLAUDE.md`, README) or a `docs/savepoints/` directory already exists, follow that instead.

## Steps

1. Pick the block's theme and a short kebab-case slug (e.g. `checkout-refactor`). Ask if it is ambiguous.
2. Collect state surgically — without inflating context: `git status --short | head`, the current branch, and only the specific files or docs needed. Do not run broad scans or read large files just to fill the note.
3. Write the file. Copy this frontmatter and adjust:

   ```yaml
   ---
   id: savepoint-YYYY-MM-DD-<slug>
   title: <short human title>
   date: YYYY-MM-DD
   type: savepoint
   status: active
   tags: [<tag>, <tag>]
   generated_at: <ISO-8601 timestamp>
   ---
   ```

   Add project-specific fields when useful.
4. Body — terse bullets, no narration:
   - **State** — one paragraph.
   - **Done** — bullets with `file:line` when technical.
   - **Pending / Blocked** — what comes next, and what stops it.
   - **Bottlenecks** — risks, flaky steps, unknowns.
   - **Sources of truth** — files, branches, docs, links.
5. If the project keeps an index (e.g. `savepoints/_INDEX.md`), update it. If none exists, skip — do not create one unprompted.
6. End with exactly: `Savepoint written to <path>. Recommend /new for the next block.`

## Rules

- Never invent state; mark missing data as `(to confirm)`.
- If the note is longer than one page, you are narrating — cut it.
- Write in the language the user is using.

See `examples/checkout-refactor.md` for a filled-in example.
