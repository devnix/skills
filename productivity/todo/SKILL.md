---
name: todo
description: Personal tracker for open tasks and concerns (a mixed drawer for dev/system/life). Use when the user wants to jot down a task or worry, log progress on an existing one, list what they have open, close one, resume one in its worktree or conversation, or set up where their tasks live. Triggers with /todo and with natural language ("note down that I'm worried about X", "what do I have open?", "pick up the X task").
argument-hint: "[setup|add|update|list|close|resume] [task] [text] — or natural language"
---

You manage the user's task tracker in a Markdown vault. One note per task. Natural language ALWAYS works: when the user gives no explicit verb, infer the intent (setup / add / update / list / close / resume) from the text.

## Personal config: `TODO.md`

The facts specific to this user and machine live in their `TODO.md`, never in this skill: the vault path, the launcher that opens a task's conversation, and the notifier that delivers reminders.

- **Pointer**: the agent's global instructions file holds one line naming the `TODO.md` path (see `setup`). Look for it in your context; if the file isn't loaded, read the global instructions file of the agent you run in (table in `setup`).
- **Every invocation**: read `TODO.md` before any verb. Below, `<vault>` means the path it declares.
- **No pointer, or the file it names is missing**: run `setup` first, then carry on with the user's request.

## Vault rules

- Task notes are the `.md` files at the vault root that carry the frontmatter below; closed ones live in `<vault>/Archive/`. `TODO.md` and any agent instructions file (`CLAUDE.md`, `AGENTS.md`) are config, not tasks.
- **Adapt to the user's language**: note prose, Log entries, questions, replies, and table headers all go in the language the user speaks to you. Only the frontmatter keys and values stay the fixed English tokens below, so notes remain queryable.
- Plain Markdown (no community plugins or executable syntax), YAML frontmatter.
- Filename: **descriptive kebab-case slug** (`switch-power-provider.md`), no date prefix.
- **Real system dates/times**: get the date with `date +%F` and the time with `date +%H:%M`. Never invent the date.
- Write only `.md` files; leave the vault's app settings (e.g. Obsidian's `.obsidian/`) untouched.

### Note format

```markdown
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: open           # open | in-progress | blocked | done
tags: [dev]            # dev | system | life | idea | ...
priority: medium       # high | medium | low
due: YYYY-MM-DD        # optional — deadline (date only)
remind: YYYY-MM-DDTHH:MM   # optional — local date and time to alert the user
resume_worktree: /abs/path # optional — directory or worktree where the task's work happens
resume_session: <id>       # optional — agent conversation to reopen
resume_prompt: "<text>"    # optional — what the resumed agent should do first
---

## Context

<in-depth detail: what it is, why it worries the user, what remains to solve. Free prose, no fixed subsections.>

## Log

### YYYY-MM-DD HH:MM
<most recent entry — newest on top>
```

Omit every optional key that has no value. Set each one only from what the user states, resolving relative dates and times against the real clock:

- `due`: a deadline ("for Friday", "before the end of the month").
- `remind`: a moment to be alerted ("remind me tomorrow at 8:15"). Recording it is the whole job here; delivery belongs to the notifier `TODO.md` declares. If it declares none, say once that nothing will fire.
- `resume_*`: the user ties the task to a directory, a worktree, or a conversation. Use absolute paths. Record `resume_session` only from an id the user gives or the agent's own session id when it is known for certain.

## Verbs

### setup — install the config
`/todo setup`, NL ("set up my task tracker"), and automatically when the pointer or its `TODO.md` is missing.

1. **Find the global instructions file** of the agent you run in:

   | Agent | File |
   |---|---|
   | Claude Code | `$CLAUDE_CONFIG_DIR/CLAUDE.md` when the variable is set, else `~/.claude/CLAUDE.md` |
   | Codex | `~/.codex/AGENTS.md` |
   | opencode | `~/.config/opencode/AGENTS.md` |
   | pi | `~/.pi/agent/AGENTS.md` |
   | other or unsure | ask the user |

2. **Vault**: ask where tasks should live. Default: a path an older declaration in that file already names, else `~/Vaults/todo/`. Create the folder and `Archive/` if missing.
3. **Reuse**: if `<vault>/TODO.md` already exists (another agent installed it), keep it and skip to step 6.
4. **Launcher**: ask how `resume` should open a task's conversation. Offer the examples in [`launchers/`](launchers/) (read the one the user picks), "other" (they describe it), or "none".
5. **Write `<vault>/TODO.md`** from [`TODO.template.md`](TODO.template.md): the vault path, the launcher's `TODO.md` block adapted to the user's agent, and `Notifier: none` unless they describe one.
6. **Write the pointer** into the global instructions file (create the file if needed), as a plain path:

   ```markdown
   - `todo` skill: personal config in `<vault>/TODO.md`. Read it only when the `todo` skill runs.
   ```

   Keep the path plain text. An `@`-import would load the file into every session. If the pointer line already exists, update its path in place. Leave every other line as it is, including older lines that mention the vault: they often carry the user's own preferences.
7. If the user runs other agents, offer to add the same pointer to their files.
8. Confirm the `TODO.md` path and each file that got the pointer.

### add — create a task
`/todo add "<text>"` or NL ("note down that...").
1. Derive a kebab-case slug from the subject. If that file already exists, adjust the slug or ask.
2. Write an **in-depth Context** from what the user said. If it's terse and key context is missing (what's broken, why it matters, what solving it looks like), ask **1-2 questions** before writing; don't invent details.
3. Infer `tags` (dev/system/life/idea) and `priority` (medium by default) from the content.
4. Set the optional keys the user stated (see Note format).
5. Write `<vault>/<slug>.md` with the frontmatter, the Context and the first Log entry; confirm the path.

### update — log progress
`/todo update <task> "<text>"` or NL ("update the freeze one: ...").
1. Resolve `<task>` by fuzzy match over root slugs/titles. If several candidates match, ask which. If none, offer to create it with `add`.
2. Insert a **new entry on top** of `## Log` (right under the heading), with `### YYYY-MM-DD HH:MM`.
3. Refresh `updated`.
4. If the progress implies a status change (unblocked, started…), update `status` (open/in-progress/blocked) and say so.
5. If the user states, moves or drops a deadline, a reminder ("snooze it an hour"), or a worktree/session, set, update or remove the key and say so.

### list — view active tasks
`/todo list`, bare `/todo`, or NL ("what do I have open?").
- Read the frontmatter of the root notes (exclude `Archive/`), filter `status != done`, sort by `updated` desc.
- Print a **Markdown table** with columns: Task (slug) · Status · Priority · Tags · Due · Remind · Updated. Leave Due and Remind blank when the key is absent; mark them when today or already past.

### close — close a task
`/todo close <task>` or NL ("close X", "X is done now").
1. Resolve `<task>` as in `update`.
2. Set `status: done`, refresh `updated`, drop `remind`, add a closing Log entry (what got resolved).
3. **Move** the file to `<vault>/Archive/<slug>.md`.
4. Confirm.

### resume — reopen a task's work
`/todo resume <task>` or NL ("pick up the X task").
1. Resolve `<task>` as in `update`.
2. Build the prompt: `Resume the task in <absolute note path>: read its Context and Log, then continue with the next step.`, followed by `resume_prompt` when set.
3. Run the launcher from `TODO.md` with `resume_worktree` (default: the user's home), `resume_session` (empty means a new conversation), and the prompt.
4. No launcher (or "none"): print the commands that would open it by hand (`cd` into the directory, then the agent's resume or start command).
5. Add a Log entry ("Resumed in …") and confirm in one line.

## Notes

- Reopening, re-reading, or rewriting the detail has no shortcut: do it via natural language (e.g. move the file out of `Archive/` and set `status: open`).
- On create/update, reply in 1-2 lines with the path and the change; don't dump the whole file unless asked.
