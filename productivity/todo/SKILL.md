---
name: todo
description: Personal tracker for open tasks and concerns (a mixed drawer for dev/system/life). Use when the user wants to jot down a task or worry, log progress on an existing one, list what they have open, close one, or set up where their tasks live. Triggers with /todo and with natural language ("note down that I'm worried about X", "what do I have open?").
argument-hint: "[setup|add|update|list|close] [task] [text] — or natural language"
---

You manage the user's task tracker in a Markdown vault. One note per task. Natural language ALWAYS works: when the user gives no explicit verb, infer the intent (setup / add / update / list / close) from the text.

## Vault rules

- **Vault location**: declared in the user's memory file `~/.claude/CLAUDE.md` (a line naming the tasks folder and this skill). If it's not already in your context, read `~/.claude/CLAUDE.md`; if nothing is declared there, run `setup` before any other verb. Below, `<vault>` means that folder.
- Live tasks at the vault root; closed ones in `<vault>/Archive/`.
- **Adapt to the user's language**: note prose, Log entries, questions, replies, and table headers all go in the language the user speaks to you. Only the frontmatter values stay the fixed English tokens below, so notes remain queryable.
- Plain Markdown (no community plugins or executable syntax), YAML frontmatter.
- Filename: **descriptive kebab-case slug** (`switch-power-provider.md`), no date prefix.
- **Real system dates/times**: get the date with `date +%F` and the time with `date +%H:%M`. Never invent the date.
- Write only `.md` files; never touch `obsidian.json` or the Obsidian app.

### Note format

```markdown
---
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: open           # open | in-progress | blocked | done
tags: [dev]            # dev | system | life | idea | ...
priority: medium       # high | medium | low
---

## Context

<in-depth detail: what it is, why it worries the user, what remains to solve. Free prose, no fixed subsections.>

## Log

### YYYY-MM-DD HH:MM
<most recent entry — newest on top>
```

## Verbs

### setup — configure where tasks live
`/todo setup` or NL ("set up my task tracker") — and run it automatically when no vault location is declared.
1. Ask the user where their tasks should live, suggesting `~/Vaults/todo/` as the default.
2. Create the folder and its `Archive/` subfolder if missing.
3. Record the choice in the user's memory file `~/.claude/CLAUDE.md` (create the file if needed): one line stating the vault path and that the `todo` skill manages it. Update any existing declaration rather than adding a second.
4. Confirm the path and where it was recorded.

### add — create a task
`/todo add "<text>"` or NL ("note down that...").
1. Derive a kebab-case slug from the subject. If that file already exists, adjust the slug or ask.
2. Write an **in-depth Context** from what the user said. If it's terse and key context is missing (what's broken, why it matters, what solving it looks like), ask **1-2 questions** before writing; don't invent details.
3. Infer `tags` (dev/system/life/idea) and `priority` (medium by default) from the content.
4. Write `<vault>/<slug>.md` with the frontmatter, the Context and the first Log entry; confirm the path.

### update — log progress
`/todo update <task> "<text>"` or NL ("update the freeze one: ...").
1. Resolve `<task>` by fuzzy match over root slugs/titles. If several candidates match, ask which. If none, offer to create it with `add`.
2. Insert a **new entry on top** of `## Log` (right under the heading), with `### YYYY-MM-DD HH:MM`.
3. Refresh `updated`.
4. If the progress implies a status change (unblocked, started…), update `status` (open/in-progress/blocked) and say so.

### list — view active tasks
`/todo list`, bare `/todo`, or NL ("what do I have open?").
- Read the frontmatter of the root notes (exclude `Archive/`), filter `status != done`, sort by `updated` desc.
- Print a **Markdown table** with columns: Task (slug) · Status · Priority · Tags · Updated.

### close — close a task
`/todo close <task>` or NL ("close X", "X is done now").
1. Resolve `<task>` as in `update`.
2. Set `status: done`, refresh `updated`, add a closing Log entry (what got resolved).
3. **Move** the file to `<vault>/Archive/<slug>.md`.
4. Confirm.

## Notes

- Reopening, re-reading, or rewriting the detail has no shortcut: do it via natural language (e.g. move the file out of `Archive/` and set `status: open`).
- On create/update, reply in 1-2 lines with the path and the change; don't dump the whole file unless asked.
