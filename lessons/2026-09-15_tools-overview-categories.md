# 🎓 OpenClaw Lesson: Tools Overview & Categories
**Date:** 2026-09-15 | **Day:** 3 of learning streak
**Source:** tools/index.md | **Difficulty:** Beginner

## 🎯 What You'll Learn

OpenClaw agents act through **tools**—callable functions that let them read files, run commands, search the web, generate media, and coordinate with other agents. This lesson explains the three capability surfaces (tools, skills, plugins) and walks through the 12 built-in tool categories so you know which tool to reach for in any situation.

## 📖 The Concept

OpenClaw provides three ways to extend what an agent can do. **Tools** are typed actions the agent calls directly—like `exec` to run shell commands, `read` to open files, or `web_search` to find information. The model only sees tools that pass through active policy filters, so you control exactly what each agent can access.

**Skills** are instruction packs (`SKILL.md` files) that teach agents repeatable workflows. Use skills when the agent already has the right tools but needs guidance on *how* to use them—like a code review rubric, a deployment checklist, or a research protocol. Skills live in workspaces, shared directories, or plugin packages.

**Plugins** add entirely new capabilities to OpenClaw itself. When you need a new integration, model provider, channel type, or runtime surface, you install or build a plugin. Plugins can register new tools, skills, hooks, and even UI components.

The built-in tools are organized into 12 categories: Runtime (exec, process), Files (read, write, edit), Human Input (ask_user, secrets), Web (web_search, web_fetch), Browser, Operator UI, Session Progress, Messaging, Sessions & Agents, Automation, Gateway & Nodes, and Media (image, music, video generation).

## 🔑 Key Takeaways

- **Tools = Actions**: Callable functions that let agents read, write, execute, and communicate
- **Skills = Instructions**: Reusable workflows that teach agents *how* to work with existing tools
- **Plugins = Extensions**: Add new capabilities like providers, channels, or custom tools
- **Policy controls visibility**: The model only sees tools that pass profile, allow/deny, provider, and sandbox filters
- **12 tool categories**: From Runtime and Files to Media and Automation—each serves a distinct purpose

## 💡 Practical Example

Imagine you're building an agent that researches AI regulations and writes a summary report:

```
1. web_search → Find recent articles on EU AI Act
2. web_fetch → Extract readable content from key URLs
3. read → Load project memory and user preferences
4. write → Draft the report to docs/reports/ai-act-summary.md
5. ask_user → Pause for approval before publishing
6. message → Send final report to Telegram channel
```

Each step uses a different tool category. The agent needs **Web** tools for research, **Files** for reading/writing, **Human Input** for approval, and **Messaging** for delivery. A **skill** could encode this entire workflow as a reusable "Research & Report" procedure.

## 🔗 Related Concepts

- [Tools and Custom Providers](/gateway/config-tools) — Full policy reference
- [Skills](/tools/skills) — Creating reusable instruction packs
- [Plugins](/tools/plugin) — Installing and building extensions
- [Exec Approvals](/tools/exec-approvals) — Controlling shell access
- [Automation](/automation) — Scheduling tool-powered workflows

## 🤔 Quick Check

You need an agent that can: (1) analyze a CSV dataset, (2) generate a visualization, and (3) post results to Slack. Which **three tool categories** would it primarily use?

<details>
<summary>Click to reveal answer</summary>

**Runtime** (code_execution or exec for Python analysis), **Media** (image_generate for visualization), and **Messaging** (message or plugin-provided Slack tool for posting).

</details>

---
*Lesson 4 of Phase 1 (Core Concepts) | Streak: 3 days*
