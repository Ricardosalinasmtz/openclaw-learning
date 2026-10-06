# 🎓 OpenClaw Lesson: Agent Runtime & Workspace
**Date:** 2026-09-11 | **Day:** 1 of learning streak
**Source:** concepts/agent.md, concepts/agent-workspace.md | **Difficulty:** beginner

## 🎯 What You'll Learn

You'll understand how OpenClaw agents run, where they "live" (their workspace), and how sessions bootstrap with persona files. This is foundational knowledge for everything else in OpenClaw.

## 📖 The Concept

OpenClaw ships with one **embedded agent runtime** — a built-in agent loop that handles tool wiring, prompt assembly, and session management. Each configured agent has its own **workspace**, which is the agent's home directory and working context for all file operations.

The workspace is where your agent's "memory" lives. It contains bootstrap files like `AGENTS.md` (operating instructions), `SOUL.md` (persona and tone), `USER.md` (your preferences), and `MEMORY.md` (curated long-term knowledge). These files are injected into every session's system prompt, making them the agent's persistent identity.

Think of the workspace as the agent's brain and `~/.openclaw/` as the nervous system — the workspace holds knowledge and personality, while the state directory stores config, credentials, and session history. They're separate by design: you can back up or migrate the workspace independently without touching runtime state.

## 🔑 Key Takeaways

- **Workspace = Agent's home**: The working directory for all file tools and context injection (default: `~/.openclaw/workspace`)
- **Bootstrap files define identity**: `AGENTS.md`, `SOUL.md`, `USER.md`, and `MEMORY.md` are injected into every session
- **Workspace is NOT a sandbox**: Tools can reach outside the workspace unless sandboxing is explicitly enabled
- **Sessions live in SQLite**: Active session history is stored in `~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite`, not in the workspace
- **Backup your workspace**: Treat it as private memory — use a private git repo to version and protect it

## 💡 Practical Example

**Scenario:** You want to back up your agent's workspace before making major changes.

```bash
# Navigate to your workspace
cd ~/.openclaw/workspace

# Check what's there
ls -la

# Initialize git if not already done
git init
git add AGENTS.md SOUL.md USER.md MEMORY.md memory/
git commit -m "Backup before major AGENTS.md refactor"

# Push to a private remote
git remote add origin https://github.com/youruser/openclaw-workspace.git
git push -u origin main
```

**Later, to restore on a new machine:**
```bash
# Clone to the default location
git clone https://github.com/youruser/openclaw-workspace.git ~/.openclaw/workspace

# Update config to point to this workspace
# Edit ~/.openclaw/openclaw.json:
# { "agents": { "defaults": { "workspace": "~/.openclaw/workspace" } } }

# Verify before starting Gateway
openclaw agents list
```

## 🔗 Related Concepts

- [Session Management](/concepts/session) - How sessions are stored and routed
- [Multi-agent Routing](/concepts/multi-agent) - Running multiple agents with different workspaces
- [Sandboxing](/gateway/sandboxing) - Isolating agent workspace access
- [Memory System](/concepts/memory) - How MEMORY.md and daily logs work

## 🤔 Quick Check

**Question:** If you delete `MEMORY.md` from your workspace, what happens on the next session start?

<details>
<summary>Click to reveal answer</summary>

**Answer:** The session starts normally — `MEMORY.md` is optional. OpenClaw only injects it if it exists. However, you'll lose access to your curated long-term memory until you restore it from backup. Daily logs in `memory/YYYY-MM-DD.md` are also unaffected.
</details>

---
*Lesson 1 of Phase 1 (Core Concepts) | Streak: 1 day*
