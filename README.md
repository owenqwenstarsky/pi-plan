# Pi plan mode extension

A local Pi extension that adds a conservative, read-only planning mode and a model-facing question tool.

## What it does

- `/plan <optional prompt>` enables plan mode and sends the prompt to Pi.
- `/plan` toggles plan mode when no prompt is supplied.
- `/plan off` explicitly leaves plan mode; `/plan status` reports the current state.
- Write tools are removed from the active tool set while plan mode is enabled.
- `bash` is limited to a conservative allowlist of read-only commands.
- `ask_questions` lets the model ask **one to three** focused multiple-choice questions.
  - Questions appear one at a time.
  - Every question includes a “Type your own response” choice.
  - `←`, `Shift+Tab`, or `Backspace` revises the previous question.
  - Answers are reviewed before submission.
- When a numbered plan is complete, Pi prompts with:
  - **Implement the plan** — exits plan mode and sends the implementation request.
  - **Make changes** — opens an editor for feedback, revises the plan, and stays in plan mode.
- Mode state and the pre-plan tool set are persisted with the session branch.

## Files

- `index.ts` — command, tool loadout, read-only enforcement, prompt context, and session state.
- `ask-questions.ts` — sequential custom TUI and `ask_questions` tool.
- `utils.ts` — session state and bash safety policy.

## Install from GitHub

Install the extension from its GitHub repository:

```bash
pi install https://github.com/owenqwenstarsky/pi-plan
```

## Try it without installing

From this directory, load it explicitly:

```bash
pi --extension ./index.ts
```

Then try:

```text
/plan Investigate how authentication works and propose a safe refactor
```

The extension is not copied to `~/.pi/agent/extensions` and does not modify Pi's configuration.

## Design notes

Plan mode is intentionally conservative. A shell command is allowed only when it starts with a known read-only command family, is a single simple command, and contains no destructive token. The injected planning prompt tells the model to prefer Pi's built-in inspection tools and never compose bash commands with `&&`, pipes, redirects, or command substitution. If a command is rejected, leave plan mode with `/plan off` rather than trying to work around the guard.

The question UI requires Pi's interactive TUI. In RPC, JSON, and print modes, the tool returns a useful cancellation/error result instead of attempting to access terminal controls.
