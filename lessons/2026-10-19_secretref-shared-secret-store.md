# 🎓 OpenClaw Lesson: SecretRef & the Shared Secret Store
**Date:** 2026-10-19 | **Day:** 14 of learning streak
**Source:** gateway/secrets.md | **Difficulty:** intermediate

## 🎯 What You'll Learn

How OpenClaw keeps credentials out of plaintext config via the SecretRef contract — a single object shape that can pull values from environment variables, files, external exec resolvers (Vault, 1Password, Bitwarden), or the shared SQLite secret store. You'll also learn how the runtime snapshot, egress-time sentinels, and the audit/configure workflow work together to shrink the local blast radius of a leaked agent workspace.

## 📖 The Concept

A **SecretRef** is a pointer, not a value. Instead of writing an API key into `openclaw.json`, you write a reference object: `{ source: "env" | "file" | "exec" | "store", provider: "...", id: "..." }`. At Gateway activation, OpenClaw eagerly resolves every active ref into an in-memory **runtime snapshot**, and all runtime request paths read from that snapshot — never re-resolving per request. Secrets never have to sit in a file the agent can read.

The four sources give you flexibility: **env** reads an environment variable (shorthand strings like `${OPENAI_API_KEY}` also work), **file** reads a JSON pointer or whole file from a path with ownership checks, **exec** runs a trusted resolver binary (this is how you plug in Vault, 1Password, Bitwarden, `pass`, or sops — each speaks a tiny JSON protocol on stdin/stdout), and **store** reads from OpenClaw's own **shared secret store**.

The shared store is a Gateway-wide, team-scoped table inside the state SQLite database, managed from Settings → Secrets in the Control UI or via `openclaw secrets store`. It has two access modes: **protected secrets** are write-only after saving (no reveal RPC, never printed), while **agent-readable env** entries are visible to admins and injected as plaintext into Gateway-hosted exec commands — a deliberate trade-off when a script genuinely needs to see the value. A key caveat: store values are *not* encrypted at rest; they're protected by file permissions, and for stronger isolation you point a `store`-or-`exec` provider at an external system instead.

The clever part is **egress-time injection**. For model-provider credentials backed by SecretRefs, OpenClaw hands the call chain an opaque `oc-sent-v2...end` sentinel instead of the real key, and only substitutes the actual value in URLs and headers at the instant a request leaves the process. An opt-in **secret egress proxy** extends the same idea to agent subprocesses: your exec environment gets the sentinel plus proxy variables, and a Gateway-owned loopback proxy decrypts it only for explicitly bound hostnames (`--allow-host api.openai.com`). A request to any other host is refused with an actionable error. It's defense in depth, not process isolation — a plaintext key left in a readable file is still readable, which is why the docs treat migration as complete only when `openclaw secrets audit --check` reports no plaintext residue.

## 🔑 Key Takeaways

- One contract, four sources: `env`, `file`, `exec`, `store` — all the same object shape, resolved eagerly into a runtime snapshot at activation.
- The shared store's protected entries are write-only: nothing (not even an RPC) can read them back; they only become useful when a config field references them or the egress proxy uses them.
- Sentinels (`oc-sent-v2...`) keep real credentials out of the model-call chain; unknown sentinels fail closed rather than being forwarded.
- The egress proxy binds each secret to exact HTTPS hosts — no wildcards, no ports, plain HTTP refused.
- Migration isn't done until the re-audit is clean: `openclaw secrets audit --check` → `openclaw secrets configure --apply` → re-audit.

## 💡 Practical Example

Move an OpenAI key from plaintext config into the shared store, then reference it:

```bash
# 1. Store the key (value is typed into a masked prompt)
openclaw secrets store set OPENAI_API_KEY
```

```json5
// 2. Point the config field at the store — no plaintext anywhere
{
  models: {
    providers: {
      openai: {
        apiKey: { source: "store", provider: "default", id: "OPENAI_API_KEY" }
      }
    }
  }
}
```

```bash
# 3. If an agent subprocess should call the API directly,
#    bind the key to its host and enable the egress proxy
openclaw secrets store set OPENAI_API_KEY --allow-host api.openai.com
openclaw config set secrets.egressProxy.enabled true --strict-json
openclaw gateway restart

# 4. Verify no plaintext residue remains
openclaw secrets audit --check
```

After restart, an agent running `curl https://api.openai.com/v1/models -H "Authorization: Bearer $OPENAI_API_KEY"` sees only the sentinel in its environment; the loopback proxy swaps in the real key for `api.openai.com` and refuses the same request to any other host.

## 🔗 Related Concepts

- [Secrets tool — Credential Management](2026-09-21_secrets-tool-credential-management.md)
- [Gateway architecture & runbook](2026-10-15_gateway-architecture-runbook.md)
- [Exec approvals & allowlists](2026-10-14_exec-approvals-allowlists.md)

## 🤔 Quick Check

You've just onboarded a teammate who runs their own Gateway on a different machine, and you want both gateways to use the same HashiCorp Vault for model-provider credentials. Based on today's lesson, which SecretRef source would you configure for each machine, and why would you run `openclaw secrets audit` with an extra flag after setting it up?

<details>
<summary>Click to reveal the answer</summary>

**Answer:** Use an `exec` source pointing at a Vault resolver binary on each machine — each provider is source-specific, so each Gateway defines its own `secrets.providers` entry with the absolute, non-symlinked binary path, `passEnv` for the Vault address/token, and `trustedDirs`, while both reference the same Vault secret path via the `id`. You'd run `openclaw secrets audit --allow-exec` because audit deliberately skips exec SecretRef resolvability checks by default to avoid command side effects; the flag actually executes the provider so you can verify the resolver really returns the expected values before trusting the migration.

</details>

---
*Lesson 23 of Phase 2 (Gateway, Configuration & Safety) | Streak: 14 days*
