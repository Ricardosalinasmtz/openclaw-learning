# 🎓 OpenClaw Lesson: Skills System — Tool Instruction Files

**Date:** 2026-09-22 | **Day:** 8 of learning streak
**Source:** tools/skills.md, tools/creating-skills.md, tools/skills-config.md, tools/skill-workshop.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

Skills are how OpenClaw teaches your agent *how* and *when* to use tools — essentially custom instructions baked into the agent system prompt. This lesson covers the entire skills ecosystem: file layout, loading precedence, gating, allowlists, the proposal workflow, and security considerations.

## 📖 The Concept

### What Are Skills?

A skill is a directory containing a `SKILL.md` file with YAML frontmatter and markdown instructions. The frontmatter defines metadata (name, description, gating), and the body gives the agent concrete instructions for using specific tools or workflows. Skills teach the agent *how* to do something — for example, "when the user asks to generate an image, use the `image_generate` tool with these parameters..."

Skills are injected into the agent's system prompt at session start, making them persistent instructions that carry across all turns within that session.

### Loading Order & Precedence

Skills load from multiple sources in a defined priority order. When the same skill name appears in multiple places, the **highest** source wins:

| Priority | Source | Path |
|----------|--------|------|
| 1 — highest | Workspace skills | `<workspace>/skills` |
| 2 | Project agent skills | `<workspace>/.agents/skills` |
| 3 | Personal agent skills | `~/.agents/skills` |
| 4 | Managed / local skills | `<state-dir>/skills` |
| 5 | Workshop skills | `<state-dir>/agents/<agentId>/agent/workshop-skills` |
| 6 | Bundled skills | Shipped with the install |
| 7 — lowest | Extra directories | `skills.load.extraDirs` + plugin skills |

Skills support **grouped layouts** — OpenClaw finds a skill whenever `SKILL.md` appears anywhere under a configured root (up to 6 levels deep):

```
<workspace>/skills/research/SKILL.md          → found as "research"
<workspace>/skills/personal/research/SKILL.md → also found as "research"
```

The folder path is for organization only. The skill's name comes from the `name` frontmatter field.

### Creating a Skill

To create your first skill:

```bash
mkdir -p ~/.openclaw/workspace/skills/hello-world
```

Then write `SKILL.md`:
```text
---
name: hello-world
description: A simple skill that prints a greeting.
---

# Hello World

When the user asks for a greeting, use the `exec` tool to run:

```bash
echo "Hello from your custom skill!"
```bash
```
Key naming rules: use lowercase letters, digits, and hyphens for `name`. Keep `description` to one line under 160 characters.


## 🔑 Key Takeaways

- **Skills are markdown instructions** that teach the agent how to use tools — they're injected into the system prompt, not standalone programs.
- **Precedence matters** — workspace skills override project, which override personal, which override managed, which override bundled skills. Higher wins on name collision.
- **Gating controls eligibility** — a skill can declare required binaries (`requires.bins`), environment variables (`requires.env`), or config paths (`requires.config`). If any gate fails, the skill is excluded.
- **Agent allowlists control visibility** — `agents.defaults.skills` sets a shared baseline; `agents.entries.*.skills` defines the final set per agent (explicit lists **replace** defaults, they don't merge).
- **Skill Workshop is the governance layer** — generated skills go through a proposal lifecycle (pending → evaluate → apply/reject/quarantine), while personal skills publish directly as managed revisions.
- **Environment injection is host-scoped** — `skills.entries.<key>.env` applies to host runs only, not sandboxed runs. For sandboxed skills, pass secrets via `agents.defaults.sandbox.docker.env`.

## 💡 Practical Example

### A Gated Skill with Config

Here's a realistic skill that only loads when Gemini CLI is installed and the API key is set:
**SKILL.md:**

---
name: gemini-search
description: Search the web using Gemini CLI for up-to-date results.
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["gemini"], "env": ["GEMINI_API_KEY"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---

# Gemini Web Search

When the user asks a question that requires up-to-date web results:

1. Use `exec` to run `gemini search "<query>"`
2. Parse the output and synthesize a response
3. Cite the source URLs provided by Gemini

**openclaw.json config to wire an API key:**
```json5
{
  skills: {
    entries: {
      "gemini-search": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" },
      },
    },
  },
}
```

### Using `$` References in Prompts

Type `$` in the Control UI composer to search and insert skill references:

```text
Use $gemini-search and $release_notes to summarize this change for the release.
```

A message can reference up to **8 distinct skills**. OpenClaw resolves each `$skill-name` by reading the referenced `SKILL.md` before acting.

## 🔗 Related Concepts

- **[Skill Workshop](/tools/skill-workshop)** — Proposal lifecycle for agent-drafted skills (create → pending → evaluate → apply)
- **[Skills Config](/tools/skills-config)** — Full `skills.*` config schema, including sandboxed env vars and install policy
- **[Creating Skills](/tools/creating-skills)** — Step-by-step guide for building custom workspace skills
- **[ClawHub](https://clawhub.ai)** — Public registry for community skills (`openclaw skills install @owner/slug`)
- **[Agent Allowlists](/tools/skills-config#agent-allowlists)** — Per-agent skill visibility controls

## 🤔 Quick Check

Imagine you're building a multi-agent setup where the `writer` agent should only have access to `docs-search` and `grammar-check`, while the `analyst` agent should see everything in `agents.defaults.skills` plus `data-utils`. How would you configure `agents.defaults.skills` and `agents.entries`?

*(Hint: explicit lists **replace** defaults — they don't merge.)*

---
*Lesson 9 of Phase 3 (Multi-agent, Channels & Extensibility) | Streak: 8 days*
