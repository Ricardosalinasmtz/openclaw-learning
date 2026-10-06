# OpenClaw Learning System

An automated learning system that teaches OpenClaw concepts daily through curated lessons.

## Overview

This project implements a structured learning approach for mastering OpenClaw:

- **Weekly Research (Mondays 09:30 Europe/Rome):** Explores OpenClaw documentation, identifies teachable topics, and maintains a learning queue
- **Daily Lessons (Mon-Fri 10:00 Europe/Rome):** Delivers one curated lesson from the queue directly to the user

## Structure

```
openclaw-learning/
├── README.md                 # This file
├── MEMORY-openclaw-learning.md  # Project decisions and learnings
├── docs/planning/            # Phase plans and strategies
│   ├── 00_master_strategy.md
│   └── XX_phase_plan.md
├── state/                    # Runtime state tracking
│   ├── progress.json         # Current version, queue, completed topics
│   └── version_history.json  # Version change tracking
├── lessons/                  # Delivered lessons archive
│   └── YYYY-MM-DD_lesson.md
├── scripts/                  # Helper scripts (if needed)
└── tmp/                      # Temporary work files
```

## Automations

1. **Weekly Research Job** - Mondays 09:30 Europe/Rome
   - Checks OpenClaw version for changes
   - Explores documentation for new/important topics
   - Updates the learning queue
   - Delegates to zuko-researcher (sokka only spawns, waits, delivers)
   - Runs in the *Openclaw Weekly Research* session

2. **Daily Lesson Delivery** - Mon-Fri 10:00 Europe/Rome
   - Extracts next topic from queue
   - Formats as an engaging lesson
   - Delivers to the *Openclaw Daily Learning* session

## Current Status

- OpenClaw version / phase / progress: see `state/progress.json` (`currentVersion`, `currentPhase`, `stats`) — not duplicated here.
- Curriculum (4 phases, 50 one-lesson items): `docs/planning/00_master_strategy.md`.

## Usage

Lessons are delivered automatically to the *Openclaw Daily Learning* session (group `openclaw-learning`). To check progress or request specific topics, ask the orchestrator. Untracked files in `lessons/` are expected: the weekly research job commits them on Mondays.
