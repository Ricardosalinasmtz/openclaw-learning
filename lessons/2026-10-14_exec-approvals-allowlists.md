# 🎓 OpenClaw Lesson: Exec Approvals & Allowlists
**Date:** 2026-10-14 | **Day:** 11 of learning streak
**Source:** tools/exec-approvals.md | **Difficulty:** advanced

## 🎯 What You'll Learn

Every time a sandboxed agent wants to run a command on a real machine — the gateway host or a paired node — OpenClaw has to decide whether that command may execute. Today we learn how the exec approvals system makes that decision: which policy knobs gate host exec, how per-agent allowlists work, and what "YOLO mode" actually takes to unlock.

## 📖 The Concept

Exec approvals are the **companion app / node host guardrail** for letting an agent run commands on a real host. A command runs only when three things agree: the configured policy, the allowlist (if one applies), and — when required — an explicit user approval. The system stacks on top of tool policy and elevated gating, and elevated `full` skips approvals entirely. Approvals are enforced locally on the execution host: the gateway host checks against the `openclaw` process on the gateway machine, and a node host checks against the node runner (macOS companion app or headless node host).

The core design rule is that the **effective policy is always the stricter** of the two layers: your OpenClaw config (`tools.exec.*`) and the host-local approvals document. Approvals can only tighten security or ask behavior, never loosen it — a stricter host-local `ask: "always"` keeps prompting even if your config requests `ask: "off"`. When a prompt is needed but no UI is reachable (or times out), `askFallback` decides what happens, and its default is `deny`. That fail-closed default is exactly why an unattended cron run that trips an approval is denied immediately unless a standing grant already covers it.

The normalized policy surface is `tools.exec.mode`, with five values: `deny`, `allowlist`, `ask`, `auto`, and `full`. The legacy `security`/`ask` pair is still honored, and `openclaw doctor --fix` migrates the well-supported combinations. Allowlists are **per agent** and consist of glob patterns — either a resolved binary path or a bare command name (bare names only match commands invoked through `PATH`, so `rg` matches `/opt/homebrew/bin/rg` but not `./rg`). An entry can be narrowed with `argPattern`, an ECMAScript regex evaluated against the parsed arguments, so `python3` can be trusted only for `python3 safe.py`. Choosing "Always allow here" in an approval card mints an entry bound to the exact argv **and** the working directory, so the same command run elsewhere is an allowlist miss.

One last frame: approvals reduce accidental-execution risk, but they are **not** a per-user auth boundary or a read-only filesystem policy — once approved, a command can mutate files according to the host's permissions. For automation (cron) runs, approvals are delivered only to connected exec approval clients (Control UI, companion apps, API clients); chat channels never receive them. "Always allow" on an automation approval mints a scoped **standing grant** bound to the exact agent, job configuration, and operation — and it fails closed back to a normal prompt whenever the job, command, working directory, or environment changes by even one byte.

## 🔑 Key Takeaways

- A host command runs only when policy + allowlist + (optional) approval all agree, and the effective policy is always the **stricter** of your config and the host-local approvals document.
- `tools.exec.mode` (`deny` / `allowlist` / `ask` / `auto` / `full`) is the normalized knob; `auto` adds a native auto-reviewer before falling back to a human approval.
- Allowlists are per-agent glob patterns (binary path or bare command name), and `argPattern` regexes restrict *which arguments* a binary is trusted with.
- "Always allow here" binds the approval to the exact argv and working directory; automation approvals mint scoped standing grants that fail closed on any change.
- `askFallback` defaults to `deny` — a prompt with no reachable UI (headless, timeout) is denied, which is why unattended jobs need standing grants to run cleanly.

## 💡 Practical Example

Inspect and tune the effective policy on a gateway host:

```bash
# See the requested policy, host sources, and the effective result
openclaw approvals get

# Switch the local machine to a cautious setup: allowlist + ask on misses + deny fallback
openclaw exec-policy preset cautious

# Allow a specific binary, then restrict python3 to one script
openclaw approvals set --stdin <<'EOF'
{
  "version": 1,
  "defaults": { "security": "allowlist", "ask": "on-miss", "askFallback": "deny" },
  "agents": {
    "main": {
      "allowlist": [
        { "pattern": "rg" },
        { "pattern": "python3", "argPattern": "^safe\\.py$" }
      ]
    }
  }
}
EOF
```

Now `rg ...` runs without prompting, `python3 safe.py` runs, and `python3 other.py` prompts (or is denied headless). To go fully unattended on a trusted box, the YOLO combination is `tools.exec.mode: "full"` in config **plus** `askFallback: "full"` in the host approvals document — opening one layer alone is not enough.

## 🔗 Related Concepts

- [Exec tool](tools/exec.md) — shell execution and session `/exec` overrides
- [Sandboxing](gateway/sandboxing.md) — where exec runs by default
- [Sandbox vs tool policy vs elevated](gateway/sandbox-vs-tool-policy-vs-elevated.md) — choosing the right control
- [Elevated mode](tools/elevated.md) — the break-glass path that also skips approvals
- [Exec approvals — advanced](tools/exec-approvals-advanced.md) — safe bins, interpreter binding, chat approval forwarding

## 🤔 Quick Check

A scheduled cron job on your gateway runs `python3 /home/ricardo/scripts/etl.py` every morning. It prompted for approval on its first run and you hit "Always allow", but the next morning the job is denied and no approval card appears anywhere. Using what you learned today, what is the most likely cause, and where would you look to fix it?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** The automation's standing grant was invalidated — grants fail closed back to a normal prompt whenever the job, command, working directory, or environment changes by even one byte (including a job edit), and with no connected exec approval client the re-prompt is denied immediately. Check `openclaw approvals grants list` to see the grant's state (revoked, expired, or invalidated) and compare the current cron job configuration and command against what was originally approved.

</details>

---
*Lesson 20 of Phase 2 (Gateway, Configuration & Safety) | Streak: 11 days*
