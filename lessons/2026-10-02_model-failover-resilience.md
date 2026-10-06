# 🎓 OpenClaw Lesson: Model Failover & Resilience
**Date:** 2026-10-02 | **Day:** 3 of learning streak
**Source:** concepts/model-failover.md | **Difficulty:** advanced

## 🎯 What You'll Learn

How OpenClaw keeps conversations alive when models and providers fail — including auth profile rotation, model fallback chains, cooldown management, and billing recovery. Understanding this is critical for building resilient agent setups that don't drop mid-task.

## 📖 The Concept

When a model call fails, OpenClaw doesn't just give up. It follows a **two-stage recovery process**:

1. **Auth profile rotation** — If the provider is configured with multiple auth profiles (e.g., a primary API key and a backup), OpenClaw tries the next profile on the same model/provider.
2. **Model fallback** — If all profiles for that provider fail, OpenClaw moves to the next model in the configured `fallbacks` list.

Before rotating anything, OpenClaw first attempts **bounded same-model recovery** for transient issues like rate limits. It preserves partial output, keeps the transcript intact, and uses jittered exponential backoff (capped at 30 seconds) with up to 10 total retry attempts for rate limits. Only after this budget is exhausted does it move to profile rotation or model fallback.

**Not everything falls back.** Explicit user model selections (`/model ollama/qwen3.5:27b`, the model picker, `sessions.patch`) are **strict** — if that provider/model fails before producing a reply, OpenClaw reports the failure instead of silently answering from a fallback. This prevents confusing the user with an unexpected model change. Conversely, configured defaults and cron job primaries use fallbacks by design.

## 🔑 Key Takeaways

- **Two-stage recovery:** Auth rotation first, then model fallback. Both are turn-local — the next turn starts fresh from the selected primary.
- **Strict vs. flexible:** User-selected models are strict (no silent fallback). Configured defaults and cron payloads use fallback chains.
- **Cooldown scaling:** Profile cooldowns escalate — 30s after the 1st failure, 1 minute for the 2nd, 5 minutes for the 3rd+. Billing failures trigger a 10-minute disable window.
- **Auth profile stickiness:** OpenClaw pins the chosen auth profile per session to keep provider caches warm. Auto-pins rotate on cooldown/expiry; user-pins survive `/new` and `/reset`.
- **Skip cache for auth failures:** Setting `OPENCLAW_FALLBACK_SKIP_TTL_MS=60000` suppresses repeat auth failures for 60 seconds per session, saving API calls to known-bad profiles.
- **Visible notices:** Outside groups, users see `↪️ Model Fallback: <model> (selected <primary>; <reason>)` when a turn moves to fallback, and a clear notice when the primary recovers.
- **What triggers fallback:** Auth failures, rate limits, overloaded/busy, timeouts, billing, and `model_not_found` (HTTP 404). What doesn't: context overflow, explicit refusals, and final unknown errors with no remaining candidates.

## 💡 Practical Example

Configure a resilient model chain in your `openclaw.json`:

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "openai/gpt-4o",
        fallbacks: [
          "anthropic/claude-sonnet-4-20250514",
          "openai/gpt-4o-mini"
        ]
      }
    }
  },
  auth: {
    profiles: {
      openai: ["openai:user@example.com", "openai:api-key-backup"]
    },
    order: {
      openai: ["openai:user@example.com", "openai:api-key-backup"]
    }
  }
}
```

With this setup, if `gpt-4o` hits a rate limit:

1. OpenClaw retries the same model up to 10 times with exponential backoff.
2. If exhausted, it rotates to `api-key-backup` on the same model.
3. If that also fails, it falls back to `claude-sonnet-4-20250514`.
4. If that fails, it tries `gpt-4o-mini`.
5. Only after **all** candidates are exhausted does the user see a failure.

You can also opt into the auth skip cache to avoid hammering a known-bad profile:

```bash
export OPENCLAW_FALLBACK_SKIP_TTL_MS=60000
```

## 🔗 Related Concepts

- [Model Providers & Selection](concepts/model-providers.md)
- [Models — Selection & Fallback Overview](concepts/models.md)
- [Retry Policy — Pacing and Exclusions](concepts/retry.md)
- [OAuth — Auth Profiles and Tokens](concepts/oauth.md)
- [Gateway Configuration — `agents.defaults.model`](gateway/configuration.md)

## 🤔 Quick Check

Imagine your configured primary is `openai/gpt-4o` with fallbacks `["anthropic/claude-sonnet-4", "openai/gpt-4o-mini"]`. A user explicitly runs `/model openai/gpt-4o` for a session. The model returns a `429 rate limit` after 10 retries, and both auth profiles are exhausted. **What happens next — does OpenClaw fall back to Claude, or report failure? Why?**

*(Answer: It reports failure. Explicit user model selections are strict — OpenClaw does not silently fall back. Fallback chains only apply to configured defaults, cron job primaries, and agent primaries with explicit fallbacks configured.)*

---
*Lesson 12 of Phase 2 (Gateway, Configuration & Safety) | Streak: 3 days*
