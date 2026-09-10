# MEMORY-openclaw-learning.md

Project-specific memory for the OpenClaw Learning System.

<!-- project: github.com/Ricardosalinasmtz/sokka-orchestrator -->

## Decisions

- **Automation over external cron:** Uses OpenClaw's built-in automations for better integration and reliability
- **Weekly collection, daily delivery:** Research on Mondays, lessons Mon-Fri (not random/hourly)
- **Version-aware learning:** Track OpenClaw version changes and prioritize new features
- **Curriculum approach:** Structured phases instead of random topic selection
- **Direct chat delivery:** Lessons delivered to main session, not webhooks

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
- Version check: Mondays at 10:00 AM during research phase

## Learning Phases

1. **Phase 1 - Core Concepts:** agent.md, session.md, memory.md, tools/
2. **Phase 2 - Gateway & Configuration:** gateway/, configuration.md, automations
3. **Phase 3 - Advanced Features:** multi-agent, channels, nodes, plugins
4. **Phase 4 - Deep Dives:** specs/, reference/, advanced patterns
