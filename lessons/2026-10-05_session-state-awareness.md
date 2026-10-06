# 🎓 OpenClaw Lesson: Session State Awareness
**Date:** 2026-10-05 | **Day:** 4 of learning streak
**Source:** `concepts/session-state.md` | **Difficulty:** Advanced

## 🎯 What You'll Learn

When several sessions work on the same problem, each one builds assumptions about the others. Those assumptions go stale as soon as a human or another agent steps in. This lesson covers how OpenClaw notices those interventions, tells the affected session once, and gives it a cheap way to catch up before it acts.

## 📖 The Concept

Session state awareness has three parts: a **durable signal log**, **watchers**, and **reconciliation**. Whenever a session changes in a way that matters, OpenClaw adds a typed event to `session_state_events` in the shared state database. An event holds metadata and a one-line summary. It never holds message content. Each event also names its actor (`human`, `agent`, or `system`). A session's **state version** is the highest sequence number in its log. `session_status` always reports it, and `sessions_list` rows include it once the session has logged changes.

Not every event wakes anyone up. Only three kinds notify watchers: `human_direct_message`, `goal_changed` and `upstream_missing`. Four more are recorded for history only: `run_completed`, `run_failed`, `compacted` and `adopted`. `child_spawned` only sets up the parent's cursor. Ordinary child completions still arrive through sub-agent announcements, so the log never duplicates them.

A **watcher** is a session that keeps a cursor on a target session. Cursors come from three places:
- **Spawn edges.** Spawning a child sets the parent's cursor automatically.
- **Ambient groups.** Under `session.groupScope: "per-group"`, the main session watches its group, room and channel sessions.
- **Explicit watches.** Calling `sessions_send` with `watch: true` starts a watch.

When a notifying event lands, the watcher gets **one** short notice telling it to reconcile. Twenty quick changes still produce a single line. Watchers are never notified about their own actions, and pending notices are rebuilt after a gateway restart.

**Reconciliation** happens on the watcher's side. It calls `session_status` with `changesSince: N` and gets back the typed events after version N (up to 200). This call doesn't advance any cursors. If the result says `historyGap: true`, the requested version is older than the retained history, and the watcher should do a full refresh instead.

## 🔑 Key Takeaways

- **Signal, not content:** the log stores event kinds and summaries, never message text. It's kept for 30 days, up to 50,000 rows, and writes are best-effort.
- **Only three kinds notify:** a human message, a goal change, or a missing upstream. Run completions, failures and compactions are logged only.
- **One notice per watcher/target pair:** the watermark is frozen and acknowledged when the notice is consumed. A new notice opens only if more changes happened in between.
- **Reconcile before acting:** `session_status changesSince N` returns the exact changes. `historyGap: true` means you need a full refresh.
- **v1 limits:** notices don't cross gateway processes, there's no unwatch command, and sessions using `session.scope="global"` get the log but no proactive notices.

## 💡 Practical Example

This setup applies to you directly: sokka sends work to zuko, and you sometimes message zuko's session yourself.

```text
1. sokka spawns zuko-researcher           → cursor seeded automatically (child_spawned)
2. You DM zuko directly on Telegram:      "Skip the 2019 papers"
   → human_direct_message logged (seq 14)
3. sokka's next turn shows ONE notice:
   Session "agent:zuko-researcher:subagent:…" changed (other actor).
   Reconcile before acting: session_status sessionKey "…" changesSince 12.
4. sokka calls:
   session_status { sessionKey: "…", changesSince: 12 }
   → { stateVersion: 14, stateChanges: { events: [
        { sequence: 14, kind: "human_direct_message", actorType: "human",
          summary: "human message via telegram" } ], historyGap: false } }
5. sokka reads zuko's history, sees your instruction, adjusts its plan
   instead of overriding you with a stale brief.
```

To coordinate with a session you didn't spawn:

```json
sessions_send { "sessionKey": "agent:katara-document-writer:main",
                "message": "Draft section 3", "watch": true }
// → result includes watched: true
```

## 🔗 Related Concepts

- [Session tools](/concepts/session-tool): `sessions_send`, `session_status`, `sessions_list`
- [Sub-agents](/tools/subagents): spawn edges and completion announcements (Lesson 7)
- [Heartbeat](/gateway/heartbeat): how queued notices wake main sessions (next in queue)
- [Session management](/concepts/session): keys, scopes and lifecycle (Lesson 2)

## 🤔 Quick Check

A child sub-agent finishes its run with `run_completed`, and two minutes later you edit its goal. **How many stale-state notices does the parent get, and why?**

<details><summary>Answer</summary>
One. `run_completed` is log-only, because completions arrive through sub-agent announcements. `goal_changed` is the only event that notifies. Even if you changed the goal five times before the parent's next turn, it would still see a single coalesced notice.
</details>

---
*Lesson 13 of Phase 1 (Core Concepts) | Streak: 4 days*
