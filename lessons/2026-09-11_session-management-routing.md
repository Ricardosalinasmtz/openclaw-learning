# 🎓 OpenClaw Lesson: Session Management & Routing

**Date:** 2026-09-11 | **Day:** 1 of learning streak
**Source:** concepts/session.md | **Difficulty:** beginner

## 🎯 What You'll Learn

You'll understand how OpenClaw organizes conversations into sessions, how messages are routed based on their source (DMs, groups, cron jobs), and how to configure isolation for multi-user setups. This is foundational to controlling where your agent "remembers" conversations.

## 📖 The Concept

A **session** in OpenClaw is a conversation container owned by the gateway. Every inbound message gets routed to a session based on where it came from. By default, all direct messages share one session for continuity, while group chats and rooms get isolated sessions per channel.

The routing behavior is configurable through `session.dmScope` and `session.groupScope`. For single-user personal agents, the default "all DMs share one session" works great. But if multiple people can message your agent, you **must** enable DM isolation—otherwise Alice's private messages would be visible to Bob.

Sessions persist until you reset them manually (with `/new` or `/reset`) or configure automatic resets (daily at a specific hour, or after idle time). The gateway stores session state in SQLite, with optional transcript archives on disk.

## 🔑 Key Takeaways

- **DMs share one session by default** (great for single-user, dangerous for multi-user)
- **Use `dmScope: "per-channel-peer"`** to isolate conversations by channel + sender
- **Group chats are isolated per-group by default** (can be merged into main session via binding)
- **Sessions don't auto-reset** unless you configure daily/idle policies or use `/new`
- **Cron jobs always get fresh sessions** per run (isolated from your main conversation)

## 💡 Practical Example

**Scenario:** You're setting up a personal agent that you and your roommate both use. You want to keep your conversations separate.

```json5
{
  session: {
    dmScope: "per-channel-peer", // Isolate by channel + sender
    identityLinks: [
      // Map your identities across channels to one canonical peer ID
      {
        peerId: "ricardo-canonical",
        identities: [
          { channel: "telegram", id: "123456" },
          { channel: "discord", id: "789012" }
        ]
      }
    ]
  }
}
```

**Verify your setup:**
```bash
openclaw security audit
```

**Check active sessions:**
```bash
openclaw sessions --json
# Or in chat: /status
```

## 🔗 Related Concepts

- [Main Session](/concepts/main-session) - The personal-agent default rolling conversation
- [Session Attachment](/concepts/session-attachment) - Continue Gateway sessions in Control UI
- [Session Search](/concepts/session-search) - Full-text recall across past transcripts
- [Compaction](/concepts/compaction) - Summarizing long conversations
- [Multi-Agent Routing](/concepts/multi-agent) - Session isolation across multiple agents

## 🤔 Quick Check

**Question:** You've configured `dmScope: "per-peer"` but notice that when the same person messages you from Telegram and Discord, they get two separate sessions. What configuration would fix this?

<details>
<summary>💡 Reveal Answer</summary>

Use `session.identityLinks` to map both of their identities (Telegram ID and Discord ID) to the same canonical peer ID. This tells OpenClaw "these are the same person" so their messages share one session across channels.
</details>

---
*Lesson 2 of Phase 1 (Core Concepts) | Streak: 1 day*
