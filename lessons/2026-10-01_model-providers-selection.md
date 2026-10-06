# 🎓 OpenClaw Lesson: Model Providers & Selection
**Date:** 2026-10-01 | **Day:** 2 of learning streak
**Source:** concepts/model-providers.md, concepts/models.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

How OpenClaw routes requests to different AI providers (OpenAI, Anthropic, Google, and 30+ others), and how model selection determines which provider, model, and runtime your agent uses for every interaction.

## 📖 The Concept

### Model Providers: The Who

A **model provider** in OpenClaw is an AI service that supplies language models — not a chat channel like Telegram or Discord. Each provider has its own authentication method, model catalog, and sometimes its own runtime.

OpenClaw ships with **official provider plugins** for 30+ providers. When you enable one and set its auth, the provider publishes its own model catalog rows automatically — you don't need to manually register every model. These include:

- **OpenAI** (`openai`) — API key or ChatGPT/Codex OAuth
- **Anthropic** (`anthropic`) — API key or Claude CLI reuse
- **Google Gemini** (`google`) — API key or Vertex AI
- **OpenRouter** (`openrouter`) — API key (proxy to many models)
- **xAI** (`xai`) — SuperGrok OAuth or API key
- **DeepSeek, Groq, Cohere, Mistral**, and many more

Plus **custom providers** via `models.providers` — useful for OpenAI/Anthropic-compatible proxies, self-hosted endpoints (Ollama, LM Studio), or when you need to override a default base URL.

Every model ref follows the pattern `provider/model`, for example `openai/gpt-5.6-sol`, `anthropic/claude-opus-5`, or `google/gemini-3.1-pro-preview`.

### Model Selection: The Which

When OpenClaw needs to generate a response, it resolves the model through a strict priority order:

1. **Primary model** — `agents.defaults.model.primary` (or `agents.defaults.model` as a plain string). This is your default starting point.
2. **Fallbacks** — `agents.defaults.model.fallbacks`, tried in order if the primary fails.
3. **Auth failover** — within a provider, if one API key hits a rate limit, OpenClaw rotates to the next key *before* moving to the next fallback model.

The source of the model selection matters:
- **Configured default** uses the full fallback chain
- **User-selected** (`/model <model>`) is strict — no automatic fallback if it becomes unreachable
- **Session-pinned** survives `/new`, `/reset`, and compaction
- **Cron jobs** use their configured primary + fallbacks unless `fallbacks: []` is set for strict runs

### Quick Policy Rules

- Set your primary to the strongest latest-generation model available to you.
- Use fallbacks for cost/latency-sensitive tasks and lower-stakes chat.
- For tool-enabled agents or untrusted inputs, avoid older/weaker model tiers.
- `modelPolicy.allow` is an optional override allowlist — omit or set `[]` to allow any model, or narrow it to specific providers using trailing wildcards like `["openai/*", "anthropic/*"]`.

### API Key Rotation

For providers that support it, you can configure multiple keys. Rotation kicks in **only on rate-limit responses** (429, quota exceeded, etc.) — non-rate-limit failures fail immediately without trying the next key. Priority order: `OPENCLAW_LIVE_<PROVIDER>_KEY` > `<PROVIDER>_API_KEYS` > `<PROVIDER>_API_KEY` > `<PROVIDER>_API_KEY_*`.

## 🔑 Key Takeaways

- **Provider ≠ chat channel.** Providers supply the AI models; channels (Telegram, Discord, WebChat) deliver the conversation.
- **30+ bundled providers** ship with OpenClaw — enable one and set auth, then pick from its published catalog. No manual model registration needed.
- **Selection order is strict:** primary → fallbacks → auth rotation. User-selected models are strict (no silent fallback).
- **Model refs use `provider/model`** format — the slash separates provider from model ID.
- **API key rotation** protects against rate limits by trying the next key before escalating to the next fallback model.

## 💡 Practical Example

### Setting up a multi-provider setup:

```json5
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-5" },
      modelPolicy: {
        allow: ["anthropic/*", "openai/*", "google/*"],
      },
    },
  },
}
```

### CLI onboarding (no hand-editing config):

```bash
openclaw onboard
# Follow prompts to set up your provider(s)

# Verify your setup
openclaw models status
openclaw models list

# Set the primary model
openclaw models set openai/gpt-5.6-sol
```

### In chat — change model scope:

```
/model openai/gpt-5.4 -s     # Only this session
/model openai/gpt-5.4 -a     # This session + this agent's default
/model openai/gpt-5.4 -g     # Global default (owner/admin only)
/model default               # Clear session pin, return to configured default
/model list                  # Browse available models
```

## 🔗 Related Concepts

- [Model Failover & Resilience](/concepts/model-failover) — cooldowns, auto-fallback recovery, and auth profile rotation mechanics
- [Agent Runtimes](/concepts/agent-runtimes) — OpenClaw vs. Codex vs. CLI backends
- [Thinking Levels](/tools/thinking) — fast mode and reasoning effort tied to model provider
- [Session State Awareness](/concepts/session-state) — how model pins survive session lifecycle events

## 🤔 Quick Check

If your primary model is `anthropic/claude-opus-5` and it's returning 429 rate-limit errors, what happens? Does OpenClaw immediately fall back to `openai/gpt-5.4`?

<details>
<summary>💡 Answer</summary>
No. OpenClaw first tries API key rotation within the Anthropic provider (if multiple keys are configured). Only after all Anthropic keys fail would it move to the next configured fallback model. Auth failover happens *before* cross-provider fallback.
</details>

---
*Lesson 11 of Phase 1 (Core Concepts) | Streak: 2 days*
