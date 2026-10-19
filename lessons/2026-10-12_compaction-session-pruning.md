# 🎓 OpenClaw Lesson: Compaction & Session Pruning
**Date:** 2026-10-12 | **Day:** 9 of learning streak
**Source:** concepts/compaction.md, concepts/session-pruning.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Two complementary mechanisms keep long conversations from overflowing the model's context window: **compaction** summarizes older turns into a compact entry, while **session pruning** trims bulky tool results. Together they let a session run for days instead of stalling at the context limit — and pruning is also what makes Anthropic prompt caching pay off.

## 📖 The Concept

**Compaction** is the heavy machinery. When a session approaches the model's context limit — or when the model returns a context-overflow error — OpenClaw summarizes older conversation turns into a compact summary that is saved in the session transcript. Recent messages stay intact, and assistant tool calls are always kept paired with their matching results: if the split point would land inside a tool block, the boundary moves so the pair survives together. The full history is never deleted from disk; compaction only changes what the model sees on the next turn.

Auto-compaction is on by default: it fires at a size threshold, or as overflow recovery (compact, then retry). Before compacting, OpenClaw reminds the agent to save important notes to memory files, so durable facts survive the summarization. You can force it manually with `/compact`, optionally with a focus hint like `/compact Focus on the API design decisions`. New configs default to a "safeguard" mode that audits summary quality — if no summary passes validation, compaction stops before writing and the original history is kept as-is.

**Session pruning** is the lighter complement. Instead of rewriting the conversation, it trims old *tool results* — exec output, file reads, search results — from the model's context. Only `toolResult` messages are eligible; normal chat text is never touched. The main mode is `cache-ttl`: pruning waits for the prompt cache's TTL to expire, then soft-trims oversized results (first and last ~1,500 characters kept, with `...` between), and if context usage is still high, hard-clears older results to a placeholder such as `[Old tool result content cleared]`. Two safety rules always apply: the last three assistant turns are never pruned, and nothing before the session's first user message is ever pruned (which protects bootstrap reads like `SOUL.md` and `USER.md`).

The division of labor is clean: pruning keeps tool output lean *between* compaction cycles (and keeps the cached prompt prefix stable for Anthropic), while compaction handles the rest by summarizing. Compaction summarizes the whole conversation and persists the summary in the transcript; pruning trims tool results only and keeps the trimmed view as a projection that survives Gateway restarts via a hidden transcript marker.

## 🔑 Key Takeaways

- Compaction summarizes old turns into a saved entry; pruning trims tool results — they complement each other rather than competing.
- Full history is always preserved on disk; both mechanisms only change what the model sees on the next turn.
- `/compact` forces compaction (optionally with a focus hint); `/new` starts a fresh session without compacting.
- Pruning's `cache-ttl` mode is tuned around Anthropic prompt caching and is off by default for other providers.
- Tool calls always stay paired with their results, and the last three assistant turns are never pruned.

## 💡 Practical Example

A research session has run 200 web searches, and tool results now dominate its context. Enable pruning:

```json5
{
  agents: {
    defaults: {
      contextPruning: { mode: "cache-ttl", ttl: "5m" },
    },
  },
}
```

Once the cache TTL expires, results over ~4,000 characters are soft-trimmed to their first and last chunks, and if context usage is still high, older results are hard-cleared to a placeholder. The session continues with less bloat, and the next compaction has far less work to do. If context feels stale after an auto-compaction, guide it: `/compact Focus on the API design decisions`.

## 🔗 Related Concepts

- Context: What the Model Sees (concepts/context.md)
- Memory System (concepts/memory.md)
- Session Management (concepts/session.md)
- Heartbeat Automation (gateway/heartbeat.md)

## 🤔 Quick Check

You're running a long coding session on a direct Anthropic API key, and you notice it auto-compacts every few hours even though the conversation text itself is small. What is most likely inflating the context, and which feature would you enable to make compaction less frequent?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** The bloat is accumulated tool results (exec output, file reads, search results), not the conversation length. Enable `agents.defaults.contextPruning.mode: "cache-ttl"`: with a direct Anthropic API key, OpenClaw delegates to server-side tool-result clearing, which strips old results before the model sees them and keeps the cached prefix stable, so compaction is triggered less often. Gating on the cache TTL matters because clearing rewrites the context prefix and would invalidate the prompt cache — doing it only after the cache has expired (and clearing enough tokens to justify a fresh cache write) keeps the cost savings real.

</details>

---
*Lesson 18 of Phase 1 (Core Concepts) | Streak: 9 days*
