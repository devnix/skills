# Launcher example: Orca

Opens the task's conversation as a tab inside Orca, in the Orca-managed worktree named by `resume_worktree`. Pick it when the user works through Orca's worktrees.

## Commands

- **Readiness**: `orca status` reports `runtimeState: ready` when the app answers. Otherwise run `orca open`, which waits for the runtime.
- **Worktree paths**: `orca worktree list` prints each worktree's absolute path; use that path in `resume_worktree`.
- **New conversation** (`<session>` empty):

  ```sh
  orca agent launch --agent <agent> --worktree "path:<worktree>" --prompt "<prompt>" --focus
  ```

- **Existing conversation**: `orca agent launch` has no resume option, so open a terminal running the agent's own resume command:

  ```sh
  orca terminal create --worktree "path:<worktree>" --title "<slug>" \
    --command "<agent resume command> '<prompt>'" --focus
  ```

  The resume command belongs to the agent (`claude --resume <session>`, `codex resume <session>`; check `<agent> --help` for others). If the agent ignores a prompt passed on resume, send it afterwards with `orca terminal send`.

## Gotchas

- Claude Code files each conversation under the directory it started in: `<worktree>` must be that same directory or `--resume` finds nothing.
- Scripts run outside the user's shell (a systemd service, cron) may lack the `orca` CLI on `PATH`. Find it with `command -v orca` in the user's shell and use the absolute path or extend `PATH` in the script.

## `TODO.md` block

```markdown
- Tool: Orca. Run `orca status`; if the runtime isn't ready, run `orca open`.
- New conversation: `orca agent launch --agent <agent> --worktree "path:<worktree>" --prompt "<prompt>" --focus`
- Existing conversation: `orca terminal create --worktree "path:<worktree>" --title "<slug>" --command "<agent> --resume <session> '<prompt>'" --focus`
```
