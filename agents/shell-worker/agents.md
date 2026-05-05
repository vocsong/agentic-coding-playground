# Shell Worker Agent

## Purpose

`shell-worker` is an OpenCode sub-agent for complex shell command generation and execution. It is intended for primary models that should avoid hand-writing risky, multi-step, or highly specific terminal commands directly.

The agent delegates those tasks to a stronger model-backed child session, such as `openai/gpt-5.5`, while keeping ordinary shell calls available to the primary model.

## Problem It Solves

Primary coding agents often need to run terminal workflows that are more complex than a single obvious command. Examples include dependency installs, chained setup commands, multi-step refactors, generated scripts, and failure recovery after a command exits non-zero.

Instead of overriding the `bash` tool, this pattern delegates at the agent level:

1. The primary model identifies that a shell task is complex.
2. The primary model calls OpenCode's `Task` tool with `subagent_type: "shell-worker"`.
3. OpenCode starts a child session using the `shell-worker` agent configuration.
4. The child agent generates, runs, debugs, and summarizes the terminal work.

## Intended Users

This agent is for OpenCode users who run a fast or cheaper primary model but want complex shell work handled by a stronger model.

It is especially useful when the primary model is configured as something lightweight, while the shell worker is configured with a stronger model such as `openai/gpt-5.5`.

## Inputs

The primary model should provide:

- A concise task description.
- Full shell-related instructions.
- Relevant constraints, such as whether installs are allowed, whether destructive operations are forbidden, or which directory to use.
- Any expected success condition, such as passing tests or producing a specific artifact.

Example `Task` call:

```json
{
  "description": "Install dependencies",
  "prompt": "Run npm install in the current project and resolve any peer dependency conflicts without using force unless necessary. Summarize what changed.",
  "subagent_type": "shell-worker"
}
```

## Outputs

The shell worker should return:

- Commands executed.
- Important command results.
- Failures encountered and how they were handled.
- Any files or dependencies changed.
- A concise final summary for the primary model.

Success means the shell task is completed safely, or the agent clearly reports why it could not complete the task.

## OpenCode Configuration

Add this to `opencode.json` in the project where you want to use the agent:

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

Adjust `model` to match the provider and model configured in your OpenCode environment.

## Delegation Instructions

Add guidance like this to the project's `AGENTS.md`:

```markdown
## Shell Command Delegation

For complex shell tasks, delegate to the `shell-worker` sub-agent instead of generating the command directly.

Use this for multi-step operations, chained commands with `&&`, installs, refactors, long commands, failure recovery, and terminal workflows where command correctness matters.

Invoke it with:

`Task(description="<what needs to happen>", prompt="<full instructions>", subagent_type="shell-worker")`

Do not delegate simple commands such as `pwd`, `date`, `ls`, `git status`, or a single obvious test command.
```

## Workflow

1. Determine whether the shell task is complex enough to delegate.
2. If simple, use the normal shell tool directly.
3. If complex, call `Task` with `subagent_type: "shell-worker"`.
4. Provide complete instructions, including constraints and success criteria.
5. Let the shell worker run commands and handle recoverable failures.
6. Use the shell worker's summary to continue the main task.

## Good Delegation Examples

- Install dependencies and resolve peer dependency conflicts.
- Run a multi-step migration command sequence.
- Generate a one-off shell script and execute it.
- Run a test command, inspect failures, install missing tools, and retry.
- Perform terminal-driven refactors that require several commands.
- Diagnose environment setup problems involving package managers, shells, or CLIs.

## Poor Delegation Examples

- `date`
- `pwd`
- `git status`
- `npm test` when no failure handling or extra logic is needed.
- Reading a specific file, which should use the normal file-reading tools.

## Edge Cases And Boundaries

- Destructive commands must be avoided unless explicitly requested by the user.
- Commands involving secrets, credentials, `.env` files, or tokens must be handled cautiously and not exposed in summaries.
- Long-running commands should use reasonable timeouts and report if they cannot complete.
- Package installs may change lockfiles and should be summarized.
- If the requested command is ambiguous, the shell worker should ask for clarification or choose the safest minimal path.
- The agent should not replace normal file search, file read, or targeted edit tools.

## Bash Tool Override

No custom `bash` override is required for this pattern. Delegation happens through OpenCode's model and agent layer, not by intercepting shell execution.

If a project already has `.opencode/tools/bash.ts`, it can usually be removed or kept as a pass-through. Only route all bash calls through this agent if the project intentionally wants every shell invocation handled by the stronger model.
