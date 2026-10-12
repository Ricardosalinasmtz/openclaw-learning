# 🎓 OpenClaw Lesson: Agent Loop Lifecycle
**Date:** 2026-10-09 | **Day:** 8 of learning streak
**Source:** concepts/agent-loop.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today we follow a single message through the OpenClaw agent loop — the serialized, per-session run that turns your text into actions and a reply. We'll see what happens between "send" and "final message," how runs are queued so sessions never race, and which events you can observe along the way.

## 📖 The Concept

When a message arrives, OpenClaw's `agent` RPC validates the params, resolves the session, persists session metadata, and returns `{ runId, acceptedAt }` immediately — the actual turn then runs in the background. The run follows a fixed sequence: session and workspace preparation (skills and bootstrap context files are loaded into the system prompt), model inference, streaming of assistant and tool deltas, tool execution, and finally persistence of the reply into the transcript.

A key property is serialization: runs are queued per session key (the session lane) and optionally through a global lane, so two turns can never race on the same session at once. To prevent a superseded run from writing stale data into the transcript, each admitted run records a durable `activeWriterRunId` claim, and every transcript append or rewrite must prove it still holds that claim inside the same commit transaction.

As the run proceeds, three event streams are exposed: `lifecycle` (start → finishing → end/error), `assistant` (streamed text deltas), and `tool` (tool start/update/end). The `agent.wait` call waits on lifecycle end/error for a given `runId` — and a timeout there means "I stopped waiting," not "the run was stopped." The run keeps going until its own budget or an abort ends it.

Finally, hooks give extension points all the way through the loop (`before_model_resolve`, `before_prompt_build`, `before_tool_call`, `agent_end`, and more), so plugins can observe or intercept each phase without changing the core sequence.

## 🔑 Key Takeaways

- `agent` returns immediately with a `runId`; the turn runs asynchronously and streams `lifecycle`, `assistant`, and `tool` events.
- Runs are serialized per session lane, and the writer-claim fence stops a superseded run from corrupting the transcript.
- `agent.wait` timing out does not cancel the run — it just reports `status: "timeout"`, and you can wait on the same `runId` again.
- Timeouts are layered: a 30s wait default, a 48h execution budget, and a model idle watchdog (120s cloud / 300s self-hosted).
- Plugin hooks like `before_tool_call` and `agent_end` run inside the loop; a `{ block: true }` decision on `before_tool_call` is terminal.

## 💡 Practical Example

```
# 1. Start a run (gateway RPC or CLI)
openclaw agent --session agent:main:telegram:42 \
  --message "Summarize the repo's test coverage"
# → { "runId": "run_9f2c…", "acceptedAt": 1762645200 }

# 2. Wait for completion (wait-only, 30s default)
agent.wait(runId: "run_9f2c…", timeoutMs: 60000)
# → { "status": "ok", "startedAt": …, "endedAt": …,
#     "terminalReply": "…coverage is at 87%…" }
```

While it runs, the UI subscribes to the run's streams: `lifecycle {phase: "start"}` → chunks of `assistant` deltas as tokens are generated → `tool` events while `exec` runs the coverage script → `lifecycle {phase: "end"}`. If the run takes longer than the wait deadline, `agent.wait` returns `{ status: "timeout" }` — the run is still executing, and waiting on the same `runId` again lets you observe its completion.

## 🔗 Related Concepts

- [System Prompt Assembly](concepts/system-prompt.md) — what is built before inference starts
- [Compaction](concepts/compaction.md) — what happens when the conversation gets too long
- [Command Queue](concepts/queue.md) — how message queue modes feed the session lane
- [Exec Approvals](tools/exec-approvals.md) — an approval gate that can pause execution budget inside the loop

## 🤔 Quick Check

Your long analysis has been running for 20 minutes and you send a new message into the same session while it is still in flight. Your client's wait on the original `runId` times out after 30 seconds and reports `status: "timeout"`. Using the loop's queueing and writer-claim behavior, explain what happens to your new message, and whether the timed-out wait affected the still-running analysis.

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Your new message joins the session lane's queue and waits for the current run to finish, because runs are serialized per session key — it will not run in parallel. The wait timeout is wait-only: it merely stopped observing, and the analysis continues under its own execution budget (48h by default). When your new turn is finally admitted, the original run's writer claim is gone, so it can no longer commit to the transcript.

</details>

---
*Lesson 17 of Phase 1 (Core Concepts) | Streak: 8 days*
