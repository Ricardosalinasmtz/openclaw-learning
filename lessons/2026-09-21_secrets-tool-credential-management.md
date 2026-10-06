# 🎓 OpenClaw Lesson: Secrets Tool — Credential Management
**Date:** 2026-09-21 | **Day:** 7 of learning streak
**Source:** [tools/secrets.md](/tools/secrets.md) | **Difficulty:** intermediate

## 🎯 What You'll Learn

How the `secrets` tool enables the agent to obtain credentials without ever seeing them. The human enters the value through a masked prompt, and the Gateway stores it in a protected shared store.

## 📖 The Concept

The `secrets` tool is designed around one core principle: **the agent should never see your credentials**. When a task requires an API key, token, or password, the agent requests it through this tool. You then type the value into a trusted masked prompt — it appears as dots or stars, not plaintext. The Gateway writes it straight into the shared secret store. The value never appears in the chat, the session transcript, the tool result, or the model's context.

This solves a real security problem. Without this flow, agents would need credentials to be pasted into chat messages, stored in environment variables visible to the agent, or hard-coded in scripts. With `secrets`, the agent only receives **metadata** (the entry name, allowed hosts, kind) and a **SecretRef** — a reference it uses to resolve the credential where needed, never the value itself.

The tool has three actions:
- **`request`** — Ask you for a new credential. You specify the name (e.g., `STRIPE_API_KEY`), a reason, and the allowed hosts (which destinations the credential can be sent to). The prompt blocks until you answer, skip, or it times out (default 15 minutes, clamped to 30–3600 seconds).
- **`list`** — Show metadata for all stored entries: name, kind, allowed hosts, last update. Secret values are structurally absent.
- **`delete`** — Soft-delete an entry (purged after 30 days).

There is intentionally **no action that lets the agent write a value**. If a credential must enter the store, it arrives through you (the human), the `/settings/secrets` page, or the `openclaw secrets store` CLI. This is a deliberate design choice.

## 🔑 Key Takeaways

- **Zero exposure:** The credential value is never visible to the agent, in chat logs, tool results, or model context — it flows only through your masked prompt.
- **Host-bound egress:** Each entry has an **allowed hosts** list that governs Gateway egress substitution. Only matching destinations receive the credential. An empty list prevents egress entirely while still allowing config SecretRef resolution.
- **Chat channels never accept the value:** On Telegram, Discord, etc., the request is delivered as a link to the Control UI prompt. Typing a credential into a chat message is exactly what this flow avoids — plain-text replies are not captured as answers.
- **Ephemeral run binding:** A credential request is bound to the requesting agent run. If that authority closes before you answer, the pending prompt is cancelled and the write is refused.
- **Egress requires the proxy:** Protected entries reach Gateway-host subprocess traffic only through automatically injected opaque environment sentinels, and only when `secrets.egressProxy.enabled` is true. Native harness shell, sandbox, and node execution do not receive these values.

## 💡 Practical Example

**Scenario:** You want your agent to interact with the Stripe API, but you don't want it to ever see your API key.

1. **Agent requests the credential:**

```json
{
  "action": "request",
  "name": "STRIPE_API_KEY",
  "kind": "secret",
  "allowedHosts": ["api.stripe.com"],
  "reason": "Need Stripe API access for payment processing",
  "timeoutSeconds": 300
}
```

2. **You see a masked prompt** in the Control UI showing the agent, session, name, reason, and editable allowed hosts. You type your key (shown as dots) and submit.

3. **The agent receives a SecretRef** — not the key:

```json
{
  "status": "stored",
  "ref": {
    "source": "store",
    "provider": "default",
    "id": "STRIPE_API_KEY"
  },
  "currentPolicy": {
    "allowedHosts": ["api.stripe.com"]
  }
}
```

4. **The agent uses the ref** wherever a SecretRef is accepted — in provider API key fields, channel tokens, etc. The Gateway resolves it automatically.

5. **On Gateway-hosted exec**, the agent never references the env var. Instead, OpenClaw injects an opaque sentinel into the process environment. The agent's commands use it transparently. No `print()`, no URL parameters, no log lines.

## 🔗 Related Concepts

- [Automations & Scheduling](/automation/index.md) — For recurring tasks that need credentials (e.g., payment webhooks)
- [Gateway Secrets](/gateway/secrets) — The shared store backing all SecretRefs
- [Tool Policy](/tools/index.md) — `secrets` is enabled by default; can be denied like any other tool via `tools.deny: ["secrets"]`

## 🤔 Quick Check

If an agent requests a `GITHUB_TOKEN` with `allowedHosts: ["api.github.com"]`, and then the Gateway proxy is disabled, can the agent still use that stored credential in an exec command? Why or why not?

---

*Lesson 8 of Phase 2 (Gateway, Configuration & Safety) | Streak: 7 days*
