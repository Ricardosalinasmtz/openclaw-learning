# 🎓 OpenClaw Lesson: System Prompt Assembly
**Date:** 2026-10-08 | **Day:** 7 of learning streak
**Source:** concepts/system-prompt.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today we look under the hood at how OpenClaw assembles the system prompt it sends to the model on every run. We'll cover the three-layer assembly pipeline, the fixed section layout, the internal prompt cache boundary that keeps prefix caching working, and the prompt modes that shrink the prompt for sub-agents.

## 📖 The Concept

OpenClaw builds its own system prompt for every agent run — there is no runtime default prompt to fall back on. Assembly happens in three layers. First, `resolveAgentSystemPromptConfig` resolves the config-backed prompt knobs for a specific agent: owner display, TTS hints, model aliases, memory citation mode, and sub-agent delegation mode. Second, `buildAgentSystemPrompt` renders the prompt from explicit inputs — it stays a pure renderer and never reads global config directly. Third, runtime adapters (embedded, CLI, command/export previews, compaction) gather live facts — tools, sandbox state, channel capabilities, context files, and provider prompt contributions — and call the configured prompt facade. This split keeps exported and debug prompt surfaces aligned with live runs without turning every runtime detail into one monolithic builder.

The rendered prompt is compact, with fixed sections: Tooling, Execution Bias, Promised Work, Safety, Runtime Context, Skills (when available), OpenClaw Control, Workspace, Documentation, Workspace Files (injected), Sandbox (when enabled), Temporal Context, Assistant Output Directives, Runtime, and Reasoning. Provider plugins can contribute cache-aware guidance without replacing the OpenClaw-owned prompt: they may replace one of three named core sections (`interaction_style`, `tool_call_style`, `execution_bias`), inject a **stable prefix** above the prompt cache boundary, or inject a **dynamic suffix** below it. The built-in GPT-5-family contribution, for example, adds a stable behavior contract plus an optional friendly-tone override.

The most important design idea is the internal prompt cache boundary. Large stable content — Project Context and the static Memory Recall instructions — stays above the boundary, while volatile per-turn sections (Runtime, UI Presentation, channel-specific hints, the current elevated level) are appended below it. That lets backends with prefix caches reuse the stable workspace prefix across channel turns. Temporal Context (date and time zone) deliberately sits below the boundary, so a day rollover doesn't invalidate the cached prefix.

Finally, prompt modes size the prompt per run: `full` (default, all sections), `minimal` (used for sub-agents — it omits the memory prompt section, Model Aliases, User Identity, Assistant Output Directives, Messaging, Collapsible Details, and Silent Replies), and `none` (just the base identity line). Workspace bootstrap files (AGENTS.md, SOUL.md, IDENTITY.md, USER.md, MEMORY.md) are injected through the same surface and truncated at configurable limits (20,000 chars per file and 60,000 total by default), with a built-in truncation notice when a file gets cut. Sub-agent sessions inject only AGENTS.md, keeping their context small.

## 🔑 Key Takeaways

- The system prompt is assembled in three layers: config resolution → pure renderer → runtime adapters supplying live facts; the renderer never reads global config directly.
- An internal cache boundary separates stable content (the reusable prefix) from volatile per-turn content, so prefix caching survives channel turns and day rollovers.
- Provider plugins can tune the prompt — three named core sections, a stable prefix, or a dynamic suffix — without replacing the OpenClaw-owned prompt.
- Prompt modes (`full`/`minimal`/`none`) shrink the prompt for sub-agents, and sub-agent sessions only inject AGENTS.md.

## 💡 Practical Example

The model learns about what is running in its environment through the Runtime Context carrier — compact facts about active exec sessions, sub-agents, and media-generation progress that travel with each user request:

```
<<<BEGIN_OPENCLAW_INTERNAL_CONTEXT>>>
Active exec sessions: npm test (running, pid 4211)
Active Subagents: none
<<<END_OPENCLAW_INTERNAL_CONTEXT>>>
```

The prompt tells the model to use this data without replying to or describing it, and the newest snapshot supersedes older ones. To see what's actually being injected, run `/context list` or `/context detail` in chat — it shows raw vs injected sizes per bootstrap file plus any truncation. That's the fastest way to debug "why is my model behaving this way?": the answer is often in the system prompt layers, not in the chat history.

## 🔗 Related Concepts

- [Context: What the Model Sees](concepts/context.md)
- [Agent Runtime & Workspace](concepts/agent-workspace.md)
- [Context engine](concepts/context-engine.md)
- [Session Management & Routing](concepts/session.md)

## 🤔 Quick Check

You're debugging why your agent stopped mentioning a background command it started two hours ago. The Tooling section of the prompt tells the model to rely on push-based completion wakes instead of polling. Given how the system prompt is assembled, where does the model currently learn about that running command, and what should you tell the operator to inspect the actual injected prompt content?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** The model learns about it from the Runtime Context carrier — the compact snapshot of active exec sessions delivered with each user message, where the newest snapshot supersedes older ones. To inspect the actual injected content, run `/context list` or `/context detail` in chat, which shows the injected sections, their sizes, and any truncation.

</details>

---
*Lesson 16 of Phase 1 (Core Concepts) | Streak: 7 days*
