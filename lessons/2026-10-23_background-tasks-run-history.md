# 🎓 OpenClaw Lesson: Background Tasks & Run History
**Date:** 2026-10-23 | **Day:** 18 of learning streak
**Source:** automation/tasks.md, automation/cron-jobs/managing-jobs.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

OpenClaw keeps a background task ledger that records every piece of work running outside your main conversation — subagent spawns, automation job runs, media generation, and CLI agent operations. Today you'll learn what actually creates a task, how tasks flow through their lifecycle, how completion notifications reach you, and how run history separates a healthy execution from a healthy delivery.

## 📖 The Concept

Tasks are records, not schedulers. Automations and the heartbeat decide *when* work runs; tasks simply track *what happened* with that work — when it started, how it ended, and whether the result made it back to you. Every ACP run, subagent spawn, automation job run (both main-session and isolated), CLI agent operation, and session-backed media generation run creates a task record. Heartbeat turns, normal interactive chat, and direct `/command` responses do not — so a quiet task ledger on a normal chat day is expected.

Each task moves through a fixed lifecycle: `queued → running → terminal`, where terminal means `succeeded`, `failed`, `timed_out`, `cancelled`, or `lost`. Transitions happen automatically from agent run lifecycle events — you never manage them by hand. A task whose backing runtime state disappears (for example after a gateway crash) is marked `lost` after a grace period. Terminal records are retained for 7 days (24 hours for `lost` records) and then pruned automatically by a sweeper that runs every 60 seconds.

The subtlety worth internalizing is that **execution and delivery are separate tracks**. A subagent or automation can finish its work perfectly (`succeeded`) while the hand-back of the result fails. In that case the task shows a terminal outcome of `blocked` — the work is done, but you never received it. Completion itself is push-driven: detached work either notifies a channel directly or queues a system event in the requester session (with an immediate heartbeat wake), so polling task state in a loop is usually the wrong shape. Poll only when you need to debug, intervene, or audit.

Run history adds one more axis for automation jobs. Each run records `status` for payload execution (`ok`, `error`, or `skipped`) and `completionStatus` for the whole run including delivery (`succeeded`, `failed`, or `unknown`). A delivery-only failure leaves execution at `ok` with `completionStatus: "failed"` — it does not increment execution error counters or trigger retry backoff. Intentional silence (a `NO_REPLY` answer, for instance) is recorded as a suppression reason, not a failure.

## 🔑 Key Takeaways

- Tasks are an activity ledger: automations and heartbeat decide when work runs; tasks record what happened to it.
- Subagents, ACP runs, every automation run, CLI agent operations, and media generation create tasks; heartbeat turns and normal chat do not.
- Lifecycle is `queued → running → terminal` (`succeeded`, `failed`, `timed_out`, `cancelled`, `lost`); terminal records are kept 7 days, then pruned automatically.
- A task can be `succeeded` while its delivery is `blocked` — execution and delivery are tracked separately, and `openclaw tasks list --status blocked` is how you find them.
- Run history separates `status` (execution: ok/error/skipped) from `completionStatus` (whole run incl. delivery: succeeded/failed/unknown), so a failed delivery never masquerades as a broken job.

## 💡 Practical Example

A realistic morning-after workflow: an isolated automation job generated an overnight report, the Telegram group never saw it, and you want to figure out what happened.

```bash
# 1. See everything running or recently finished, newest first
openclaw tasks list

# 2. Find tasks that finished but whose delivery is stuck
openclaw tasks list --status blocked

# 3. Inspect the full record: timing, delivery state, error, terminal summary
openclaw tasks show <lookup>

# 4. Recover the missing delivery (new fenced generation), or
#    record that you intentionally don't want it
openclaw tasks retry <lookup>
openclaw tasks dismiss <lookup>

# 5. Cross-check the automation job's own run history
openclaw automations runs <jobId> --limit 50
```

In the `runs` output, `status: "ok"` with `completionStatus: "failed"` is the signature of a delivery-only failure — the job executed fine, the hand-back didn't land. If instead `status` were `error`, you'd be debugging the run itself (and recurring jobs would have entered backoff: 30s, 60s, 5m, 15m, 60m, resetting after a successful run).

## 🔗 Related Concepts

- Automations & Scheduling (Phase 2, 2026-09-17)
- Sub-agents & Delegation (Phase 3, 2026-09-18)
- Heartbeat Automation (Phase 2, 2026-10-07)
- Doctor, Health & Troubleshooting (Phase 2, 2026-10-22)

## 🤔 Quick Check

You launch an isolated automation job that analyzes a log file and posts the summary to a Telegram channel. The next morning the group chat never received the summary, but `openclaw tasks list` shows the job's task as `succeeded`. What does that combination of signals tell you about where the failure happened, and which commands would you use to inspect and recover the missing result?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** It tells you the execution succeeded but the delivery failed — the docs track these as separate tracks, so a finished task whose result couldn't be handed back shows a `blocked` terminal outcome even though the underlying run is `succeeded`. You would confirm with `openclaw tasks show <lookup>` (and `openclaw tasks list --status blocked` to locate it), then recover with `openclaw tasks retry <lookup>` to start a new fenced delivery generation, or `openclaw tasks dismiss <lookup>` if the result is no longer wanted. The canonical result is retained for 7 days, so nothing is lost yet.

</details>

---
*Lesson 27 of Phase 2 (Gateway, Configuration & Safety) | Streak: 18 days*
