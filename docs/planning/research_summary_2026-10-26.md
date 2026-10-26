# Weekly Research Summary — 2026-10-26

## Version Status

- `openclaw --version` → **OpenClaw 2026.9.3 (1391f7c)**
- Tracked baseline in `state/progress.json` → **2026.9.3 (1391f7c)**
- **Result: unchanged.** `currentVersion.checkedAt` updated to `2026-10-26T09:34:50+01:00`. No other state changes from version tracking; `state/version_history.json` untouched (no new entry).
- No docs moved or disappeared relative to the curriculum baseline (2026.9.3). All curriculum source paths for this week's topics verified present under `/home/ricardo/.npm-global/lib/node_modules/openclaw/docs/`.

## State at Start

- `currentPhase`: `phase_3_advanced` (phase_1_core 13/13 and phase_2_gateway 12/12 complete — all 25 curriculum items delivered)
- Queue before this run: **3 items** (<5 → add 7 this week, per queue rule)
- Note: last week's summary reported the queue as 8; five phase_2 lessons since then (10-19 → 10-23) were delivered and removed, leaving the 3 phase_3 items added on 2026-10-19. Consistent.

## Gaps Found per Phase

### phase_1_core — COMPLETE
All 13 curriculum items have delivered lessons. No gaps.

### phase_2_gateway — COMPLETE
All 12 curriculum items have delivered lessons (last five: SecretRef, Sandboxing, Auth & OAuth, Doctor, Background Tasks — all delivered 2026-10-19 → 2026-10-23). No gaps.

### phase_3_advanced (current phase) — 11 gaps at start (2 of 13 items done)
Delivered: Sub-agents & Delegation (2026-09-18), Skills System (2026-09-22).
Already queued at start: Multi-agent routing & bindings, Session tools & yield handoff, Channels overview & routing.
Unqueued gaps, in curriculum order:
1. Control UI & dashboards (item 6)
2. Plugins: install & manage (item 7)
3. MCP servers (item 8)
4. Skill Workshop & self-learning (item 9)
5. Hooks & standing orders (item 10)
6. Task Flow (item 11)
7. Browser control (item 12)
8. Nodes & media understanding (item 13)

### phase_4_deep — LOCKED
12 items, all pending; not eligible until every phase_3 item is delivered.

## Topics Added (7)

Queue held 3 items before this run (<5), so exactly 7 topics added. All are uncovered `phase_3_advanced` curriculum items (the only current-phase gaps) → priority 2.

| # | Topic | Source (verified to exist) | Section | Priority | Difficulty |
|---|-------|---------------------------|---------|----------|------------|
| 1 | Control UI & Dashboards | `web/control-ui.md`, `web/dashboards.md`, `concepts/session-attachment.md` | phase_3_advanced | 2 | intermediate |
| 2 | Plugins: Install & Manage | `tools/plugin.md`, `plugins/architecture.md` | phase_3_advanced | 2 | intermediate |
| 3 | MCP Servers | `tools/mcp.md` | phase_3_advanced | 2 | intermediate |
| 4 | Skill Workshop & Self-learning | `tools/skill-workshop.md`, `tools/self-learning.md` | phase_3_advanced | 2 | intermediate |
| 5 | Hooks & Standing Orders | `automation/hooks.md`, `automation/standing-orders.md`, `concepts/standing-intents.md` | phase_3_advanced | 2 | advanced |
| 6 | Task Flow | `automation/taskflow.md`, `tools/lobster.md` | phase_3_advanced | 2 | intermediate |
| 7 | Browser Control | `tools/browser.md` | phase_3_advanced | 2 | intermediate |

**Rationale:**
- **Control UI & Dashboards** — curriculum item 6 of phase 3. The browser-side operator surface: chat/activity/config panels, session dashboards and widgets, and how Gateway-owned sessions attach across clients.
- **Plugins: Install & Manage** — item 7. Extensibility half of the phase: install/configure plugins, discovery and load rules, plus the capability model that defines what plugins own.
- **MCP Servers** — item 8. Connecting external Model Context Protocol servers via Control UI, CLI, or config — how agents gain third-party tool surfaces.
- **Skill Workshop & Self-learning** — item 9. The self-authoring loop: generating and reviewing skill drafts from chat, and the propose/auto modes that turn completed work into reusable skills.
- **Hooks & Standing Orders** — item 10. Event-driven automation (lifecycle/command hooks) and permanent operating authority for autonomous programs, including event-conditioned standing intents. Marked advanced: two distinct mechanisms plus safety boundaries.
- **Task Flow** — item 11. The durable orchestration layer above background tasks, with Lobster's typed workflow runtime and resumable approval gates.
- **Browser Control** — item 12. The integrated browser control service: agent-driven navigation/actions, settings and lifecycle, and debugging common interference with the user's own browser.

Each is exactly one daily lesson; none overlaps an existing queue entry or any phase's `completedTopics`.

After this run, the only uncovered phase_3 curriculum item is **Nodes & media understanding** (item 13) — it will be the natural priority-1 candidate next week.

## Queue Length

- Before: **3**
- Added: **7**
- After: **10** (verified: existing 3 entries preserved in order with original `addedAt` `2026-10-19T09:30:00+02:00`; file re-read after write, JSON parses, length check passed; no duplicates within queue or vs any `completedTopics`)

## Next Week's Candidates (not added)

- Nodes & media understanding (phase 3, item 13) — the last phase_3 gap; priority 1 (only 3 may be added next week if the queue stays ≥5, so phase_4 waits)
- Once every phase_3 item is delivered, `currentPhase` unlocks `phase_4_deep`; first candidates: Gateway WebSocket protocol & TypeBox, Memory architecture (tiers, dreaming, provenance), Context engine.
