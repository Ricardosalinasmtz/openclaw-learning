# Master Strategy: OpenClaw Learning System

## Objective

Create an automated, sustainable learning system that teaches OpenClaw concepts through weekly research and daily curated lessons.

## Approach

### Weekly Research Cycle (Mondays 10:00 AM Europe/Rome)

**Agent:** zuko-researcher

**Responsibilities:**
1. Check current OpenClaw version against tracked version
2. If version changed: identify new features, breaking changes, updated docs
3. Explore documentation structure systematically
4. Extract 5-7 teachable topics for the week
5. Add topics to queue with metadata (difficulty, section, estimated lessons)
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

### Daily Lesson Delivery (Mon-Fri 2:00 PM Europe/Rome)

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

**Lesson Format:**
```markdown
# Lesson: [Topic Name]
**Date:** YYYY-MM-DD
**Source:** [doc path]
**Difficulty:** [beginner/intermediate/advanced]
**Phase:** [phase number]

## What is [Topic]?

[Clear explanation in plain language]

## Key Concepts

- Concept 1
- Concept 2
- Concept 3

## Practical Example

[Code snippet or workflow example]

## Why This Matters

[Real-world application]

## Related Topics

- [Link to related concept 1]
- [Link to related concept 2]

## Quick Check

[Reflection question or mini-quiz]
```

## Learning Phases

### Phase 1: Core Concepts (Weeks 1-3)
- agent.md, session.md, memory.md
- tools/ directory essentials
- Basic automations
- Memory system

### Phase 2: Gateway & Configuration (Weeks 4-6)
- Gateway architecture
- Configuration files
- Secrets management
- Tool policies

### Phase 3: Advanced Features (Weeks 7-9)
- Multi-agent orchestration
- Channels and messaging
- Nodes and Control UI
- Plugins and skills

### Phase 4: Deep Dives (Weeks 10+)
- Specifications and protocols
- Reference documentation
- Advanced patterns
- Case studies

## Success Metrics

- **Consistency:** Daily lessons delivered Mon-Fri
- **Completion:** All core topics covered
- **Retention:** User can reference and apply concepts
- **Adaptability:** System responds to version changes

## Risk Mitigation

| Risk | Mitigation |
|------|------------|
| Queue runs empty | Research job prioritizes filling queue |
| Version changes break lessons | Version check flags incompatible content |
| User misses lessons | Lessons archived in lessons/ folder |
| Topics too advanced | Difficulty tagging, adjust based on feedback |

## Git Workflow

- **Before research:** Commit phase plan
- **After research:** Commit research summary
- **Weekly:** Commit lesson archive
- **Version changes:** Tag commit with version

## Next Steps

1. ✅ Create project structure
2. ✅ Initialize state tracking
3. ⏳ Set up automations (weekly research + daily delivery)
4. ⏳ Run first research cycle
5. ⏳ Deliver first lesson
6. ⏳ Monitor and iterate
