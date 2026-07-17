---
name: read-last-handover
description: MANDATORY trigger any time the user says "/read-last-handover", "read last handover", "resume last handover", "continue from handover", "pick up where I left off", or otherwise indicates they want to resume from the most recent saved session handover. Reads `~/.claude/handovers/latest.md` and continues the work described there. Companion skill to `handovers` (which writes the file).
---

# Read Last Handover — Resume Previous Session

The user has a saved handover from a previous session at `~/.claude/handovers/latest.md`. This skill loads it and continues the work — saving ~500 tokens vs. pasting the handover into chat.

## Step 1 — Read the global handover file

Use the Read tool on `~/.claude/handovers/latest.md`.

If the file doesn't exist:
1. Tell the user: "No handover found at `~/.claude/handovers/latest.md`. Was one ever written? Use `/handovers` at the end of a session to create one."
2. Stop. Don't fabricate context.

## Step 2 — Also check for a project-local handover

If the current working directory contains `./.claude/HANDOVER.md`, read that too — it's the per-project snapshot and may be more specific than the global one.

If both exist and disagree (e.g., different projects), prefer the project-local file and tell the user which one you used.

## Step 3 — Confirm and resume

In 2–4 short lines, tell the user:
1. Which file(s) you loaded
2. The Goal from the handover (one line)
3. The first item in Next (so they know what you're about to do)

Then proceed with that first Next action. Don't re-paste the full handover — they wrote it, they don't need it read back at them.

Example:
```
Loaded ~/.claude/handovers/latest.md.
Goal: <goal line>
Picking up at: <first Next item>
```

Then start working on it.

## What NOT to do

- Don't dump the whole handover back into chat — defeats the cost savings.
- Don't ask "should I continue?" — the user already said `/read-last-handover`, that IS the continue signal.
- Don't read other files in `~/.claude/handovers/` unless the user explicitly asks for a dated archive.
