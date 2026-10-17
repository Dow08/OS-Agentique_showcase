<p align="right"><b>English</b> · <a href="../fr/politiques-et-schemas.md">Français</a> · <a href="../../README.md">← Back</a></p>

# Policies and schemas: what actually runs beneath the screenshots

The source code stays private. But the **rules** it enforces are data, and data can be shown. Everything below is reproduced from the private repository: the structure of a capability token, the decision order of the enforcement point, and the **sealed** policy files (changing any of them means re-reading it, then re-sealing, under Windows Hello).

---

## 1. A turn's capability token (ADR-081)

Issued by the kernel for **every turn** of an agent, signed with **Ed25519**. The hook that enforces it runs in another process and only holds the public key: it can verify, never forge.

```json
{
  "sub":   "coder",
  "task":  "3f9c…",
  "mode":  "write",
  "root":  "<the task's working folder: the isolated worktree>",
  "allow": [],
  "deny":  ["<resolved junctions, forbidden zones>"],
  "exp":   1759250000,
  "jti":   "a1b2…"
}
```

| Field | Meaning |
|---|---|
| `sub` | the agent profile the token is valid for; a token for another profile is refused |
| `task` | the kernel task that issued the turn |
| `mode` | `read` (no write, no terminal) or `write` (bounded to `root`) |
| `root` | the write root; real paths (links and junctions resolved), never a name prefix |
| `allow` / `deny` | permitted or forbidden sub-scopes inside `root` |
| `exp` | 20 minutes; a token is not individually revocable, it expires (the kill switch cuts everything) |
| `jti` | unique identifier, logged with `capability.issued` |

**Decision order of the enforcement point**, on every tool call, before the effect:

1. Private zones: forbidden to file tools for **every** profile, token or not (including listing a folder that contains them).
2. A token presented but invalid, expired or issued for another profile → refusal.
3. No token: a “kernel only” profile (the offensive department) has **no** tool at all; the others keep their default behaviour.
4. `read` token: no `write_file`, no `patch`, no `terminal`, no `process_manage`, no `execute_code`.
5. `write` token: every path touched, resolved, must be inside `root`, inside `allow` if declared, never under `deny`. The hidden multi-file mode of `patch` is parsed: its paths count. An unverifiable path refuses.

A refusal is logged **without the tool's arguments** (never a file's content nor a command):

```json
{"at":"2026-10-16T21:14:03.512Z","profile":"coder","tool":"write_file","code":"outside_root","task":"3f9c…"}
```

---

## 2. Capability policy (sealed)

```json
{
  "kernelOnly": ["red-team-lead", "recon-specialist", "exploit-analyst", "pentest-prep", "conseil-ethique"]
}
```

The five profiles of the offensive department have **no tool** outside a task launched by the kernel, hence one that went through the risk policy and Windows Hello. A test requires that every profile whose card carries a “high” risk floor is in this list.

---

## 3. Risk policy as data (sealed, ADR-105)

The fixed rules stay in code and are not negotiable here: a git commit or a high risk → approval; a medium risk → approval; low → runs. The file can only **raise** risk (`floor`) or **refuse** (`deny`), never lower it. If unreadable, it closes everything: no task starts.

```json
{
  "version": 1,
  "rules": [
    {
      "id": "formulation-destructive",
      "effect": "floor",
      "risk": "high",
      "when": { "allowEdits": true, "instructionMatches": "\\b(rm\\s+-rf|rmdir\\s+/s|del\\s+/[sq]|git\\s+push\\s+(--force|-f)\\b|git\\s+reset\\s+--hard|git\\s+clean\\s+-[a-z]*f|format\\s+[a-z]:|Remove-Item\\s+[^\\n]*-Recurse)" },
      "reason": "write instruction with a destructive wording (recursive delete, forced push, hard reset): Windows Hello required"
    },
    {
      "id": "profils-hermes-proteges",
      "effect": "deny",
      "when": { "allowEdits": true, "workspaceInside": "@hermes-home/profiles" },
      "reason": "no task writes into the agents' profiles (card, config, hooks): they are deployed by scripts, under the seal"
    }
  ]
}
```

Every rule has its test; a rule without a test fails the suite. The same function serves the real door and the simulation: `policy simulate "…"` explains why a task would be raised or refused, without creating it.

---

## 4. Write transactions (sealed, ADR-105)

```json
{
  "mode": "isolated",
  "executors": ["hermes-agent", "claude-code", "cursor", "shell"],
  "approveDiff": "always",
  "maxPatchBytes": 2097152,
  "maxFiles": 500
}
```

Every write task of one of these executors, inside a git repository, works in a **dedicated worktree**, never in my files. Its diff is shown to me with its fingerprint; integrating it is a second approval. `mode: "direct"` is the tested rollback.

---

## 5. Egress classification per profile (sealed, ADR-071)

Each profile carries a data class; the class decides whether a hosted brain is allowed to see what the agent handles. A real-time hook enforces the rule before the call.

```json
{
  "schema": "hermes.security.egress-policy/1",
  "default": "INTERNAL",
  "classes": {
    "SECRET":       ["rh", "cv-builder", "recruteur", "analyste-carriere", "mentor-carriere", "archiviste", "post-mortem"],
    "CONFIDENTIAL": ["rssi", "analyste-secu", "auditeur-secu", "pentest-prep", "iso-auditor", "red-team-lead", "recon-specialist", "exploit-analyst", "conseil-ethique", "audit-externe", "osint-passif"]
  },
  "cloudAllowed": { "PUBLIC": true, "INTERNAL": true, "CONFIDENTIAL": false, "SECRET": false },
  "exceptions": [
    { "kind": "llm", "executor": "hermes", "agents": ["rh", "cv-builder", "recruteur", "analyste-carriere", "mentor-carriere"], "destination": "Anthropic", "reason": "operator's choice: the HR department may reason with Claude (their subscription); any other cloud stays forbidden" }
  ]
}
```

Accepting a risk means **writing it here**, and the operator's seal signs it. The HR department (my personal data) is classed SECRET: only the local model, or Claude by explicit exception, may serve it; no other provider, ever.

---

## 6. Untrusted data

Everything an agent reads that I did not write myself (a web page, a file, another agent's message, the Lounge thread) is injected into its context under an explicit wrapper, whose inner tags are neutralised:

```xml
<donnees_non_fiables source="web page read by Iris">
…content…
</donnees_non_fiables>
```

The operator's instruction and the agent's memory are never concatenated with that content: they are separate blocks. And whatever that content says, the tool call it might provoke goes through the turn's capability token (§1): the defence does not rest on the model's vigilance, but on what it **cannot** do.

---

Next: [**ADR-081, full text**](adr-081-capability-tokens.md) · [**ADR-105, full text**](adr-105-write-transactions.md) · [**Design & architecture**](design.md)
