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

## Git Remote & Push (2026-10-14)

- **Remote `origin`:** `git@github.com:Ricardosalinasmtz/openclaw-learning.git` (added 2026-10-14, user request; no push yet).
- `git ls-remote origin` returned success with no refs — the GitHub repo is empty, so the first push needs no `pull --rebase`.
- SSH: the user's GitHub key `~/.ssh/id_ed25519_github_macbook` via the `Host github.com` block in `~/.ssh/config`; `ls-remote` authenticated, so the key was unlocked at setup time. If a later push fails with `Permission denied (publickey)`, re-run `SSH_AUTH_SOCK=/run/user/1000/gnupg/S.gpg-agent.ssh ssh-add ~/.ssh/id_ed25519_github_macbook` in the user's terminal.
- At setup time, uncommitted work in the tree (daily-lesson `progress.json` edits + 3 untracked lessons waiting for Monday's research commit) — pushing only sends committed history.

## Delivery failure 2026-10-19 and switch to `delivery.mode: none` (user change, NOT committed)

- **What happened:** the 10-19 weekly research run (`a4d548e1-9dda-4917-9be0-2f7fae348584`, 09:30) ended with status `error`: "Channel is required (no configured channels detected)…". Only the delivery step failed. The work completed: commits `2b0d731` (Weekly research: added 3 topics) and `863c5db` (Archive lessons for week ending 2026-10-19) exist, `research_summary_2026-10-19.md` is written, and the queue went 5 → 8 (3 topics added; corrected 2026-10-26, this note first said 4 → 7, which was the count after that day's lesson). No duplicates. The job was not re-run, so there are no duplicate commits.
- **Cause:** no chat channel is configured on this host (`openclaw channels list` shows none), so `delivery.mode: announce` has nothing to resolve. Docs: `docs/automation/cron-jobs/delivery.md`.
- **Change made by the user (2026-10-19):** both OpenClaw jobs set to `delivery.mode: none`. Daily `59423975-1f51-4c31-b0b7-1b5e5b12c03a`, weekly `a4d548e1-9dda-4917-9be0-2f7fae348584`; `sessionTarget` now reads `isolated` (was `current`). The `sessionKey` pointing at the Openclaw Daily Learning / Weekly Research sessions is still set, but with `none` nothing is posted there. Results are the files (`lessons/`, `state/`, `docs/planning/`) and the run history.
- **Test:** the user manually ran the daily job after the change. It finished `ok` (status in `openclaw cron list`: `ok`, delivery "not requested") and did not post to Openclaw Daily Learning, as expected. State after it: `totalLessonsDelivered` 23, `currentStreak` 14, `lastLessonDate` 2026-10-19, queue 7, new file `lessons/2026-10-19_secretref-shared-secret-store.md` (untracked, `progress.json` modified, both left for next Monday's research run). The streak is consistent with 10-06 (5) plus 9 consecutive business days.
- **Not verified:** the weekly job's `lastRunStatus` still shows the old `error` until its next run on Mon 2026-10-26 09:30. That run is the first weekly test with `none`; it should end `ok` with two commits.
- **Not chosen:** `delivery.bestEffort: true` (hides the error but does not pick a channel).
- **Side effect to remember:** with no delivery, the weekly payload's "Deliver" step and the daily "Reply" step still run but go nowhere. The lesson text is only in the file, and the "corrected source path" / "fell back to currentPhase" notes live in the run history. Check those there.
- **Status:** note left uncommitted on purpose (user's instruction).

## First weekly run with `delivery: none` — 2026-10-26 (verified 10:18)

- **Weekly job `a4d548e1-…`:** `ok`, delivery "not requested". Commits `ad13382` (Weekly research: added 7 topics to queue) and `e858cc6` (Archive lessons for week ending 2026-10-23), both 09:37:13, named paths only. Summary `docs/planning/research_summary_2026-10-26.md`: version unchanged (2026.9.3, `1391f7c`, only `checkedAt` updated), queue 3 → 10, exactly 7 added, old entries kept their `addedAt`, all 7 are Phase 3 gaps at priority 2, every source path verified to exist. `phases`/`stats` untouched. The 10-19 failure is therefore resolved by `none`.
- **Not provable from files:** whether zuko or the sokka fallback did the research. The summary has no fallback warning and the commits look normal. Run history is in the Automations page; `automations runs` from an interactive chat returned "not found".
- **Daily job `59423975-…` at 10:00:** `ok`. Lesson 28 of Phase 3, streak 19: "Multi-agent Routing & Bindings" (`lessons/2026-10-26_multi-agent-routing-bindings.md`, footer correct). Queue 10 → 9, `lastUpdated` is a plain date (`2026-10-26`), so the ISO-date payload fix from 10-06 is confirmed on a real run. Lesson and `progress.json` stay uncommitted until next Monday's research run.
- **Progress:** Phase 1 and 2 `completed` (13/13, 12/12), Phase 3 `in_progress` (3 delivered of 13), Phase 4 `locked`. Delivered 28, streak 19.
- **Phase 3 gap:** only "Nodes & media understanding" (curriculum item 13) is not queued yet; expect the 11-02 research run to add it.
- **`origin` still unpushed.** Local `main` is ahead of the empty GitHub repo. Before the first push the user must run the `ssh-add` line above in their own terminal; no push without the user's go.
- **Minor, left as is:** `addedAt` mixes full timestamps (older entries) and plain dates (new ones); sorting still works.

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
- Daily job was paused by the user during the rewrite and re-enabled after the manual test (confirmed by user 2026-10-06).

### Manual test 2026-10-06 10:20 — passed
Forced daily run: "Context: What the Model Sees", lesson 14 of Phase 1, streak 5 (continued from 4), source `concepts/context.md`. Quick Check was one open question with the answer in `<details>`. State: `totalLessonsDelivered` 14, queue 7 → 6, Phase 1 has 9 completed topics, no root `completedTopics`, trailing newline present. No commit made; lesson file and `progress.json` stay uncommitted until Monday's research run. Delivered to Openclaw Daily Learning, not Home.
- Minor: the run wrote `lastUpdated` as a full timestamp. Daily payload step 7 now says "today's ISO date (YYYY-MM-DD, no time)" (applied to the live job 2026-10-06, user-approved; draft `tmp/payload_daily.md` updated). Not yet verified on a run; first check is tomorrow's 10:00 lesson.

### Open: end-of-curriculum behaviour (deferred)
Nothing defines what happens when all 4 phases are complete (see copilot-learning MEMORY for the options; maintenance mode recommended). Decide before Phase 4 nears completion. Also open: first real zuko hand-off is AI Act's 2026-10-12 08:30 run; ours is 09:30 the same day.
