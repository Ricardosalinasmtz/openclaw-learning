# 🎓 OpenClaw Lesson: Sandboxing vs Tool Policy vs Elevated
**Date:** 2026-10-20 | **Day:** 15 of learning streak
**Source:** gateway/sandboxing.md, gateway/sandbox-vs-tool-policy-vs-elevated.md | **Difficulty:** advanced

## 🎯 What You'll Learn

OpenClaw gates tool execution with three independent controls that answer three different questions: sandboxing decides *where* a tool runs, tool policy decides *whether* a tool exists at all, and elevated is an exec-only escape hatch back out of the sandbox. Once you can tell the three apart, "why is this tool blocked?" stops being a mystery and becomes a two-minute lookup.

## 📖 The Concept

Sandboxing moves tool execution — `exec`, the file tools, optionally the browser — into a container or remote machine. It's off by default and shaped by three independent settings under `agents.defaults.sandbox`: `mode` (`off`, `non-main`, `all`) decides *which sessions* are sandboxed, `scope` (`agent`, `session`, `shared`) decides *how many containers* exist, and `backend` (`docker`, `podman`, `ssh`, `openshell`) decides *where they run*. The Gateway process itself always stays on the host — only the tools move. Docker's default posture is deliberately harsh: `network: "none"`, a read-only root filesystem, all capabilities dropped, and the agent workspace read-only unless you explicitly mount it.

Tool policy is a separate layer that runs before sandboxing ever gets involved. Profiles plus `tools.allow`/`tools.deny` (global, per-agent, and sandbox-specific) decide which tools the model can even see. Two rules dominate: `deny` always wins, and if `allow` is non-empty, everything not listed is blocked. Critically, tool policy filters by tool name — it never inspects what a shell command actually does. If `exec` is allowed, denying `write`, `edit`, and `apply_patch` does not make your shell read-only.

Elevated is an escape hatch, not a permission. `tools.elevated` (or the `/elevated` directive for authorized senders) only changes *where `exec` runs* — outside the ordinary sandbox, on the gateway or node host. It grants no new tools, doesn't override deny rules, and cannot escape a sandbox that a creator operator role has made required. If sandboxing is off entirely, elevated is a no-op, because exec already runs on the host.

## 🔑 Key Takeaways

- Three separate gates: sandbox = where tools run, tool policy = which tools exist, elevated = exec-only escape from the sandbox.
- `deny` always wins; a non-empty `allow` list blocks everything else; and tool policy never looks inside a command.
- The classic `non-main` surprise: group and channel sessions are *not* main sessions, so they get sandboxed even when your main session doesn't.
- `openclaw sandbox explain` is your debug Swiss Army knife — it prints the effective mode, workspace access, tool policy, and the fix-it config keys.
- `docker.binds` pierces the sandbox filesystem with whatever mode you set; prefer `:ro`, and OpenClaw blocks dangerous sources (system paths, Docker socket, SSH credential roots) by default.

## 💡 Practical Example

A Telegram group session suddenly can't use `web_search`. The diagnosis and fix:

```bash
# 1. Inspect what OpenClaw is actually doing for that session
openclaw sandbox explain --session agent:work:group:telegram:12345
# → mode=non-main, sandboxed=true
# → effective sandbox tools allow: ["group:runtime", "group:fs"]

# 2. Option A: allow web tools inside the sandbox
openclaw config set tools.sandbox.tools.allow \
  '["group:runtime", "group:fs", "group:web"]'

# 3. Option B: audit log confirms which rule fired
openclaw logs | grep "agents/tool-policy"
```

Now say the group user needs one genuinely host-level command — installing a system package. That's elevated's job: set `tools.elevated.enabled: true`, add the user to `tools.elevated.allowFrom.<provider>`, then the user runs `/elevated on` for that session and their next `exec` lands on the host. Turn `/elevated off` afterward to go back to the sandbox.

## 🔗 Related Concepts

- Exec approvals & allowlists (2026-10-14) — the approval layer that can still gate elevated exec
- Configuration basics (2026-10-16) — where all these config keys live
- Gateway architecture & runbook (2026-10-15) — why the Gateway itself never moves into the sandbox

## 🤔 Quick Check

Your agent runs with sandbox mode `non-main` on the Docker backend, which defaults to `network: "none"` and a read-only root filesystem. A user in a group chat asks the agent to install a system package on the host so a later script can use it. The `exec` tool is allowed and the command runs — but the install fails inside the container. Which of the three controls — sandbox, tool policy, or elevated — should you change to give this one user what they asked for, and why can't the other two fix it without giving up something you wanted?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Enable elevated: set `tools.elevated.enabled` to true, add the user to `tools.elevated.allowFrom.<provider>`, and have them run `/elevated on` so their next `exec` runs on the gateway host where the package manager works. Tool policy can't fix it because `exec` is already allowed — policy only decides which tools exist, never where they run, so allowing more tools still leaves the command inside the network-less, read-only container. Switching the sandbox mode off would also get the command onto the host, but it strips the isolation from *every* non-main session instead of granting host access to one authorized sender for one session, which is exactly the blast-radius control the sandbox was there to give you.

</details>

---
*Lesson 24 of Phase 2 (Gateway, Configuration & Safety) | Streak: 15 days*
