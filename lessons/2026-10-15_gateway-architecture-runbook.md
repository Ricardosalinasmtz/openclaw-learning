# 🎓 OpenClaw Lesson: Gateway Architecture & Runbook

**Date:** 2026-10-15 | **Day:** 12 of learning streak
**Source:** concepts/architecture.md, gateway/index.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today we look under the hood of the process that connects OpenClaw to the outside world: the Gateway. We'll learn how a single long-lived daemon owns every channel and every control-plane client, how its WebSocket wire protocol works, and how to operate it day to day — startup, health checks, supervision, hot reload, and the failure signatures worth memorizing.

## 📖 The Concept

The Gateway is one long-running daemon per host. It owns all messaging surfaces — WhatsApp, Telegram, Slack, Discord, Signal, iMessage, WebChat — and all control-plane clients (macOS app, CLI, web UI, automations) connect to it over WebSocket, on `127.0.0.1:18789` by default. There's one Gateway per machine; it's the only place a WhatsApp session opens, so two instances can't share one. The same port is multiplexed: OpenAI-compatible HTTP endpoints (`/v1/models`, `/v1/chat/completions`, `/v1/embeddings`, `/v1/responses`), plugin routes, and the Control UI all ride the same listener.

The wire protocol is JSON over WebSocket text frames. The client's very first frame must be `connect`; after the gateway answers with `hello-ok` (a snapshot of presence, health, and policy limits), everything becomes requests — `{type:"req", id, method, params}` answered by `{type:"res", ...}` — plus server-pushed events like `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, and `cron`. Agent runs are two-stage: an immediate "accepted" ack with a runId, streamed `agent` events in between, and a final completion response at the end. Side-effecting methods like `send` and `agent` require idempotency keys so retries stay safe.

Trust is layered. Gateway auth (shared-secret token/password, or identity-bearing modes like Tailscale Serve and trusted-proxy) applies to *every* connection, local or remote. On top of that, each client carries a device identity: new device IDs need pairing approval, after which the gateway issues a device token; local loopback connects can be auto-approved, but tailnet, LAN, and non-local connects always require explicit approval. Events are never replayed, so a client that notices a sequence gap must refresh state (health, system-presence) instead of assuming it caught up.

The runbook side is about operations: start with `openclaw gateway` (or install it as a launchd/systemd/scheduled-task service for auto-restart), verify with `openclaw gateway status` and `openclaw channels status --probe`, and lean on config hot reload — `hybrid` mode, the default, hot-applies safe changes and restarts when needed. Invalid config exits with code 78, and the Linux systemd unit sets `RestartPreventExitStatus=78` so a broken config doesn't spin in a restart loop. Remote access prefers Tailscale/VPN, with an SSH tunnel as fallback — but the tunnel does not bypass auth: clients must still send their token even over the tunnel.

## 🔑 Key Takeaways

- One Gateway per host, one multiplexed port (default 18789) carrying WebSocket RPC, OpenAI-compatible HTTP, and the Control UI together.
- The first WS frame must be `connect`; agent runs are two-stage (accepted ack → streamed events → final response), and side-effecting methods require idempotency keys.
- Auth and pairing are separate layers: `gateway.auth.*` covers every connection, while device pairing decides which clients may operate — loopback can auto-approve, everything else needs explicit approval.
- Operate it with `openclaw gateway status`, `openclaw channels status --probe`, and `openclaw doctor`; supervise via launchd/systemd/schtasks; hot reload defaults to `hybrid`.
- Exit code 78 means invalid config — service definitions deliberately stop restarting until you fix it.
- Tailscale/VPN is the preferred remote-access path; an SSH tunnel never bypasses auth.

## 💡 Practical Example

Day-2 operations ladder for a gateway on a fresh machine:

```bash
# 1. Start it in the foreground first (logs to stdout)
openclaw gateway --port 18789 --verbose

# 2. Prove it's healthy — not just reachable
openclaw gateway status                  # Runtime: running, Connectivity probe: ok
openclaw gateway status --require-rpc    # read-scope RPC proof
openclaw channels status --probe         # live per-account channel probes

# 3. Hand it to the OS so it survives reboots
openclaw gateway install                 # launchd (macOS) / systemd (Linux) / schtasks (Windows)

# 4. After changing gateway.port, refresh the service metadata
openclaw doctor --fix

# 5. Remote access (from your laptop, to the gateway host)
ssh -N -L 18789:127.0.0.1:18789 user@gateway-host
# ...or simply put the host on your Tailscale and connect directly
```

Failure signatures to recognize: `another gateway instance is already listening` / `EADDRINUSE` is a port conflict; `refusing to bind gateway ... without auth` is a non-loopback bind without a valid auth path; `Gateway start blocked: set gateway.mode=local` means the config is in (or damaged into) remote mode.

## 🔗 Related Concepts

- [Gateway Protocol](/gateway/protocol) — the full WebSocket contract and TypeBox-generated schemas
- [Agent Loop](/concepts/agent-loop) — what happens inside the two-stage accepted → final ack
- [Security](/gateway/security) — trust model, pairing, and hardening
- [Queue](/concepts/queue) — command queueing and concurrency behind the `agent` method
- [Gateway Troubleshooting](/gateway/troubleshooting) — symptom-first diagnosis ladders

## 🤔 Quick Check

You've moved your gateway from a laptop to a headless Ubuntu server and want to run the CLI and Control UI from your own machine. A colleague suggests the fastest fix: open port 18789 on the LAN and point your clients at the server's IP. Based on today's lesson, what do you do instead — and if you did go with the SSH-tunnel fallback, what would still have to happen on the client side before it could operate?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** I'd prefer Tailscale (or a VPN) over an open LAN bind — the runbook names it the preferred path, and a non-loopback bind without a proper auth path is exactly the `refusing to bind ... without auth` failure. Even with the SSH tunnel, the client must still authenticate: the tunnel does not bypass gateway auth, so it still sends the shared-secret token/password (or satisfies an identity-bearing auth path like trusted-proxy). On top of that, my laptop's device identity is new to this gateway, so it needs pairing approval before it can operate.

</details>

---
*Lesson 21 of Phase 2 (Gateway, Configuration & Safety) | Streak: 12 days*
