# OpenClaw Learning System

An automated learning system that teaches OpenClaw concepts daily through curated lessons.

## Overview

This project implements a structured learning approach for mastering OpenClaw:

- **Weekly Research (Mondays 10:00 AM):** Explores OpenClaw documentation, identifies teachable topics, and maintains a learning queue
- **Daily Lessons (Mon-Fri 2:00 PM):** Delivers one curated lesson from the queue directly to the user

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

1. **Weekly Research Job** - Mondays 10:00 AM Europe/Rome
   - Checks OpenClaw version for changes
   - Explores documentation for new/important topics
   - Updates the learning queue
   - Delegates to zuko-researcher

2. **Daily Lesson Delivery** - Mon-Fri 2:00 PM Europe/Rome
   - Extracts next topic from queue
   - Formats as an engaging lesson
   - Delivers to the main chat session

## Current Status

- OpenClaw Version: 2026.9.3 (1391f7c)
- System Status: Active
- Learning Phase: Core Concepts

## Usage

Lessons are delivered automatically to the main chat. To check progress or request specific topics, ask the orchestrator.
