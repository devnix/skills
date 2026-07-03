---
name: todo
description: Personal tracker for open tasks and concerns (a mixed drawer for dev/system/life). Use when the user wants to jot down a task or worry, log progress on an existing one, list what they have open, or close one. Triggers with /todo and with natural language ("note down that I'm worried about X", "what do I have open?").
---

You manage the user's task tracker in the **`~/Vaults/todo/`** vault. One Markdown note per task. Natural language ALWAYS works: when no explicit verb is given, infer the intent (add / update / list / close) from the text.

## Vault rules

- Path: `~/Vaults/todo/`. If the user (or their project config) names a different vault path, use that instead. Live tasks at the root; closed ones in `~/Vaults/todo/Archive/`.
- **Adapt to the user's language**: note prose, Log entries, your questions, replies and table headers all go in the language the user speaks to you. Only the frontmatter values stay the fixed English tokens below, so notes remain queryable.
- Plain Markdown (no community plugins or executable syntax), YAML frontmatter.
- Filename: **descriptive kebab-case slug** (`switch-power-provider.md`), no date prefix.
- **Real system dates/times**: get the date with `date +%F` and the time with `date +%H:%M`. Never invent the date.
- Don't touch `obsidian.json` or the Obsidian app; you only write `.md` files.

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

### add — create a task
`/todo add "<text>"` or NL ("note down that...").
1. Derive a kebab-case slug from the subject. If that file already exists, adjust the slug or ask.
2. Write an **in-depth Context** from what the user said. If it's terse and key context is missing (what's broken, why it matters, what solving it looks like), ask **1-2 questions** before writing; don't invent details.
3. Infer `tags` (dev/system/life/idea) and `priority` (medium by default) from the content; when unsure about priority, leave `medium`.
4. Write `~/Vaults/todo/<slug>.md` with the frontmatter, the Context and the first Log entry; confirm the path.

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
3. **Move** the file to `~/Vaults/todo/Archive/<slug>.md`.
4. Confirm.

## Notes

- Reopening, re-reading or rewriting the detail have no shortcut: do it via natural language (e.g. move the file out of `Archive/` and set `status: open`).
- On create/update, reply in 1-2 lines with the path and the change; don't dump the whole file unless asked.
