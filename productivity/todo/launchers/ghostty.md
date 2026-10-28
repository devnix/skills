# Launcher example: Ghostty

Opens the task's conversation in a new [Ghostty](https://ghostty.org) window, with the agent started in `resume_worktree`. Pick it when the user runs agents in a plain terminal.

## Commands

- **New conversation** (`<session>` empty):

  ```sh
  ghostty --working-directory="<worktree>" -e <agent> "<prompt>"
  ```

- **Existing conversation**: run the agent's own resume command (`claude --resume <session>`, `codex resume <session>`; check `<agent> --help` for others):

  ```sh
  ghostty --working-directory="<worktree>" -e <agent resume command> "<prompt>"
  ```

## Gotchas

- Claude Code files each conversation under the directory it started in: `<worktree>` must be that same directory or `--resume` finds nothing.

## `TODO.md` block

```markdown
- Tool: Ghostty, one new window per task.
- New conversation: `ghostty --working-directory="<worktree>" -e <agent> "<prompt>"`
- Existing conversation: `ghostty --working-directory="<worktree>" -e <agent> --resume <session> "<prompt>"`
```
