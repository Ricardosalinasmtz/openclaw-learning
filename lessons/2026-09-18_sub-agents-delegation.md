# 🎓 OpenClaw Lesson: Sub-agents & Delegation
**Date:** 2026-09-18 | **Day:** 6 of learning streak
**Source:** `tools/subagents.md` | **Difficulty:** Intermediate

## 🎯 What You'll Learn

How to spawn background agent runs that execute in parallel without blocking your main session — and how to wait for their results cleanly without polluting your workflow with polling loops.

## 📖 The Concept

Sub-agents are **background agent runs** spawned from an existing agent session. Each one runs in its own isolated session (`agent:<agentId>:subagent:<uuid>`) and announces its result back to the requester for review. Think of them as handing off a task to a colleague who works on it independently and reports back when done.

### Why sub-agents exist

They solve three problems:

1. **Parallelism** — Research, long tasks, and slow tool work don't have to block your main conversation. Spawn a sub-agent and keep working.
2. **Isolation** — Each sub-agent starts with clean context (`context: "isolated"` is default), keeping token usage lower and preventing one task's context from bleeding into another.
3. **Controlled tool surface** — Sub-agents don't get session or message tools by default, reducing the chance of accidental misuse.

### Context modes: `isolated` vs `fork`

| Mode | When to use | Behavior |
|------|-------------|----------|
| `isolated` | Fresh research, independent work, anything that can be briefed in the task text | Creates a clean child transcript. Default for non-thread spawns. |
| `fork` | Work that depends on the current conversation, prior tool results, or nuanced instructions already present | Branches the requester transcript into the child session before it starts. |

Use `fork` sparingly — it's for context-sensitive delegation, not a replacement for writing a clear task prompt.

### Completion delivery: push-based, not polling

This is the most important architectural detail: **sub-agent completion is push-based**. When you spawn a child, OpenClaw hands the result back through an `agent` turn — it doesn't ask you to check.

The workflow is:
1. **Spawn** with `sessions_spawn` → returns a run id immediately (non-blocking)
2. **Yield** with `sessions_yield` when you need the result before replying
3. **Review** the announced result in the next turn

**Never** poll `subagents list`, `sessions_list`, or `sessions_history` in a loop to wait for completion. That's an anti-pattern. Use `sessions_yield` instead.

### Key modes and options

- **`visible: true`** — Creates a persistent dashboard session that shows in the sidebar. Use this for work the user will watch, return to, or steer (coding, multi-step work, reports with deliverables).
- **`runtime: "acp"`** — For external ACP harnesses (Claude Code, Gemini CLI, Codex). OpenClaw hides this until ACP is enabled.
- **`cleanup: "delete"`** — Archives the session immediately after announce (keeps transcript via rename).
- **`thread: true`** — Binds the sub-agent to a channel thread so follow-up messages in that thread route to the same session (supported on Discord, iMessage, Matrix, Telegram).
- **`taskName`** — A stable handle (e.g., `review_docs`) for later targeting with `/subagents info review_docs`.

### Swarm mode

For large parallel fan-out (five or more similar children), use `collect: true` with an `outputSchema`. Swarm collectors don't send completion notifications — you explicitly collect their results with `agents_wait`. Reserve this for batching, not for one or a few tasks.

### Cost awareness

Each sub-agent has its own context and token usage. For heavy or repetitive tasks:
- Set a cheaper model via `agents.defaults.subagents.model`
- Keep the main agent on a higher-quality model
- Use `context: "isolated"` to keep token use lower unless you need `fork`

## 🔑 Key Takeaways

- **Spawn with `sessions_spawn`**, wait with **`sessions_yield`** — never poll.
- **`isolated`** is default; **`fork`** when you need the parent's conversation context.
- **`visible: true`** for work users will watch or return to; omit for internal/background work.
- **Completion is push-based** — OpenClaw announces results back automatically.
- **Sub-agents don't get message tools** — they return plain text to the parent; human-visible replies stay with the parent.
- **Swarm mode** (`collect: true`) is for large fan-out (5+ children); ordinary spawns for 1–4.

## 💡 Practical Example

A typical orchestration pattern:

```
# 1. Spawn two independent sub-agents in parallel
sessions_spawn(task="Research latest AI Act amendments", taskName="research_aiact", visible=false)
sessions_spawn(task="Write executive summary of findings", taskName="write_summary", visible=false)

# 2. When you need the research result before replying
sessions_yield()
# → The announced result from research_aiact arrives here

# 3. Review the result and proceed
# The child's result is a report/evidence — synthesize it, don't forward raw
```

For visible sessions (user-facing work):

```
sessions_spawn(
  task="Build a dashboard for project tracking",
  label="Dashboard development",
  visible: true,
  worktree: true,
  worktreeName: "dashboard-work"
)
# → Returns sessionUrl and owner; user can see progress in sidebar
```

## 🔗 Related Concepts

- [Sessions Overview](/concepts/session.md) — Session lifecycle and routing
- [Exec Tool](/tools/exec.md) — Shell execution (sub-agents use this for code work)
- [Automations & Scheduling](/automation/index.md) — Cron jobs that can spawn sub-agents
- [Tool Policy](/tools/index.md) — Which agents have `sessions_spawn` available

## 🤔 Quick Check

1. You spawn a sub-agent to analyze a dataset. Ten minutes later you need to reply to the user but the sub-agent might still be running. Which tool do you use to wait for it — `subagents list` in a loop, or `sessions_yield`? Why?
2. When would you choose `context: "fork"` over the default `isolated`? Give one concrete example.

---
*Lesson 7 of Phase 3 (Multi-agent, Channels & Extensibility) | Streak: 6 days*
