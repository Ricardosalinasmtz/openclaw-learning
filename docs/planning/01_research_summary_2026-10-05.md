# Weekly Research Summary — 2026-10-05

## Version Status
- `openclaw --version`: **2026.9.3 (1391f7c)** — unchanged since 2026-09-10. No version_history update needed.
- Learning phase: **1 — Core Concepts** (12 topics completed, 2 carried over in queue).

## Carried Over
- Session State Awareness (`concepts/session-state.md`, P3)
- Heartbeat Automation (`gateway/heartbeat.md`, P2)

## Topics Added (6)

### 1. Context: What the Model Sees
- **Source:** `concepts/context.md` · beginner · Core Concepts · 1 lesson · P1
- **Rationale:** Foundational gap — how context is built and inspected (`/context`, `/status`). Prerequisite for compaction, pruning and system prompt topics.

### 2. System Prompt Assembly
- **Source:** `concepts/system-prompt.md` · intermediate · Core Concepts · 1 lesson · P2
- **Rationale:** How workspace bootstrap files, tools list and skills are injected (incl. truncation behaviour). Directly relevant to tuning the orchestrator/sub-agent AGENTS.md files.

### 3. Agent Loop Lifecycle
- **Source:** `concepts/agent-loop.md` · intermediate · Core Concepts · 2 lessons · P2
- **Rationale:** Lifecycle events, streams and wait semantics — ties together sessions, sub-agents and failover already covered.

### 4. Compaction & Session Pruning
- **Source:** `concepts/compaction.md`, `concepts/session-pruning.md` · intermediate · Core Concepts · 2 lessons · P2
- **Rationale:** Long-running sessions (daily lesson jobs, orchestrator) hit context limits; covers auto-compaction, `/compact`, tool-result trimming and prompt-cache effects.

### 5. Memory Search & Hybrid Retrieval
- **Source:** `concepts/memory-search.md` · intermediate · Memory · 1 lesson · P3
- **Rationale:** Deepens the completed Memory Overview: embeddings, hybrid retrieval, provider choice.

### 6. Exec Approvals & Allowlists
- **Source:** `tools/exec-approvals.md` · advanced · Tools · 2 lessons · P3
- **Rationale:** Follow-up to Exec Tool lesson; policy knobs, allowlists, strict/YOLO workflows and MCP grants — key for safe unattended jobs.

## Selection Rationale
- Version unchanged → continued Phase 1 curriculum.
- Prioritised the "context pipeline" cluster (context → system prompt → agent loop → compaction/pruning), the biggest remaining Phase 1 gap.
- Deferred candidates for later weeks: `concepts/active-memory.md`, `concepts/queue.md` / `queue-steering.md`, `concepts/standing-intents.md`, `concepts/multi-agent.md` (Phase 3).
- With 8 items queued (~13 lessons), Phase 1 should wrap up in ~2 weeks; next research run should begin Phase 2 (gateway/, configuration).
