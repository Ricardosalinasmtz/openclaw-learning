# Weekly Research Summary — 2026-09-21

## Version Status

| Field | Value |
|-------|-------|
| Current version | 2026.9.3 (1391f7c) |
| Version changed? | **No** — same as state file (last checked 2026-09-10) |
| New docs to prioritize | None |

## Topics Added to Queue (7)

All topics target **Phase 1 (Core Concepts)** and **Phase 2 (Gateway)** — the next phases after the 7 completed topics (Agent Runtime, Session Management, Memory, Tools Overview, Exec, Automations, Sub-agents).

### 1. Secrets Tool — Credential Management
- **Source:** `tools/secrets.md`
- **Difficulty:** Intermediate
- **Section:** Tools
- **Estimated lessons:** 1
- **Priority:** 1 (High)
- **Rationale:** Core security mechanism — the agent must understand how to request and use credentials without ever seeing them. Critical for any agent that interacts with external services.

### 2. Skills System — Tool Instruction Files
- **Source:** `tools/skills.md` (+ `tools/creating-skills.md`, `tools/skills-config.md`, `tools/skill-workshop.md`)
- **Difficulty:** Intermediate
- **Section:** Tools
- **Estimated lessons:** 2
- **Priority:** 1 (High)
- **Rationale:** Skills are how agents learn custom tool usage. Loading order, precedence, and gating are essential knowledge. Includes the skill lifecycle (create → propose → apply).

### 3. Thinking Levels & Directives
- **Source:** `tools/thinking.md`
- **Difficulty:** Beginner
- **Section:** Tools
- **Estimated lessons:** 1
- **Priority:** 2
- **Rationale:** The `/think` directive system controls reasoning effort across providers. Understanding resolution order (inline → session → agent default → global) is foundational for model interaction.

### 4. Model Providers & Selection
- **Source:** `concepts/model-providers.md` (+ `concepts/models.md`)
- **Difficulty:** Intermediate
- **Section:** Core Concepts
- **Estimated lessons:** 2
- **Priority:** 2
- **Rationale:** How providers are configured, how model refs work (`provider/model`), the OpenAI runtime split (OpenClaw vs Codex), and provider-specific behaviors. Foundational for understanding agent routing.

### 5. Model Failover & Resilience
- **Source:** `concepts/model-failover.md`
- **Difficulty:** Advanced
- **Section:** Core Concepts
- **Estimated lessons:** 1
- **Priority:** 2
- **Rationale:** Auth profile rotation, model fallback chains, retry logic for transient failures, and user override strictness. Important for understanding agent reliability guarantees.

### 6. Session State Awareness
- **Source:** `concepts/session-state.md`
- **Difficulty:** Advanced
- **Section:** Core Concepts
- **Priority:** 3
- **Estimated lessons:** 2
- **Rationale:** Signal log, watchers, and reconciliation for multi-agent coordination. Explains how parent agents detect child session changes and how humans can intervene without breaking agents.

### 7. Heartbeat Automation
- **Source:** `gateway/heartbeat.md`
- **Difficulty:** Intermediate
- **Section:** Gateway
- **Priority:** 2
- **Estimated lessons:** 2
- **Rationale:** Heartbeat is a system-owned automation for periodic agent turns. Covers cadence, event-driven wakes, flood guards, and the distinction from independent automation jobs. Complements the already-completed Automations topic.

## Selection Criteria

- **Coverage gaps:** These 7 topics fill the biggest holes in Phase 1 and Phase 2. Specifically: security (secrets), customization (skills), model control (thinking, providers, failover), multi-agent coordination (session state), and continuous operation (heartbeat).
- **Progression:** Beginner topics (thinking) are placed after intermediate ones in priority but will be taught first in the curriculum due to lower barrier to entry.
- **Cross-references:** Several topics overlap (e.g., model providers + failover + thinking all relate to model selection). Lessons will reference each other to reinforce understanding.
