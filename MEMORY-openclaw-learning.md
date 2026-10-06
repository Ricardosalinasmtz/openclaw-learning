# MEMORY-openclaw-learning.md

Project-specific memory for the OpenClaw Learning System.

<!-- project: github.com/Ricardosalinasmtz/sokka-orchestrator -->

## Decisions

- **Automation over external cron:** Uses OpenClaw's built-in automations for better integration and reliability
- **Weekly collection, daily delivery:** Research on Mondays 09:30, lessons Mon-Fri (not random/hourly)
- **Version-aware learning:** Track OpenClaw version changes and prioritize new features
- **Curriculum approach:** Structured phases instead of random topic selection
- **Direct chat delivery:** via announce, not webhooks (sessions: see below)

## Technical Learnings

- OpenClaw automations support cron schedules with timezone specification
- Automation payloads can delegate to sub-agents (zuko-researcher, katara-document-writer)
- State tracking via JSON files in project directory

## Dead Ends / Avoided Approaches

- ~~Hourly random topic selection~~ → Too chaotic, no curriculum structure
- ~~External cron + webhook delivery~~ → Unnecessary complexity when OpenClaw automations exist
- ~~Single combined job~~ → Separation of concerns: research vs delivery

## Version Tracking

- Initial version: 2026.9.3 (1391f7c)
- Baseline: `state/progress.json` → `currentVersion` (history in `state/version_history.json`); do not duplicate here
- Version check: Mondays 09:30 during research

## Learning Phases

Authoritative curriculum (50 one-lesson items) is in `docs/planning/00_master_strategy.md`. Keys: `phase_1_core`, `phase_2_gateway`, `phase_3_advanced`, `phase_4_deep`.

## Review 2026-10-06 (ported from copilot-learning and ai-act-learning)

User-approved; full rationale for each pattern is in the sibling MEMORY files. Backups: `tmp/progress.pre-restructure.json`, `tmp/version_history.pre-restructure.json` (git-ignored). Payload drafts: `tmp/payload_weekly.md`, `tmp/payload_daily.md`; curriculum draft `tmp/curriculum_draft.md`.

### Data repaired
- **Streak was 14, real 4** (09-23→09-29 gap, no lessons; 09-30 restarted at 1). All 13 `Day:` lines and footers re-derived from delivery order. 10-02 said Lesson 13 (was 12).
- No queue topics had been lost (luck: the old "Add topics to queue" wording is the one that wiped siblings' queues).
- Lessons were never committed; archived in one commit.

### State restructured
`currentVersion{version,hash,checkedAt}`, `currentPhase`, `phases{…completedTopics[{topic,completedDate,lessonFile}]}`, `queue`, `skippedTopics`, `stats`, `settings` (job ids). Dropped root `completedTopics`, `learningPhase`, `phaseName`, `versionHash`, `nextResearchDate`, `estimatedLessons`; `currentVersion` removed from `version_history.json`. Queue entries carry exactly topic, source, difficulty, section, priority, addedAt.
- Delivered lessons re-filed by content: Phase 1 ×8; Phase 2 ×3 (Automations, Secrets, Failover); Phase 3 ×2 (Sub-agents, Skills).
- Queue: Heartbeat and Exec Approvals → phase_2; the rest phase_1. Memory Search and Exec Approvals raised P3 → P2 (curriculum items).

### Payload rules adopted
Lowest priority number wins, `addedAt` breaks ties · Quick Check one open question, answer in `<details>` · footer derived from `stats` + `section` · phase complete only when every curriculum item delivered · source doc path checked at research and delivery · research adds 3 (queue ≥5) or 7 (<5), append-only with length check · one topic = one lesson · `research_summary_YYYY-MM-DD.md` (legacy `01_` files untouched) · two explicitly staged commits, never `git add -A` · version change updates both history and baseline · weekly delegates to zuko (`agentId: zuko-researcher`, fallback warning); sokka does no research.

### Sessions
- Weekly `a4d548e1` → **Openclaw Weekly Research** `agent:sokka-orchestrator:dashboard:ead9f3e0-b72a-4f8a-8011-e3244692acfd`
- Daily `59423975` → **Openclaw Daily Learning** `agent:sokka-orchestrator:dashboard:208100e5-09ec-4a30-98ae-0bdfb347e9d2`
- Owner stays `agent:sokka-orchestrator:main` (archiving the dedicated sessions may still disable the jobs). Rollback = `sessionKey: agent:sokka-orchestrator:main`.
- Both payloads `thinking: medium` (set by user). Only `sessionKey`, `description`, `message` changed.
- Lesson commit timing: lessons wait until the next Monday research run; untracked files in `lessons/` are not a failure.
- Daily job was paused by the user during the rewrite; re-enable after the manual test.

### Open: end-of-curriculum behaviour (deferred)
Nothing defines what happens when all 4 phases are complete (see copilot-learning MEMORY for the options; maintenance mode recommended). Decide before Phase 4 nears completion. Also open: first real zuko hand-off is AI Act's 2026-10-12 08:30 run; ours is 09:30 the same day.
