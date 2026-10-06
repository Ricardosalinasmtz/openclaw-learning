# 🎓 OpenClaw Lesson: Thinking Levels & Directives
**Date:** 2026-09-30 | **Day:** 1 of learning streak
**Source:** tools/thinking.md | **Difficulty:** beginner

## 🎯 What You'll Learn

How OpenClaw controls "how hard" a model thinks before answering — the `/think` directive ladder, its resolution order, and how it interacts with fast mode, verbose output, and reasoning visibility.

## 📖 The Concept

OpenClaw exposes a graduated "thinking level" scale, roughly mirroring the old Anthropic "think" < "think hard" < "think harder" < "ultrathink" ladder: `off, minimal, low, medium, high, xhigh, adaptive, max, ultra`. You set it inline in any message with `/t <level>`, `/think:<level>`, or `/thinking <level>`. A directive-only message (just the command, nothing else) makes it sticky for the session; an inline directive attached to a real message only applies to that one turn.

Not every model supports every level — provider plugins declare which levels a given model/runtime actually accepts (this is called a "thinking profile"). If you ask for an unsupported level, OpenClaw rejects it and tells you the valid options for that model. Stored levels that become unsupported (e.g., you switch models) get remapped automatically by rank — `adaptive` falls back to `medium`, `xhigh`/`max` fall back to the highest level the new model actually supports.

Resolution happens in a clear priority order: (1) inline directive on the current message, (2) a stored session override, (3) the agent's configured default, (4) the global default, (5) a provider-declared fallback. This means you can set a personal default once via a directive-only message and then override it ad hoc per-message without losing your baseline.

Thinking levels are one of three related but distinct directive families: `/fast` controls processing priority/speed (not reasoning depth), `/verbose` controls how much tool-call detail gets echoed back to you, and `/reasoning` controls whether the model's internal thinking text itself is shown in the reply (as a separate "Thinking" message) versus kept hidden.

## 🔑 Key Takeaways

- `/think:<level>` (or `/t`, `/thinking`) sets reasoning depth; send it alone to make it the session default, or inline to affect just one message.
- Resolution order is: inline > session override > per-agent config default > global default > provider fallback.
- Unsupported levels are rejected with a hint listing what the current model actually supports — you never silently get ignored.
- `/think`, `/fast`, `/verbose`, and `/reasoning` are separate independent directives — depth, speed, tool-detail, and visibility of thinking are four different knobs.

## 💡 Practical Example

```
# Set a personal default for this session (sticky):
/think:high

# Just for this one question, think less (doesn't change your default):
/think:low What's 2+2?

# Check what's currently active:
/think

# Clear your override and fall back to the configured/provider default:
/think default
```

## 🔗 Related Concepts

- Fast mode (`/fast`) — processing priority, separate from reasoning depth
- Reasoning visibility (`/reasoning`) — whether thinking text is shown in replies
- Elevated mode — see /tools/elevated

## 🤔 Quick Check

If you set `/think:high` as your session default, then send one message with `/think:low` attached to a real question — what thinking level applies to your *next* message after that?

(Answer: `high` — the inline `/think:low` only affected that single message; your sticky session override of `high` is still in effect.)

---
*Lesson 10 of Phase 1 (Core Concepts) | Streak: 1 day*
