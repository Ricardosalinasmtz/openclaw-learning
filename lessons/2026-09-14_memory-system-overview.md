# 🎓 OpenClaw Lesson: Memory System Overview

**Date:** 2026-09-14 | **Day:** 2 of learning streak
**Source:** concepts/memory.md | **Difficulty:** Beginner

## 🎯 What You'll Learn

Understand how OpenClaw agents remember information across sessions — not through hidden AI magic, but through simple, transparent Markdown files on disk. Learn the four memory layers and what goes where.

## 📖 The Concept

OpenClaw has no hidden memory. There is no invisible state the model "knows" between turns unless something is saved to a file. Everything the agent remembers is stored as plain Markdown in your workspace. This means your agent's memory is **auditable, editable, and portable** — just text files anyone can read.

Your agent has four types of memory files:

1. **`USER.md`** — Stable preferences about you (name, timezone, communication style, active projects). Loaded at session start with a dedicated small token budget. This is the *compact user-model layer*.

2. **`MEMORY.md`** — Long-term memory of durable facts, decisions, and patterns. Curated, not raw. It's not a transcript or daily log — it's the distilled wisdom your agent starts each session with.

3. **`memory/YYYY-MM-DD.md`** — Daily notes. Running context, observations, session summaries. Today's and yesterday's load automatically on a fresh session. These are indexed for search but not injected into every turn's context.

4. **`DREAMS.md`** (optional) — A dream diary and consolidation summaries for your review. This is where the "dreaming" system records what it's considered promoting into long-term memory.

The key principle: **write durable facts to `MEMORY.md`, working notes to daily files, and stable user preferences to `USER.md`**. Each file has a distinct job.

Over time, the **dreaming system** (enabled by default) automatically reviews daily notes, scores candidates for durability, and promotes qualified items into `MEMORY.md`. You review what's being promoted in `DREAMS.md`. If `MEMORY.md` grows too large, OpenClaw truncates the injected copy (but keeps the file intact on disk) — this is a signal to move detail into daily notes.

## 🔑 Key Takeaways

- **No hidden state:** Everything is Markdown on disk. If it's not saved, the agent won't remember it.
- **Four layers:** `USER.md` (you), `MEMORY.md` (curated facts), `memory/YYYY-MM-DD.md` (daily notes), `DREAMS.md` (review surface for dreaming).
- **Action-sensitive memories:** For memories that change behavior, include *when* it's safe to act, expiry conditions, and authority boundaries.
- **Dreaming automates curation:** Short-term recall → scored candidates → promoted to long-term memory, all with a human review layer.

## 💡 Practical Example

Here's how action-sensitive memory looks in practice:

```md
The API migration is being designed in another session. Future turns should
not edit the API implementation from this thread; use findings here only as
design input until the migration plan lands.
```

Notice how this captures **what** (the migration is in another session), **what behavior changes** (don't edit the implementation), **the boundary** (design input only), and **when it changes** (after the migration plan lands). Without this context, the agent might incorrectly edit code it shouldn't touch.

## 🔗 Related Concepts

- [Memory Search](/concepts/memory-search) — semantic + keyword search across memory
- [Dreaming](/concepts/dreaming) — background promotion of short-term to long-term memory
- [User Model](/concepts/user-model) — directive-based durable preferences
- [Memory Tools](/reference/memory-config) — `memory_search`, `memory_get`, and `intent`

## 🤔 Quick Check

> Imagine your agent completes a task and you want it to remember a specific preference about how you want future reports formatted. Which memory file should you ask it to write to, and why?

---
*Lesson 3 of Phase 1 (Core Concepts) | Streak: 2 days*
