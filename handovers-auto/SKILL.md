---
name: handovers-auto
description: Use this skill when the user says "/handovers-auto", "auto handover", "handover and push", "handoff and push", or otherwise wants a session handover that is automatically committed AND pushed to git so a Remote (cloud) session can pick it up. Same as the `handovers` skill, but additionally commits and pushes `./.claude/HANDOVER.md` to the project repo. Use plain `handovers` instead when the user does NOT want an automatic git push (e.g. repos with deploy-on-push). Writes `~/.claude/handovers/latest.md` (global) and `./.claude/HANDOVER.md` (in-project), and displays a chat summary.
---

# Handovers (Auto-Push) — Cross-Environment Session Handover

Same handover as the `handovers` skill, plus it **commits and pushes** the in-project
`./.claude/HANDOVER.md` so a **Remote / cloud session** (which never sees your local
`~/.claude/handovers/latest.md`) can read it after cloning the repo.

Use this variant only when the user explicitly wants the push (`/handovers-auto`). The user
accepted that pushing can trigger CI/deploy on repos with deploy-on-push — that is the
intended tradeoff of this skill. If unsure, use plain `handovers` instead.

## Step 1 — Gather the handover content

Collect before writing:

1. **Goal** — what the user is doing this session
2. **Done** — concrete changes, decisions, files edited, commands run
3. **State** — current status (passing? broken? mid-edit?)
4. **Next** — immediate 1–3 next actions
5. **Open questions / blockers** — anything waiting on user or external systems
6. **Key paths** — files, branches, URLs, IDs the next session needs
7. **Gotchas** — env quirks, ordering constraints, things already tried

Omit empty sections — don't pad.

**Token budget: aim for under ~500 words total.** Bullets, not prose. No narrative
explanations of how something was diagnosed or built — state the outcome. The one exception:
decisions likely to be revisited get a single-line "why". Always keep **State** (half-open
items: uncommitted files, live ports, deployed-but-untested) — that's what a cold session
misreads most.

## Step 2 — Build the handover markdown

```
# Handover — <short title> (<YYYY-MM-DD>)

## Goal
<1–2 sentences>

## Done
- <bullet>

## State
<1–3 sentences>

## Next
1. <action>

[Include Open questions / blockers, Key paths, Gotchas only if non-empty]
```

Concrete beats vague: `src/auth/login.ts:42`, not "the login file".

## Step 3 — Write the files

Use the date from system context as `YYYY-MM-DD`.

1. **Always write:** `~/.claude/handovers/latest.md` — the global "what was I just doing?"
   pointer for local sessions. (In a Remote container this won't persist across sessions;
   that's fine — the git-tracked file below is the cross-environment channel.)

2. **If in a project directory** (detect `.git`, `package.json`, `go.mod`, etc.): also write
   `./.claude/HANDOVER.md` — the durable per-project snapshot.

3. **Do NOT write a dated archive** unless the user explicitly asks.

## Step 4 — Commit and push `./.claude/HANDOVER.md` (the auto part)

Only if you wrote `./.claude/HANDOVER.md` in Step 3 and the directory is a git repo.

1. **Make sure it isn't ignored.** Run `git check-ignore .claude/HANDOVER.md`. If it prints
   the path (i.e. it *is* ignored), tell the user the file is gitignored so Remote won't see
   it, and stop the git step (still report the local writes). Do not `-f` force-add without
   asking — the ignore may be intentional.

2. **Stage ONLY the handover file** — never sweep up the user's other uncommitted work:
   ```
   git add .claude/HANDOVER.md
   ```

3. **Commit just that file:**
   ```
   git commit -m "Update session handover" -- .claude/HANDOVER.md
   ```
   If nothing changed (identical content), git will report "nothing to commit" — that's fine,
   skip the push.

4. **Push to the current branch:**
   ```
   git push
   ```
   - If push fails because there's no upstream, run `git push -u origin HEAD`.
   - If push is rejected (remote ahead), do NOT force. Report the rejection and let the user
     reconcile — a handover is never worth a force-push.

5. **If the repo has deploy-on-push** (Netlify, Vercel, GH Pages, CI/CD): the push may kick
   off a build. That is the accepted behavior of this skill, but mention it in the confirm
   line so the user isn't surprised (e.g. "pushed to main — this may trigger a Netlify build").

## Step 5 — Confirm in chat

One line stating what happened, e.g.:
```
Handover saved to ~/.claude/handovers/latest.md and ./.claude/HANDOVER.md;
committed + pushed to <branch> (a4f9c21). Remote sessions can now read it.
```
If the git step was skipped (not a repo / ignored / nothing to commit), say why.

## Step 6 — Next session picks it up cheaply

- **Local:** `read ~/.claude/handovers/latest.md and continue`, or `/read-last-handover`.
- **Remote:** the session clones the repo, so `./.claude/HANDOVER.md` is already there —
  `/read-last-handover` checks it automatically.

## What NOT to do

- Don't stage or commit anything other than `.claude/HANDOVER.md`.
- Don't force-push, ever.
- Don't force-add a gitignored HANDOVER.md without asking.
- Don't write a handover if the session has done nothing — tell the user there's nothing to
  capture and ask what to save.
- Don't overwrite the project HANDOVER.md if there's uncommitted work the user might lose that
  belongs *in* the handover — capture it first.
