# Shell Worker Agent

OpenCode sub-agent for delegating complex shell command generation and execution to a stronger model.

See [`agents.md`](agents.md) for the full workflow, configuration, and delegation guidance.

## Quick Install

Add this to your project's `opencode.json`:

```json
{
  "agent": {
    "shell-worker": {
      "model": "openai/gpt-5.5",
      "description": "Generates and executes complex shell commands. Use for multi-step terminal tasks, refactors, installs, chained commands.",
      "mode": "subagent",
      "temperature": 0.2,
      "steps": 20
    }
  }
}
```

Then add delegation guidance to `AGENTS.md`:

```markdown
## Shell Command Delegation

For complex shell tasks, delegate to the `shell-worker` sub-agent instead of generating the command directly:

`Task(description="<what needs to happen>", prompt="<full instructions>", subagent_type="shell-worker")`
```

Use it for installs, chained commands, multi-step terminal work, refactors, and shell failure recovery. Keep simple commands on the normal shell tool.
