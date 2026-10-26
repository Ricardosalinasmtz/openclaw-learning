# 🎓 OpenClaw Lesson: Authentication & OAuth
**Date:** 2026-10-21 | **Day:** 16 of learning streak
**Source:** gateway/authentication.md, concepts/oauth.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

How OpenClaw authenticates to model providers — API keys, OAuth (PKCE), and Claude CLI reuse — where the credentials actually live, and how to juggle multiple accounts and profiles on a single gateway.

## 📖 The Concept

OpenClaw separates two kinds of "login": **gateway connection** auth (who is allowed to talk to the gateway: tokens/passwords — covered in the configuration lessons) and **model provider** auth (which credentials the agent uses when it calls LLMs). This lesson is the second one. There are two big families: **API keys**, the most predictable option for an always-on gateway host (set as an env var on the gateway host, or in `~/.openclaw/.env` so a systemd/launchd daemon can read it), and **OAuth** ("subscription auth"), where you log in with a provider account through a browser flow — notably OpenAI Codex/ChatGPT OAuth and Claude CLI reuse for Anthropic.

The subtle part is where the tokens live. Each agent owns a local SQLite credential store (`openclaw-agent.sqlite`, with `auth_profile_store` rows) sitting on top of a shared read-through store at `~/.openclaw/state/openclaw.sqlite`. OpenClaw deliberately treats that store as a **token sink**: many OAuth providers rotate refresh tokens on every refresh and invalidate the old one, so logging in through both OpenClaw and the CLI can randomly log one of them out. With one canonical store per agent, refresh tokens are spent in one place, and once OpenClaw owns a local OAuth profile, its refresh token is canonical — the runtime re-reads externally managed CLI credentials instead of spending copied refresh tokens.

The OAuth exchange itself is a standard PKCE flow: generate a verifier/challenge, open the provider's authorize URL, capture the redirect on `localhost:1455/auth/callback` (or manually paste the redirect URL if you're on a headless host and the callback can't bind — whichever wins first), then exchange the code for `{access, refresh, expires, accountId}`. Refresh is automatic at runtime: if the stored token is expired, OpenClaw refreshes it and writes the new credentials back to the owning store, so you rarely manage tokens by hand.

Finally, multiple accounts. The cleanest pattern is **separate agents** — `openclaw agents add personal` / `openclaw agents add work` — so credentials, sessions, and workspaces never interact. Within a single agent, the profile store supports multiple `openai:*` / `anthropic:*` profiles for the same provider: global selection order comes from `auth.order`, and any chat can pin a specific credential per session with `/model ...@<profileId> -s`. Legacy `openai-codex:*` profile ids and the old JSON auth files (`auth-profiles.json`, `auth-state.json`, ...) are all migrated by `openclaw doctor --fix`.

## 🔑 Key Takeaways

- For always-on gateway hosts, API keys are the most predictable auth: set the env var on the gateway host (or `~/.openclaw/.env` for daemons), then verify with `openclaw models status`.
- OAuth login is a PKCE flow with a localhost callback; on headless hosts you can paste the redirect URL manually instead — manual paste races the browser callback and the first to complete wins.
- Credentials live in per-agent SQLite stores on top of a shared store, and OpenClaw treats the store as a single **token sink** so refresh-token rotation doesn't knock out other clients sharing the same account.
- Refresh is automatic; use `openclaw models status` (plus `--check` / `--probe`) to audit auth health, and `openclaw doctor --fix` to migrate legacy JSON auth files and legacy `openai-codex:*` profile ids.
- Multi-account: separate agents for hard isolation, or multiple profiles with global `auth.order` plus per-session pinning via `/model ...@<profileId> -s`.

## 💡 Practical Example

Two ChatGPT/Codex accounts (personal + work) on one gateway:

```bash
# Log in twice with distinct profile ids
openclaw models auth login --provider openai --profile-id openai:personal
openclaw models auth login --provider openai --profile-id openai:work

# See what's stored
openclaw models auth list --provider openai
```

Then in any chat, pin the credential for the current session only:

```
/model gpt-5@openai:work -s
```

`/model status` shows the full picture: candidates plus the next auth profile that would be used. If a stored profile is stuck, expired, or tied to the wrong account, `openclaw models auth login --provider openai --force` deletes the saved profiles for that provider in the agent and re-runs the flow (it does not revoke anything at the provider).

If the two accounts must never interact at all, skip profiles entirely — `openclaw agents add personal` and `openclaw agents add work` give each its own sessions, credentials, and workspace.

## 🔗 Related Concepts

- Model failover & resilience (`concepts/model-failover.md`) — what happens when a credential rate-limits or keys get rotated
- Secrets management & SecretRef (`gateway/secrets.md`) — credential storage for non-OAuth paths
- Gateway configuration (`gateway/configuration.md`) — gateway *connection* auth (token/password), a different layer from model provider auth

## 🤔 Quick Check

You're deploying an always-on gateway host for a client team. They share one provider subscription account, but you also need a second, separate profile for your own experiments so the two never mix. How would you structure the credentials so each chat session reliably uses the account it should?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Store two profiles with `openclaw models auth login --provider <id> --profile-id <provider>:client` and `--profile-id <provider>:personal`, set the default with `auth.order`, and pin per session with `/model <model>@<provider>:client -s` in client chats (or `@<provider>:personal -s` in yours). If the accounts must be fully isolated — separate billing, sessions, and workspace — put them in separate agents instead; profile pinning only steers which credential a shared agent uses.

</details>

---
*Lesson 25 of Phase 2 (Gateway, Configuration & Safety) | Streak: 16 days*
