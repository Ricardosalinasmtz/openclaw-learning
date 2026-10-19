# 🎓 OpenClaw Lesson: Configuration Basics
**Date:** 2026-10-16 | **Day:** 13 of learning streak
**Source:** gateway/configuration.md, gateway/configuration-reference.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

OpenClaw is configured with a single, entirely optional JSON5 file at `~/.openclaw/openclaw.json`. Today covers the two-bucket layout of that file, the four ways to edit it (wizard, CLI, Control UI, direct file edit), why OpenClaw refuses to boot on an invalid config, and how hot reload decides which changes apply live versus which force a restart.

## 📖 The Concept

The config file is JSON5, so comments and trailing commas are fine. If the file doesn't exist, OpenClaw simply runs on safe defaults — you only add config when you need channels, models, tool policies, sandboxing, or automation. Two layout rules matter: the file must be a regular file (a symlinked config gets its target replaced on writes, so avoid symlinked layouts), and if you keep config elsewhere, point `OPENCLAW_CONFIG_PATH` at the real file. Structurally, config follows a two-bucket rule: root-level siblings hold infrastructure and cross-agent defaults, while `agents.defaults` holds agent-loop behavior. Entries under `agents.entries` may override either bucket where the schema supports a per-agent override. When in doubt about a field, `config.schema.lookup` is the first stop for exact field-level docs before editing.

You can edit config four ways: the interactive `openclaw onboard` / `openclaw configure` wizards, CLI one-liners like `openclaw config get/set/unset`, the Control UI's schema-rendered Config tab (with a Raw JSON escape hatch), or a direct file edit that the Gateway watches. The strictness is the big deal: OpenClaw only accepts configs that fully match its schema. Unknown keys, malformed types, or invalid values cause the Gateway to **refuse to start**. When that happens, only diagnostic commands work (`doctor`, `logs`, `health`, `status`), and you repair with `openclaw doctor` to see the exact issues and `openclaw doctor --fix` to apply repairs. A last-known-good copy is kept after each successful startup, but it's only restored by `doctor --fix` — never automatically. Invalid hot reloads are skipped and the runtime keeps the last accepted config, and writes that look like accidental clobbers (dropping `gateway.mode` or halving the file size) are blocked outright.

Once the file is valid, the Gateway watches it and applies changes automatically in `hybrid` reload mode (the default): hot-reloadable settings apply live, and a restart is triggered only when required. Most of the day-to-day surface hot-applies — agent and model settings, automation (`hooks`, `cron`, heartbeat), sessions, tools, plugin config, and even rotating the gateway auth token if the auth mode stays the same. A smaller set needs a full restart, notably `gateway.*` server settings like port, bind, and TLS, plus a few infrastructure keys. Programmatic updates go through the config RPC: `config.schema.lookup` to inspect a subtree, `config.get` for a snapshot with a `hash`, `config.patch` for partial updates (objects merge, `null` deletes, array removals require explicit `replacePaths`), and `config.apply` only for a full replacement. Finally, secrets never have to sit in the file in plaintext: env vars load from `.env` files and inline `env.vars`, config strings support `${VAR_NAME}` substitution, and credential fields accept SecretRef objects pointing at the shared secret store.

## 🔑 Key Takeaways

- Config is optional JSON5 in `~/.openclaw/openclaw.json`; missing file means safe defaults, and it must be a regular file, not a symlink.
- Two-bucket rule: root siblings = infrastructure and cross-agent defaults, `agents.defaults` = agent-loop behavior, `agents.entries` = per-agent overrides.
- Strict validation is real: an invalid config means the Gateway won't start — use `openclaw doctor` and `doctor --fix`; last-known-good is only restored by `doctor --fix`, never automatically.
- Hybrid hot reload applies most changes live (agents, automation, sessions, tools, plugins); gateway server settings like port, bind, and TLS require a restart.
- Check the live schema before editing: `openclaw config schema` or `config.schema.lookup`; the reference page is a field map that delegates deep detail to dedicated subsystem pages.

## 💡 Practical Example

Change the heartbeat cadence with a CLI one-liner and verify it:

```bash
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config get agents.defaults.heartbeat
```

Because heartbeat lives under `agents`, the Gateway hot-applies it in hybrid mode — no restart, the next cadence just runs. Now add a Telegram channel:

```json5
// ~/.openclaw/openclaw.json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "${TELEGRAM_BOT_TOKEN}",   // env substitution, secret not in file
      dmPolicy: "pairing",                  // unknown senders get a pairing code
    },
  },
}
```

The watcher picks up the edit, validates it against the schema, and restarts just the channel subsystem. If you typo a key instead, you'll see `config reload skipped (invalid config)` — the running config keeps working, and you fix it with `openclaw doctor --fix`. If the change had been the gateway port, hybrid mode would have scheduled a full restart automatically.

## 🔗 Related Concepts

- [SecretRef & the shared secret store](gateway/secrets.md)
- [Doctor, health & troubleshooting](gateway/doctor.md)
- [Gateway architecture & runbook](gateway/index.md)
- [Multi-agent routing](concepts/multi-agent.md)

## 🤔 Quick Check

You maintain an OpenClaw gateway on a laptop and need two changes: shorten the heartbeat cadence and switch the gateway's bind port from 18789 to 19000. Explain, using what you learned today, how you'd make each change and what the Gateway does differently for each one — then say what happens if a stray comma makes the file invalid while you're editing, and what you'd run to recover.

<details>
<summary>Click to reveal the answer</summary>

**Answer:** The heartbeat setting lives under `agents.defaults`, which hot-applies in hybrid reload mode — edit it via `openclaw config set` or the file, and the new cadence takes effect with no restart. The bind port is a `gateway.*` server setting, which requires a full Gateway restart; hybrid mode will schedule that restart automatically when it applies the change. If the stray comma makes the file invalid, the hot reload is skipped and the Gateway keeps running on the last accepted config; if a restart or boot is then required, the Gateway refuses to start and only diagnostic commands work. You'd run `openclaw doctor` to see the exact issue and `openclaw doctor --fix` to repair — that's also the only path that restores the last-known-good copy.

</details>

---
*Lesson 22 of Phase 2 (Gateway, Configuration & Safety) | Streak: 13 days*
