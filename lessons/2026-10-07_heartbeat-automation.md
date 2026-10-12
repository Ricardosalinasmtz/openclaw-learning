# 🎓 OpenClaw Lesson: Heartbeat Automation
**Date:** 2026-10-07 | **Day:** 6 of learning streak
**Source:** gateway/heartbeat.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Today we cover the heartbeat — OpenClaw's system-owned automation that runs periodic agent turns in the main session so the model can surface anything needing attention without spamming you. You'll learn how heartbeat cadence is actually scheduled, where alerts are delivered, how the agent signals "nothing to see" versus "look at this," and when to reach for heartbeat versus a regular automation job.

## 📖 The Concept

**What a heartbeat is.** Heartbeat is a scheduled main-session turn, not a background task. On each tick, the gateway sends the heartbeat prompt (verbatim as a user message) into the agent's main session and lets the model decide whether anything needs attention. Because it runs in the main session, the model sees your recent conversation context — but it does not create background-task records; those belong to detached work like subagents and isolated jobs.

**How scheduling works under the hood.** The Automations scheduler owns the actual cadence: the gateway maintains one system-owned automation job per heartbeat-enabled agent, visible in `openclaw cron list --all` as `Heartbeat (agent-id)`. The `agents.*.heartbeat` config is the desired-state input; the persisted monitor schedule owns the real tick. So you always edit `agents.*.heartbeat`, never the automation job. Scheduled heartbeats require automations to be enabled — if `cron.enabled` is false, scheduled heartbeats don't run, though manual and event-driven wakes still do. Setting `every: "0m"` disables only the recurring cadence; targeted event wakes can still trigger one-off turns.

**The prompt and the response contract.** The default prompt is deliberately narrow: follow the monitor scratch checklist if present, keep recurring work in automation jobs (never infer old tasks from prior chats), and reply `NO_REPLY` when nothing needs attention. This is what keeps a default install quiet. A heartbeat can instead call `heartbeat_respond` with `notify: true` and alert text for a visible alert, or `notify: false` for a silent-but-remembered update. Legacy `HEARTBEAT_OK` at the start or end of a reply is still accepted and dropped when the remaining content is trivial.

**Delivery, visibility, and cost.** Alerts go to the operator's DM by default (`target: "owner"`, resolved from `commands.ownerAllowFrom`), never to a group; `target: "last"` follows the most recent conversation, and `target: "none"` runs the turn without external delivery. Per-channel `heartbeatVisibility` flags (`showOk`, `showAlerts`, `useIndicator`) control what's visible, and if all three are off the run is skipped entirely. The monitor scratch — a tiny checklist stored with the heartbeat job — is appended to the prompt each tick; if it's effectively empty, the run is skipped to save API calls. To keep token cost down, use `isolatedSession: true` (fresh session, no conversation history), `lightContext: true` (skip workspace bootstrap files), or a cheaper heartbeat model.

**Heartbeat vs automations.** The rule of thumb: heartbeat is ambient attention on a fixed cadence (scan inboxes, notice a blocked task, check in if daytime). Recurring work with its own cadence, run history, and enable/disable state belongs in automation jobs. Heartbeat can *react* to completed background tasks, but you should not encode schedules into heartbeat scratch — create jobs instead.

## 🔑 Key Takeaways

- Heartbeat = periodic agent turns in the main session, scheduled by the Automations system as a hidden `Heartbeat (agent-id)` job; edit `agents.*.heartbeat`, not the job.
- The response contract is quiet-by-default: reply `NO_REPLY` when nothing needs attention; alerts come from non-OK replies or `heartbeat_respond` with `notify: true`.
- `target: "owner"` (default) delivers only to a resolvable operator DM — never groups; use `target: "last"` to follow the recent conversation or `"none"` for internal-only runs.
- The monitor scratch is a tiny checklist appended to the prompt; keep it small (or empty-skips save API calls), and use automation jobs — not scratch — for recurring schedules.
- Cost controls: `isolatedSession: true` + `lightContext: true` + a cheaper `model`; `activeHours` restricts ticks to a time window (equal start/end means a zero-width window that skips everything).

## 💡 Practical Example

A quiet-but-useful ops setup: 30-minute cadence, daytime only, silent except for real alerts, and cheap to run:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "owner",              // operator DM only, never groups
        isolatedSession: true,        // fresh session: ~2-5K tokens instead of full history
        lightContext: true,           // skip workspace bootstrap files
        activeHours: {
          start: "08:00",
          end: "22:00",
          timezone: "Europe/Rome"
        }
      }
    }
  }
}
```

Then give the heartbeat something small to check every tick via the monitor scratch (job id from `openclaw cron list --all`):

```bash
openclaw cron scratch <jobId> --set "
# Heartbeat checklist
- Quick scan: anything urgent in inboxes?
- If a task is blocked, write down what is missing and ask next time.
- Otherwise reply NO_REPLY.
"
```

Ticks outside 08:00–22:00 Rome time are skipped until the next tick inside the window. For the guaranteed "health check every morning at 09:00 sharp" style of work, add an automation job instead — heartbeat stays for ambient attention.

## 🔗 Related Concepts

- [Automation](/automation) — all automation mechanisms at a glance
- [Background Tasks](/automation/tasks) — how detached work is tracked
- [Timezone](/concepts/timezone) — how timezone affects heartbeat scheduling
- [Troubleshooting](/automation/cron-jobs#troubleshooting) — debugging automation issues

## 🤔 Quick Check

You run an ops agent whose heartbeats every 30 minutes are pinging the team at 3 AM, and your colleague argues you should just replace heartbeats with a single 09:00 automation job to "do a morning sweep." Using `activeHours`, `target`, and the heartbeat/automation split, how would you configure the heartbeat to stop the night pings — and what does the doc say happens to ticks that fall outside the window, and why might that matter for the colleague's proposed fix?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Set `activeHours: { start: "08:00", end: "22:00", timezone: "<your tz>" }` (with `target: "owner"` so alerts only reach your DM) — outside that window ticks are skipped until the next tick inside the window. That matters for the colleague's fix because skipped ticks don't accumulate or fire up at 08:00, so anything that went wrong overnight would not surface in a heartbeat until the next daytime tick. If the morning sweep needs a guaranteed check, keep it as its own automation job at 09:00 — heartbeat is for ambient daytime attention, automation jobs are for scheduled work with a real run history.

</details>

---
*Lesson 15 of Phase 2 (Gateway, Configuration & Safety) | Streak: 6 days*
