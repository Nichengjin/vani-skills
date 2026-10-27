---
name: using-rws
description: Runs build, test, training, Docker, and GPU commands from a local Git workspace on a configured SSH Linux host through the rws CLI. Use when work should execute on AutoDL or another remote machine while source files and the Coding Agent remain local, or when diagnosing, monitoring, canceling, or pulling an rws task.
---

# Using rws

Keep the Agent and authoritative Git workspace local. Use `rws` locally to send an immutable snapshot to Linux; do not edit the remote task checkout manually. The remote shim installs automatically.

## Workflow

1. Require an explicit SSH target. Prefer `--target`; otherwise use `RWS_TARGET` or repository-local `git config rws.target`. Never silently run the requested command locally after an rws failure. Global flags go before the command name: `rws --json --target <host> run …`.
2. Check a new target once:

   ```sh
   rws --target gpu-box doctor
   ```

3. Run commands from the local Git workspace:

   ```sh
   rws --target gpu-box run -- go test ./...
   rws --target gpu-box run --detach -- make build
   ```

   Pass argv directly after `--`. For pipes, redirects, variables, or multiple shell commands, use `run -- sh -lc '...'` explicitly. Prefer `--detach` for anything longer than a minute so the task survives the chat ending.

4. Save the returned task ID (`tsk_` + 32 hex). Inspect and control the durable task with:

   ```sh
   rws status <task-id>
   rws logs <task-id> --follow
   rws cancel <task-id>
   rws pull <task-id>
   ```

5. Use `rws pull <task-id> --apply` only when the user wants remote code changes merged into the local workspace. Keep the default non-applying pull for artifacts and review.

## Show the task to the user

Whenever you start, check on, or finish a task, write its reference in your reply as `rws:<task-id>`, for example `rws:tsk_0123456789abcdef0123456789abcdef`. Clients that understand it (Vani) render a live card with status, duration, and Pull / Apply / Cancel actions; elsewhere it stays readable text.

- Put the reference **alone on its own line** for a card. Several references on consecutive lines (or a list of them) become one group with a summary.
- Mid-sentence, the reference becomes a small inline chip: `第 2 片 rws:tsk_… 失败了`.
- Use the exact ID rws printed. Never invent, shorten, or reformat it; a wrong ID shows as "Not found".
- Do not wrap it in a link or a code block. Inline backticks around the whole reference are fine. Inside fenced code blocks and shell commands it stays literal, so commands like `rws pull tsk_… --apply` are unaffected.
- A bare `tsk_…` (without `rws:`) is never turned into a card, so logs and NDJSON can quote IDs freely.

```text
Submitted to gpu-box with --detach; it keeps running if this chat closes.

rws:tsk_0123456789abcdef0123456789abcdef

Pull only downloads into .git/rws/pulls/; Apply is what changes your workspace.
```

## Safety

- Treat the SSH account as the command's full authority; rws is not a sandbox.
- Before publishing a snapshot, exclude credentials and large generated files with Git ignore rules or `.rwsignore`.
- A timeout or broken SSH connection does not mean the remote task stopped. Query its task ID instead of starting a duplicate.
- Do not invent a target, weaken SSH host-key checking, or copy private keys.
