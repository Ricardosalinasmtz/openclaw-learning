# 🎓 OpenClaw Lesson: Memory Search & Hybrid Retrieval
**Date:** 2026-10-13 | **Day:** 10 of learning streak
**Source:** concepts/memory-search.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today we unpack how `memory_search` finds the right note when the query uses different words than the note. You'll see the hybrid retrieval pipeline (vector search + BM25 keyword search), the deterministic ranking that blends relevance, recency, and importance, and the fallback behavior when embeddings are unavailable.

## 📖 The Concept

`memory_search` chunks your memory files into small pieces and searches them with two parallel retrieval paths: **vector search**, which matches similar meaning ("gateway host" finds "the machine running OpenClaw"), and **BM25 keyword search**, which matches exact terms like IDs, error strings, and config keys. A separate filename index means exact full paths, basenames, and filename stems rank ahead of partial path matches. The two result sets are merged with a weighted merge, so a query gets both the semantic coverage of embeddings and the precision of keyword matching.

On top of the hybrid relevance score, the builtin engine applies deterministic ranking: `hybrid relevance × recency decay × importance multiplier`. Importance is scored once when a memory workflow writes an entry (missing importance is neutral, so old indexes keep working). Dated daily notes decay with a 30-day half-life, while curated files like `MEMORY.md` and `USER.md` are evergreen. Finally, an MMR (maximal marginal relevance) pass reorders the candidate set to reduce redundant near-duplicate snippets — it changes ordering only, not scores.

The provider story matters in practice: OpenAI embeddings are the default, but you can switch to `gemini`, `voyage`, `mistral`, `bedrock`, `local` (managed llama.cpp GGUF, no API key), `ollama`, `lmstudio`, `github-copilot`, or any `openai-compatible` endpoint. Setting `provider: "none"` gives deliberate FTS-only (keyword-only) search. A subtle but important failure mode: if you explicitly name a provider and it becomes unavailable at request time (bad auth, network failure), `memory_search` reports memory as unavailable instead of silently degrading to keyword-only results — a broken provider stays visible. Only `"auto"` (or unset) silently falls back to keyword ranking.

Two extras are worth knowing. **Deterministic trigger recall** compares the inbound message with short trigger phrases stored on indexed entries and can inject up to three compact entries into hidden context on eligible interactive turns — but only promoted, trusted entries qualify (root `MEMORY.md` and `USER.md`), never daily notes or transcripts. **Session memory search** is an opt-in experimental complement: set `experimental.sessionMemory: true` and add `"sessions"` to `sources`, so `memory_search` can semantically recall earlier conversations, while `sessions_search` remains the exact full-text path for transcripts.

## 🔑 Key Takeaways

- Hybrid search runs vector (semantic) and BM25 (keyword) retrieval in parallel and weighted-merges them, so paraphrases and exact identifiers both hit.
- Final ranking is deterministic: hybrid relevance × recency decay (30-day half-life for dated notes; `MEMORY.md`/`USER.md` evergreen) × importance multiplier, followed by an MMR pass that only reorders for diversity.
- Explicit providers fail loudly (reported unavailable) instead of silently degrading; `provider: "none"` is the deliberate keyword-only mode, and `"auto"` silently falls back.

## 💡 Practical Example

You notice the agent keeps missing a note about a config key you wrote a month ago. Work through it like this:

```bash
# 1. Check the index health and which provider is active
openclaw memory status
openclaw memory status --deep

# 2. Index is empty or stale? Force a rebuild (also fixes CJK keyword issues)
openclaw memory index --force

# 3. Want local embeddings without an API key?
openclaw plugins install @openclaw/llama-cpp-provider
```

Then in config:

```json5
{
  memory: {
    search: {
      provider: "local" // managed llama.cpp GGUF (~0.3 GB), no API key needed
    }
  }
}
```

And in the agent, just ask: `memory_search` with a query like "how did we decide to configure the gateway port?" — semantic matching finds the note even if it used different wording, while BM25 picks up the exact config key `gateway.port` if you include it in the query. If you *only* get keyword matches after a provider switch, run `openclaw memory status --deep` — the embedding provider likely isn't configured.

## 🔗 Related Concepts

- [Memory overview](/concepts/memory)
- [Active memory](/concepts/active-memory)
- [Builtin memory engine](/concepts/memory-builtin)
- [Session search](/concepts/session-search)
- [Memory configuration reference](/reference/memory-config)

## 🤔 Quick Check

You switch your gateway from OpenAI embeddings to a local Ollama deployment and everything seems fine for a week, but one day the Ollama server restarts and fails to come back up. A user then asks the agent a question whose answer exists verbatim in a daily note. What does `memory_search` do in this scenario, and why might that behavior be preferable to silently returning keyword-only results?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Because Ollama was explicitly configured, `memory_search` reports memory as unavailable instead of silently degrading to keyword-only results. The verbatim answer would actually be findable by BM25 keyword matching, but the explicit-failure behavior is deliberate: it keeps a broken configured provider visible instead of letting you run for days believing semantic search is working when it has quietly degraded to keywords. You fix the Ollama deployment (or set `provider: "auto"`/`"none"` if you want a fallback), and the semantic path is restored on purpose rather than by accident.

</details>

---
*Lesson 19 of Phase 1 (Core Concepts) | Streak: 10 days*
