# 🎓 OpenClaw Lesson: Automations & Scheduling Overview
**Date:** 2026-09-17 | **Day:** 5 of learning streak
**Source:** `automation/index.md`, `automation/cron-jobs/schedules.md`, `automation/cron-jobs/payloads.md`, `automation/cron-jobs/delivery.md` | **Difficulty:** Intermediate

## 🎯 What You'll Learn

OpenClaw's automation system is its built-in scheduler — the engine behind reminders, recurring reports, triggered jobs, and background workflows. This lesson covers the five schedule kinds, four payload types, and three delivery modes that make up the complete automation stack.

## 📖 The Concept

### Why Automations Matter

Without automations, your agent can only respond when you message it. Automations change that by letting you schedule work in advance: a daily briefing at 9 AM, a weekly code review, a one-shot reminder in 20 minutes, or a job that fires when an external event occurs (like a CI build failure). The scheduler persists jobs, wakes the agent at the right time, and delivers output to wherever you need it — a chat channel, a webhook, or silently into a session.

### The Automation Ecosystem

OpenClaw offers several automation mechanisms, each suited to a different need:

| Mechanism | Best For | Example |
|-----------|----------|---------|
| **Automations** | Scheduled or one-shot work | Daily report, 20-min reminder |
| **Heartbeat** | Ambient monitoring | Check inbox every 30 min |
| **Tasks** | Tracking detached work | Inspect a subagent's status |
| **Task Flow** | Multi-step orchestration | Research → summarize → notify |
| **Hooks** | Lifecycle event reactions | Run script on session reset |
| **Standing Orders** | Persistent instructions | "Always check compliance before replying" |

Think of it this way: use **Automations** for work with its own schedule or instructions. Use **Heartbeat** for ambient awareness. Use **Hooks** for event reactions. Use **Standing Orders** for rules that apply to every interaction.

### The Five Schedule Kinds

Every automation job needs a schedule — *when* it should fire:

| Kind | Description | Example |
|------|-------------|---------|
| `at` | One-shot at a specific time | Remind me tomorrow at 3 PM |
| `every` | Fixed interval | Check inbox every 30 minutes |
| `cron` | Cron expression (wall-clock aware) | Daily at 9 AM in my timezone |
| `on-exit` | Fire when a watched command exits | Notify after a deploy finishes |
| `stream` | Fire from a process's output lines | React to CI build output |

**Important:** Cron expressions without `--tz` use the Gateway host timezone. Always specify `--tz` for predictable behavior. When both day-of-month and day-of-week are non-wildcard, cron uses **OR logic** (matches either) — not AND. So `0 9 15 * 1` fires on the 15th AND on every Monday, roughly 5-6 times per month.

### Payload: What Runs When It Fires

A job carries exactly one payload kind — *what* happens when the schedule triggers:

| Payload | What It Does |
|---------|-------------|
| **System event** | Enqueues text into the main session (no model call) |
| **Agent message** | Spawns a model-backed agent turn |
| **Command** | Runs a shell script directly (no model call) |
| **Script** | Headless code-mode script using agent tools |

Agent-turn jobs support rich configuration: per-job model overrides, thinking level, tool allowlists, fallback models, and light-context mode. A key best practice: **pick the model for the job's difficulty**, not the agent's default. Routine checks run well on lighter models — cheaper, faster, and it adds up across a schedule.

### Execution Styles: Where It Runs

| Style | Session Value | Runs In |
|-------|---------------|---------|
| **Main session** | `main` | Owning agent's main session |
| **Isolated** | `isolated` | Dedicated `cron:<jobId>` session |
| **Current** | `current` | Detached run, commits back to creation conversation |
| **Custom** | `session:<id>` | Persistent named session |

Isolated jobs get a fresh transcript each run (safe preferences carried over, but no ambient context). Current sessions read a bounded conversation tail and commit results back. Custom sessions persist context across runs — ideal for workflows that build on previous summaries.

### Delivery: Where Output Goes

| Mode | What Happens |
|------|-------------|
| **Announce** | Fallback-delivers final text to a chat channel |
| **Webhook** | POSTs finished event to a URL |
| **None** | No runner fallback delivery (silent) |

Failure protection is built in: jobs auto-alert after 2 consecutive execution failures with a 1-hour cooldown. After 10 consecutive failures, the job auto-disables as a safety backstop. You can customize thresholds, channels, and cooldowns per job via `failureAlert`.

## 🔑 Key Takeaways

- **Five schedule kinds** — `at`, `every`, `cron`, `on-exit`, `stream` — cover one-shot, interval, wall-clock, process-exit, and stream-driven triggers
- **Four payload types** — system event, agent message, command, script — define what runs at fire time
- **Pick models by difficulty** — light models for routine checks, heavy models only when deep reasoning is needed
- **Cron uses OR logic** — non-wildcard day-of-month AND day-of-week match either, not both
- **Failure auto-protection** — 2-failure alerts, 10-failure auto-disable, customizable per job
- **Heartbeat vs Automations** — heartbeat is the ambient monitor; separate automation jobs own explicit schedules

## 💡 Practical Example

Here's a complete automation that checks a CI pipeline status and investigates when it changes:

```bash
# Condition watcher: fires only when PR CI status changes
openclaw automations add \
  --name "PR CI watcher" \
  --every 30s \
  --trigger-script ./watch-pr-ci.js \
  --message "Investigate the CI status change" \
  --session isolated \
  --announce --channel telegram --to "-1001234567890"
```

The trigger script returns JSON like:
```json
{ "fire": true, "message": "PR 123 CI: pending -> failed", "state": { "status": "failed" } }
```

When `fire: true`, the agent gets the message and investigates. When `fire: false`, state persists silently. The `once: true` flag disables the job after the first fire if you only need a single notification.

For a simpler recurring task — a daily summary at 9 AM CET:

```bash
openclaw automations add \
  --name "Daily project summary" \
  --cron "0 9 * * *" \
  --tz "Europe/Rome" \
  --message "Summarize today's project progress. Respond in English." \
  --session isolated \
  --announce \
  --model cineca/qwen3.6-35b-a3b
```

## 🔗 Related Concepts

- [Heartbeat](/gateway/heartbeat) — ambient monitoring every 30 minutes
- [Background Tasks](/automation/tasks) — task ledger for all detached work
- [Task Flow](/automation/taskflow) — multi-step flow orchestration
- [Hooks](/automation/hooks) — event-driven lifecycle scripts
- [Standing Orders](/automation/standing-orders) — persistent agent instructions
- [Plugin hooks](/plugins/hooks) — in-process tool and message interception

## 🤔 Quick Check

1. You want a job that runs every Monday and Friday at 8 AM. Why would `0 8 * * 1,5` NOT work as expected? How would you fix it?
2. A recurring job uses a lighter model for efficiency. Which configuration fields let you override the model and thinking level per job?
3. What's the difference between an `isolated` and a `current` session execution style?

---
*Lesson 6 of Phase 2 (Gateway, Configuration & Safety) | Streak: 5 days*
