# Master Strategy: OpenClaw Learning System

## Objective

Create an automated, sustainable learning system that teaches OpenClaw concepts through weekly research and daily curated lessons.

## Approach

### Weekly Research Cycle (Mondays 09:30 Europe/Rome)

**Agent:** zuko-researcher

**Responsibilities:**
1. Check current OpenClaw version against tracked version
2. If version changed: identify new features, breaking changes, updated docs
3. Explore documentation structure systematically
4. Add 3 topics (queue ≥5) or 7 (queue <5); one topic = one lesson
5. Append topics to queue (topic, source, difficulty, section, priority, addedAt)
6. Update progress.json state
7. Commit research summary

**Workflow:**
```
1. Read state/progress.json
2. Check openclaw --version
3. Compare versions, detect changes
4. Explore docs/ directory structure
5. Identify priority topics based on:
   - Version changes (highest priority)
   - Learning phase curriculum
   - User interest signals (if any)
6. Update queue in progress.json
7. Write docs/planning/XX_research_summary.md
8. Commit changes
```

### Daily Lesson Delivery (Mon-Fri 10:00 Europe/Rome)

**Agent:** sokka-orchestrator (main session)

**Responsibilities:**
1. Read next topic from queue in state/progress.json
2. Read source documentation
3. Format as engaging lesson (5-10 min read)
4. Include:
   - Concept overview
   - Key takeaways
   - Practical examples
   - Related concepts
   - Quiz/reflection question (optional)
5. Deliver to main chat
6. Mark topic as completed
7. Update streak counter

**Lesson Format:** defined in the daily job payload (sections: What You'll Learn, The Concept, Key Takeaways, Practical Example, Related Concepts, Quick Check with answer in `<details>`, derived footer). The payload is authoritative.

## Learning Phases (curriculum, OpenClaw 2026.9.3)

Each item below is exactly ONE daily lesson. **A phase is complete only when every item listed for it has a delivered lesson in that phase's `completedTopics`.** Topics outside this list do not count. Phase keys are the only valid values for a queue entry's `section`. Paths are relative to the OpenClaw docs root. Items marked (done)/(queued) reflect state on 2026-10-06 and are informational only; `state/progress.json` is authoritative.

### `phase_1_core` — Core Concepts
1. Agent runtime & workspace - concepts/agent.md, concepts/agent-workspace.md
2. Session management & routing - concepts/session.md
3. Memory system overview - concepts/memory.md
4. Tools overview & categories - tools/index.md
5. Exec tool & shell execution - tools/exec.md
6. Thinking levels & directives - tools/thinking.md
7. Model providers & selection - concepts/model-providers.md, concepts/models.md
8. Session state awareness - concepts/session-state.md
9. Context: what the model sees - concepts/context.md
10. System prompt assembly - concepts/system-prompt.md
11. Agent loop lifecycle - concepts/agent-loop.md
12. Compaction & session pruning - concepts/compaction.md, concepts/session-pruning.md
13. Memory search & hybrid retrieval - concepts/memory-search.md, concepts/memory-builtin.md

### `phase_2_gateway` — Gateway, Configuration & Safety
1. Automations & scheduling - automation/index.md, automation/cron-jobs.md
2. Secrets tool - tools/secrets.md
3. Model failover & resilience - concepts/model-failover.md
4. Heartbeat automation - gateway/heartbeat.md
5. Exec approvals & allowlists - tools/exec-approvals.md
6. Gateway architecture & runbook - concepts/architecture.md, gateway/index.md
7. Configuration basics - gateway/configuration.md, gateway/configuration-reference.md
8. SecretRef & the shared secret store - gateway/secrets.md
9. Sandboxing vs tool policy vs elevated - gateway/sandboxing.md, gateway/sandbox-vs-tool-policy-vs-elevated.md
10. Authentication & OAuth - gateway/authentication.md, concepts/oauth.md
11. Doctor, health & troubleshooting - gateway/doctor.md, gateway/health.md, gateway/troubleshooting.md
12. Background tasks & run history - automation/tasks.md, automation/cron-jobs/managing-jobs.md

### `phase_3_advanced` — Multi-agent, Channels & Extensibility
1. Sub-agents & delegation - tools/subagents.md
2. Skills system - tools/skills.md, tools/creating-skills.md, tools/skills-config.md
3. Multi-agent routing & bindings - concepts/multi-agent.md, concepts/agent-bindings.md
4. Session tools & sub-agent yield handoff - concepts/session-tool.md, concepts/subagent-yield-handoff.md
5. Channels overview & routing - channels/index.md, channels/channel-routing.md, channels/telegram.md
6. Control UI & dashboards - web/control-ui.md, web/dashboards.md, concepts/session-attachment.md
7. Plugins: install & manage - tools/plugin.md, plugins/architecture.md
8. MCP servers - tools/mcp.md
9. Skill Workshop & self-learning - tools/skill-workshop.md, tools/self-learning.md
10. Hooks & standing orders - automation/hooks.md, automation/standing-orders.md, concepts/standing-intents.md
11. Task Flow - automation/taskflow.md, tools/lobster.md
12. Browser control - tools/browser.md
13. Nodes & media understanding - nodes/index.md, nodes/media-understanding.md, tools/media-overview.md

### `phase_4_deep` — Deep Dives
1. Gateway WebSocket protocol & TypeBox - gateway/protocol.md, concepts/typebox.md
2. Memory architecture: tiers, dreaming, provenance - concepts/memory-architecture.md, concepts/dreaming.md, concepts/memory-provenance.md, concepts/active-memory.md
3. Context engine - concepts/context-engine.md
4. Message queue & run steering - concepts/queue.md, concepts/queue-steering.md, tools/steer.md
5. Prompt caching & token use - reference/prompt-caching.md, reference/token-use.md, concepts/usage-tracking.md
6. Observability: audit, logging, OpenTelemetry - gateway/audit.md, gateway/logging.md, gateway/opentelemetry.md
7. Security: threat model & incident response - security/THREAT-MODEL-ATLAS.md, security/incident-response.md, gateway/operator-scopes.md
8. ACP agents & external harnesses - tools/acp-agents.md, plugins/codex-harness.md
9. Plugin SDK - plugins/sdk-overview.md, plugins/building-plugins.md
10. Remote access & multiple gateways - gateway/remote.md, gateway/tailscale.md, gateway/multiple-gateways.md
11. Cloud workers & sessions - gateway/cloud-workers.md, gateway/cloud-sessions.md
12. Code Mode & swarm - tools/code-mode.md, tools/swarm.md

## Phase Rules

- Delivery order: lowest `priority` number first, oldest `addedAt` breaks ties; phases interleave.
- Curriculum gaps get priority 1 or 2. `currentPhase` = earliest phase whose curriculum is not fully covered.
- Source docs are local paths; check they exist and match the topic at research and at delivery.
- Version tracking: `openclaw --version` vs `state/progress.json` `currentVersion`; history in `state/version_history.json`.

## Success Metrics

- Daily lessons Mon-Fri; every curriculum item eventually delivered; system adapts to version changes.

## Git Workflow

- Two explicitly-staged commits per week by the research job: research output; `lessons/` archive. Never `git add -A`.
- Version changes: tag commit with version.
