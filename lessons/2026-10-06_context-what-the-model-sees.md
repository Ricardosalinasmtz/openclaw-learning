# 🎓 OpenClaw Lesson: Context: What the Model Sees
**Date:** 2026-10-06 | **Day:** 5 of learning streak
**Source:** concepts/context.md | **Difficulty:** beginner

## 🎯 What You'll Learn

Today we pin down what "context" actually means in OpenClaw: the exact set of tokens sent to the model for each run, what makes it up, and how to inspect it when behavior seems off. By the end you'll know which parts of the window are fixed costs, which ones grow with your session, and the commands that let you see the bill before it blows the window.

## 📖 The Concept

Context is everything OpenClaw sends to the model for a run, bounded by the model's context window (its token limit). A useful mental model has three buckets: the **system prompt** (OpenClaw-built each run — rules, tool list, skills list, time/runtime metadata, and your injected workspace files), the **conversation history** (your messages plus the assistant's replies in this session), and **tool calls/results plus attachments** (command output, file reads, images, audio).

A classic source of confusion is that context is not the same thing as memory. Memory can live on disk and be reloaded later; context is strictly what's inside the model's current window at the moment of the run. If something isn't in the window, the model can't "know" it — no matter how many times it was said earlier or written to a file it hasn't re-read.

The system prompt itself has several cost centers worth knowing. Tool definitions count **twice**: once as the short tool list text you can see, and again as the full JSON schemas that are sent to the model but never shown as plain text. Workspace bootstrap files (`AGENTS.md`, `SOUL.md`, `IDENTITY.md`, `USER.md`, `BOOTSTRAP.md`) are injected verbatim under "Project Context", truncated per file (`agents.defaults.bootstrapMaxChars`, default 20000 chars) and capped in total (`bootstrapTotalMaxChars`, default 60000 chars) — and when truncation happens, the model gets a notice that some files were cut. Skills are cheap by design: the prompt only carries a compact list of names, descriptions, and locations; the actual skill instructions are read on demand with the `read` tool when needed.

The window is finite, so OpenClaw has two distinct relief valves. **Compaction** summarizes older conversation history into a compact summary entry that persists in the transcript, keeping recent messages intact. **Pruning** drops old tool results from the in-memory prompt to free space, but does *not* rewrite the session transcript — the full history stays inspectable on disk. Knowing which one applies matters when you're debugging "why did it forget that."

## 🔑 Key Takeaways

- Context = system prompt + conversation history + tool calls/results + attachments; everything the model receives counts, including hidden provider wrappers.
- Context ≠ memory: only what's in the current window is "known" to the model.
- Tools have two costs: visible list text plus invisible JSON schemas; schemas can dominate (check `/context detail`).
- Skills are listed cheaply in the prompt and loaded on demand with `read` — that's why skill overhead stays small.
- `/status`, `/context list`, `/context detail`, and `/context map` let you audit what's in the window and how big each contributor is.
- Compaction rewrites (summarizes) the transcript; pruning only trims in-memory tool results and leaves the transcript on disk untouched.

## 💡 Practical Example

Session has been running long and the model keeps "forgetting" early decisions. Instead of guessing, audit the window:

```text
> /status            # quick: how full is the window + session settings
> /context list      # per-file injected sizes + totals
> /context detail    # top tool schemas, top skill entries, system prompt size
> /context map       # treemap image of everything eating the window
```

A typical `/context list` reading shows the fixed costs:

```text
System prompt (run): 38,412 chars (~9,603 tok) (Project Context 23,901 chars)
Tool schemas (JSON): 31,988 chars (~7,997 tok)
Session tokens (cached): 14,250 total / ctx=32,000
```

If the conversation group is what's swollen, run `/compact` to summarize older history into a compact entry. If it's old tool results bloating the in-memory prompt, pruning handles that without touching the transcript. Either way you can see before you act which bucket is the problem — that inspection loop is the core skill today.

## 🔗 Related Concepts

- [System Prompt Assembly](/concepts/system-prompt) — the other queued lesson on how the fixed-cost half is built
- [Compaction](/concepts/compaction) — summarizing long conversations to stay inside the window
- [Session Pruning](/concepts/session-pruning) — dropping old tool results from the in-memory prompt
- [Agent Loop](/concepts/agent-loop) — where context assembly sits in the run cycle

## 🤔 Quick Check

You're running a long analysis session and the model starts overlooking details from early in the conversation. You run `/context list` and see the system prompt and tool schemas are a small, stable fixed cost, while the conversation transcript is the dominant contributor and has grown past the point where the model's recall has degraded. Which mechanism — compaction or pruning — do you reach for first, and why is the other one not the right tool for this specific symptom?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Compaction, via `/compact`. The dominant cost is the conversation transcript itself, and compaction is the mechanism that summarizes older history into a compact entry that persists in the transcript, freeing window space while keeping recent messages intact. Pruning is not the right tool here because it only drops old tool results from the in-memory prompt — it doesn't rewrite the transcript or shrink the bloated conversation history, so it wouldn't relieve the symptom that the early-conversation content is drowning the model.

</details>

---
*Lesson 14 of Phase 1 (Core Concepts) | Streak: 5 days*
