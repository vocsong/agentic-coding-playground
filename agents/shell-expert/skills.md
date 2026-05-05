# shell-expert

Delegates all shell/terminal command execution to the `shell-expert` OpenCode sub-agent, backed by a stronger model. The primary model never uses the bash tool directly.

## Who / When

- Anyone using OpenCode with a fast/lightweight primary model who wants shell work handled by a stronger model
- Triggered by **any** shell, terminal, or command execution request, including:
  - Tool/CLI names: `npm`, `pip`, `git`, `docker`, `curl`, `wget`, `make`, `cargo`, `go`, `kubectl`, `helm`, `brew`, `apt`, `yarn`, `pnpm`, `poetry`, `mysql`, `psql`, `ssh`, `scp`, `rsync`, `systemctl`, `journalctl`
  - Action words: `run`, `execute`, `install`, `build`, `test`, `deploy`, `shell`, `bash`, `terminal`, `command`, `cmd`, `cli`, `script`
  - Phrases: `run npm install`, `execute this command`, `check with curl`, `build the project`, `list files with ls`, `show git status`, `what does pwd show`

## Workflow

### Phase 1: Check agent configuration

1. Verify `opencode.json` contains the `shell-expert` agent definition.
2. If missing, tell the user to add the shell-expert config (provide the JSON snippet).

### Phase 2: Delegate to shell-expert

**Critical: Never use the bash tool directly. Always delegate to the shell-expert sub-agent.**

```
Task(description="<what needs to happen>", prompt="<full instructions>", subagent_type="shell-expert")
```

This applies to every shell invocation — single commands, multi-step pipelines, read-only queries, and mutating operations.

### Phase 3: Use the result

1. Read the shell expert's structured summary.
2. Integrate the result into the main task.
3. If the task failed, follow the shell expert's recommendation for next steps.

## Important Rules

- **Never generate shell commands directly.** If you're about to use the bash tool or write a shell command, stop and delegate to `shell-expert` instead.
- **All shell tasks qualify.** Simple commands like `ls`, `pwd`, `git status` and complex pipelines like `npm install && npm test` all go through shell-expert.
- File reading, file searching, and code editing should use their dedicated tools — only terminal/shell operations go through shell-expert.
