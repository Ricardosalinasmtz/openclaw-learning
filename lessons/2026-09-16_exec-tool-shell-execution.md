# 🎓 OpenClaw Lesson: Exec Tool & Shell Execution
**Date:** 2026-09-16 | **Day:** 4 of learning streak
**Source:** tools/exec.md | **Difficulty:** Intermediate

## 🎯 What You'll Learn

The `exec` tool is OpenClaw's primary interface for running shell commands — from simple file listings to complex multi-step builds. You'll learn how exec works, its parameters, execution modes (foreground vs background), and the approval/security model that governs when commands run freely vs. when they need human or auto-reviewer approval.

## 📖 The Concept

### What Is `exec`?

`exec` is a **mutating shell surface** — it can create, edit, or delete files anywhere the selected host or sandbox permits. Crucially, disabling other filesystem tools (like `write`, `edit`, or `apply_patch`) does **not** make `exec` read-only. It is the most powerful tool in the agent's arsenal and must be understood carefully.

Commands run on one of four hosts:
- **`auto`** (default): Resolves to `sandbox` when active, `gateway` otherwise
- **`sandbox`**: Isolated container execution
- **`gateway`**: Runs directly on the host machine
- **`node`**: Runs on a paired remote node

### Foreground vs Background

`exec` supports two execution modes:

**Foreground** — the agent waits for the command to finish before proceeding:
```json
{ "tool": "exec", "command": "ls -la" }
```

**Background** — the command runs asynchronously, and the agent uses `process` to check on it:
```json
{ "tool": "exec", "command": "npm run build", "yieldMs": 1000 }
```
After backgrounding, use `process` actions like `poll`, `log`, `write`, `send-keys`, or `kill` to manage the running command.

Key parameters:
- **`yieldMs`** (default 10000ms): How long to wait in foreground before auto-backgrounding
- **`background`**: Immediately background instead of waiting for `yieldMs`
- **`timeoutSeconds`**: Maximum runtime (default 1800s); `0` disables the limit

### Security Modes

The `tools.exec.mode` setting controls how strictly exec commands are gated:

| Mode | Security | Approval | Behavior |
|---|---|---|---|
| `deny` | deny | off | Exec is blocked entirely |
| `allowlist` | allowlist | off | Only allowlisted commands run |
| `ask` | allowlist | on-miss | Allowlist runs directly; everything else asks a human |
| `auto` | allowlist | on-miss | Allowlist runs directly; rest goes through auto-reviewer, then human |
| `full` | full | off | No approval gate — everything runs |

The default for gateway and node hosts is typically `full` (no approval needed). Sandboxed agents often use `auto` or `ask` for extra safety.

### Session Overrides

You can override the host per-session using `/exec`:
```
/exec host=node node=worker-1
/exec security=allowlist ask=always inspect the build output
```
These only last for the current message (except placement overrides for authorized senders).

## 🔑 Key Takeaways

- **`exec` is mutating**: It can modify files anywhere host/sandbox permissions allow — it's not just for reading output
- **`yieldMs` and `background` control async execution**: Use them to avoid blocking; manage with `process`
- **`timeoutSeconds: 0` disables the timeout limit** — useful for long-running services (e.g., dev servers)
- **`pty=true` enables TTY mode** for interactive CLIs and terminal UIs that require a terminal
- **Security mode matters**: `full` skips all approvals; `auto` uses model auto-review before asking humans; `allowlist` restricts to a permit list
- **`host=auto` is context-aware**: It stays in the sandbox when one is active, falls back to gateway otherwise

## 💡 Practical Example

Here's a real-world workflow: starting a development server in the background and monitoring it.

```json
// Step 1: Start the server in the background
{
  "tool": "exec",
  "command": "npm run dev",
  "workdir": "/home/ricardo/projects/my-app",
  "background": true,
  "timeoutSeconds": 0
}

// Step 2: Poll for status
{
  "tool": "process",
  "action": "poll",
  "sessionId": "<returned-session-id>",
  "timeout": 5000
}

// Step 3: Read output logs
{
  "tool": "process",
  "action": "log",
  "sessionId": "<returned-session-id>"
}

// Step 4: Stop when done
{
  "tool": "process",
  "action": "kill",
  "sessionId": "<returned-session-id>"
}
```

For inline scripts, always set `workdir` to the project directory so the command uses the right environment. For Python in `uv` projects, use `uv run python3 script.py` — don't rely on bare `python3` unless the project env is on PATH.

## 🔗 Related Concepts

- **[Process Tool](/tools/process.md)** — Managing background exec sessions
- **[Exec Approvals](/tools/exec-approvals)** — Allowlist, safe bins, and approval flows
- **[Sandboxing](/gateway/sandboxing)** — Running commands in isolated containers
- **[Automations](/automation/index.md)** — Scheduling work instead of exec sleep loops
- **[Tools Overview](/tools/index.md)** — The full tool catalog

## 🤔 Quick Check

A sandboxed agent with `mode=auto` receives the command `python -c "import os; print(os.getcwd())"`. The command is not in the allowlist. What happens, in order?

<details>
<summary>Click to reveal the answer</summary>

The auto-reviewer evaluates the command. Since `python -c` is an inline-eval form, if `strictInlineEval` is enabled, it requires explicit approval. Otherwise, the auto-reviewer may allow it as low-risk (since it's just printing the current working directory), and the command runs without human involvement. If the reviewer is uncertain, it falls back to asking a human.
</details>

---
*Lesson 5 of Phase 1 (Core Concepts) | Streak: 4 days*
