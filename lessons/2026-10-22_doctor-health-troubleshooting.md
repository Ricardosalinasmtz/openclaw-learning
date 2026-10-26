# 🎓 OpenClaw Lesson: Doctor, Health & Troubleshooting
**Date:** 2026-10-22 | **Day:** 17 of learning streak
**Source:** gateway/doctor.md, gateway/health.md, gateway/troubleshooting.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today you'll learn the three tools OpenClaw gives you for keeping a gateway healthy: `openclaw doctor` (the repair and migration tool), the health check commands (`status`, `health`, and the HTTP probe endpoints), and the deep troubleshooting runbook. By the end you'll know which command answers which question — "is it configured correctly?", "is it alive right now?", and "which failure domain is actually broken?" — and you'll be able to run a safe, read-only preflight check in CI.

## 📖 The Concept

**Doctor is the repair and migration tool.** `openclaw doctor` inspects your config and on-disk state, normalizes legacy config shapes into the current schema, migrates old disk layouts (legacy session stores, agent directories, WhatsApp auth), cleans stale locks, checks model auth health (OAuth expiry, token refresh), audits gateway services and supervisor configs, and gives you actionable repair steps. The flags determine how much it touches: bare `doctor` is interactive; `--fix` (alias `--repair`) applies recommended repairs without prompting; `--fix --force` also applies aggressive repairs; `--non-interactive` only applies safe migrations and skips restart/service actions; and `--deep` scans system services for extra gateway installs. There's also a read-only sibling, `openclaw doctor --lint`, built for CI and preflight automation: same rule registry, no prompts and no writes, structured JSON findings with per-check ids, and exit codes (0 = clean, 1 = findings at or above the severity threshold, 2 = runtime failure). `--severity-min` (default `warning`), `--all`, `--only <id>`, and `--skip <id>` let you scope a lint run tightly. Notably, `--fix` and `--lint` deliberately do not share selection: a finding that appears in `--lint --all` is diagnostic and does not mean `--fix` will mutate that area.

**Health commands answer "is it alive, right now?"** The quick ladder is `openclaw status` (local summary: reachability, channel auth age, sessions), `openclaw status --deep` (asks the running gateway for a live probe), and `openclaw health` (fetches the gateway's health snapshot over WebSocket, with `--verbose` forcing a live probe and `--json` for machine-readable output). The gateway also exposes unauthenticated HTTP probe pairs: `/health` (server is live — use this for uptime monitors), `/startup` (startup complete and not draining — use for orchestrator traffic admission), and `/ready` (startup complete *and* configured channel accounts pass deep readiness — use when you want hard channel failures to surface). A key operational detail: never point an uptime monitor at `/v1/chat/completions` — every request creates a full agent session with context assembly and LLM calls, so a 15-minute ping creates ~96 bloated sessions a day.

**The troubleshooting runbook is symptom-based, and the first lesson is that channels have separate failure domains.** A channel can hold a healthy transport connection (sending replies fine) while its durable ingress queue is unavailable, admitting zero inbound messages — in that state the account is unhealthy regardless of transport, and `openclaw channels status` tells you directly. The runbook's standing advice is a command ladder, in order: `openclaw status` → `openclaw gateway status` → `openclaw logs --follow` → `openclaw doctor` → `openclaw channels status --probe`. After updates specifically, the sequence is `openclaw status --all`, `openclaw update status --json`, `openclaw gateway status --deep`, `openclaw doctor --fix`, then `openclaw gateway restart`. The runbook also covers nastier scenarios: split-brain installs where an older binary refuses to mutate a config stamped with a newer `meta.lastTouchedVersion`, protocol mismatches after a rollback (a newer client process keeps reconnecting; find its PID via `gateway status --deep`), invalid-config rejections (look for timestamped `.clobbered.*` and `.rejected.*` files beside `openclaw.json` — doctor `--fix` restores last-known-good while preserving the broken edit), and memory-pressure crashes (inspect the payload-free stability bundle with `openclaw gateway stability --bundle latest`).

## 🔑 Key Takeaways

- `openclaw doctor` is both doctor and fixer: it detects legacy config keys and old disk layouts, then `--fix` applies repairs; `--lint` is the read-only CI/preflight version with JSON findings and meaningful exit codes (0/1/2).
- Startup does its own safe legacy-key migration, but if anything remains it refuses to start and points you at `openclaw doctor --fix`; previous configs are kept in a five-slot `.bak` backup ring, and broken writes are preserved as `.clobbered.*` / `.rejected.*` files.
- Uptime monitors should hit `GET /health` (instant, no session created), never `/v1/chat/completions` (each ping creates a full agent session).
- `/health` vs `/startup` vs `/ready` are three different contracts: liveness, startup complete, and startup complete plus channels passing deep readiness — pick the one your orchestrator or operator monitoring actually needs.
- Channel connectivity and inbound admission are separate failure domains: a healthy transport can still admit zero inbound messages when the ingress queue is unavailable; recovery is the ordinary restart path, logged as `health-monitor: restarting (reason: ingress-unavailable)`.
- The troubleshooting command ladder is fixed: `status` → `gateway status` → `logs --follow` → `doctor` → `channels status --probe`; for "no replies", check pairing, mention gating, and allowlists before reconnecting anything.

## 💡 Practical Example

A realistic weekly preflight you can wire into CI or a scheduled job — fully read-only, no prompts, no writes:

```bash
# 1. Structured lint: exit 0 = clean, 1 = findings at/above warning, 2 = runtime failure
openclaw doctor --lint --severity-min warning --json > /tmp/doctor-lint.json

# 2. Live gateway probe (asks the running gateway, per-account channel probes)
openclaw status --deep

# 3. Machine-readable health snapshot from the gateway
openclaw health --json

# 4. If anything looks off, go deep before touching anything:
openclaw gateway status --deep
openclaw logs --follow
```

If the lint finds stale legacy config keys after an update, the repair is explicit and auditable: `openclaw doctor --fix` (or `--fix --force` for aggressive repairs), then `openclaw gateway restart`. For bug reports, don't paste raw logs — run `openclaw gateway diagnostics export` and attach the generated zip (Markdown summary, newest stability bundle, sanitized log metadata and health snapshots; chat text, credentials, and secret values are omitted or redacted).

## 🔗 Related Concepts

- [Gateway runbook](/gateway)
- [Heartbeat automation](/gateway/heartbeat)
- [Configuration basics](/gateway/configuration)
- [Authentication & OAuth](/gateway/authentication)
- [Background tasks & run history](/automation/tasks)

## 🤔 Quick Check

You update OpenClaw overnight and, the next morning, the gateway service reports running and `openclaw status` shows your Telegram channel as connected, but no inbound messages are being processed and replies you send manually go out fine. Walk through the order you would run the diagnostic commands, and say what specific output would let you distinguish a broken ingress queue from a transport problem from a sender policy block (pairing, allowlist, or mention gating).

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Run the standard ladder in order: `openclaw status`, then `openclaw channels status --probe` for live per-account transport status, then `openclaw logs --follow` while checking policy state with `openclaw pairing list --channel telegram` and `openclaw config get channels`. A broken ingress queue shows the channel with a healthy transport but a start failure and the message "Channel cannot admit inbound events; its durable ingress queue is unavailable. Outbound may still work." — and it recovers via the normal restart path, logged as `health-monitor: restarting (reason: ingress-unavailable)`. A transport problem would show as failing/unhealthy probes in `channels status --probe` rather than connected, with reconnect attempts or relink status codes (409–515) in the logs. A policy block keeps everything green and shows up only in the logs as `drop ... (mention required)`, `pairing request`, or `blocked`/`allowlist` lines — which is why replies going out fine while nothing comes in should push you toward policy and ingress checks, not reconnecting.

</details>

---
*Lesson 26 of Phase 2 (Gateway, Configuration & Safety) | Streak: 17 days*
