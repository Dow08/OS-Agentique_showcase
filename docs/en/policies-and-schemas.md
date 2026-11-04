<p align="right"><b>English</b> · <a href="../fr/politiques-et-schemas.md">Français</a> · <a href="../../README.md">← Back</a></p>

# Policies and schemas: what actually runs beneath the screenshots

The source code stays private. But the **rules** it enforces are data, and data can be shown. Everything below is reproduced from the private repository, as of 4 November 2026: the structure of a capability token, the decision order of the enforcement point, and the **sealed** policy files (changing any of them means re-reading it, then re-sealing, under elevation).

---

## 1. A turn's capability token (ADR-081)

Issued by the kernel for **every turn** of an agent, signed with **Ed25519**. Since ADR-144, it is verified in the **single guard chain** of the native engine: in the agent loop, or in the MCP server that serves my tools to the Claude client. Both call the same function. The chain verifies; it forges nothing.

```json
{
  "sub":   "coder",
  "task":  "3f9c…",
  "mode":  "write",
  "root":  "<the task's working folder: the isolated worktree>",
  "allow": [],
  "deny":  ["<resolved junctions, forbidden areas>"],
  "hosts": ["<web hosts allowed for this turn, if any>"],
  "exp":   1759250000,
  "jti":   "a1b2…"
}
```

| Field | Meaning |
|---|---|
| `sub` | the agent profile the token is valid for; a token from another profile is refused |
| `task` | the kernel task that issued the turn; a token from another task is refused |
| `mode` | `read` (no write, no terminal, reading bounded to the working folder) or `write` (bounded to `root`) |
| `root` | the write root; real paths (links and junctions resolved), never a name prefix |
| `allow` / `deny` | sub-scopes allowed or forbidden inside `root` |
| `hosts` | the turn's web bounds, read from the verified token, never from the agent's context |
| `exp` | 20 minutes; a token cannot be revoked individually, it expires (the emergency stop cuts everything) |
| `jti` | unique identifier, logged with `capability.issued` |

**The guard chain**, on every tool call, before its effect:

1. Is the tool in the turn's set? If not, it does not exist for this turn.
2. Argument normalisation, with no extension of rights.
3. Validation against the tool's JSON schema.
4. Network (UNC) path refused.
5. Emergency stop: no tool, whatever the token.
6. Kernel decision (below), with the token verified on every call.
7. The tool, bounded to 120 seconds.
8. Egress registry: profile, tool, session, never the arguments.
9. Output bounded to 20,000 characters and secrets masked, before the text reaches the model or a log.

**The kernel's decision order:**

1. Agent imported into a bureau: never a write or an execution, whatever the token.
2. Private zones: forbidden to file tools for **every** profile, token or not (including listing a folder that contains them).
3. Token presented but invalid, expired or issued to another profile → refused.
4. No token: a "kernel only" profile has **no** tools.
5. `read` token: no `write_file`, `patch`, `terminal`, `process_manage` or `execute_code`. Reading stays in the working folder and never touches a secrets file.
6. `write` token: every path touched, once resolved, must be inside `root`, inside `allow` if declared, never under `deny`. The multi-file mode of `patch` is parsed: its paths count. An unverifiable path is refused.

A refusal is logged **without the tool's arguments** (never the content of a file or command):

```json
{"at":"2026-10-16T21:14:03.512Z","profile":"coder","tool":"write_file","code":"outside_workspace","task":"3f9c…"}
```

---

## 2. Capability policy (sealed)

```json
{
  "kernelOnly": ["red-team-lead", "recon-specialist", "exploit-analyst", "pentest-prep", "conseil-ethique", "oeil", "forgeron", "forgeron-juge", "forgeron-banc"]
}
```

These nine profiles have **no tools** outside a task started by the kernel: the five in the offensive department, Lucie (the eye) and the three in the forge. A test requires any profile whose card carries a "high" risk floor to be on this list.

---

## 3. Risk policy as data (sealed, ADR-105)

The fixed rules stay in code and are not negotiable here: a git commit or high risk → approval; medium risk → approval; low → runs. The file can only **raise** risk (`floor`) or **refuse** (`deny`), never lower it. If unreadable, it closes everything: no task starts.

```json
{
  "version": 1,
  "rules": [
    {
      "id": "formulation-destructive",
      "effect": "floor",
      "risk": "high",
      "when": { "allowEdits": true, "instructionMatches": "\\b(rm\\s+-rf|rmdir\\s+/s|del\\s+/[sq]|git\\s+push\\s+(--force|-f)\\b|git\\s+reset\\s+--hard|git\\s+clean\\s+-[a-z]*f|format\\s+[a-z]:|Remove-Item\\s+[^\\n]*-Recurse)" },
      "reason": "consigne d'écriture à formulation destructive (suppression récursive, push forcé, reset dur) : Windows Hello exigé"
    },
    {
      "id": "profils-hermes-proteges",
      "effect": "deny",
      "when": { "allowEdits": true, "workspaceInside": "@hermes-home/profiles" },
      "reason": "aucune tâche n'écrit dans les profils Hermes (SOUL, config, hooks) : ils se déploient par scripts/team.ps1 et les scripts de profil, sous le sceau"
    }
  ]
}
```

*(Reasons are kept in French, as in the file: a write instruction with a destructive wording requires Windows Hello; no task writes into the agent profiles.)*

The second rule dates from the old engine. There is no longer one profile per agent; it stays in place, with no purpose today.

Each rule has its test; a rule without a test fails the suite. The same function serves the real door and the simulation: `policy simulate "…"` explains why a task would be raised or refused, without creating it.

---

## 4. Write transactions (sealed, ADR-105)

```json
{
  "mode": "isolated",
  "executors": ["hermes-agent", "agent-natif", "claude-code", "cursor", "shell"],
  "approveDiff": "always",
  "maxPatchBytes": 2097152,
  "maxFiles": 500
}
```

Any write task from one of these executors, in a git repository, works in a **dedicated worktree**, never in my files. Its diff is shown to me with its fingerprint; integrating it is a second approval. `mode: "direct"` is the tested rollback. `hermes-agent` is now only an alias of `agent-natif`, accepted at the door.

---

## 5. Egress classification per profile (sealed, ADR-071)

Each profile carries a data class. Extract:

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
    { "kind": "llm", "executor": "claude-cv", "reason": "ADR-027 : l'operateur a choisi Claude (son abonnement) pour la redaction des CV" },
    { "kind": "llm", "executor": "hermes", "agents": ["rh", "cv-builder", "recruteur", "analyste-carriere", "mentor-carriere"], "destination": "Anthropic", "reason": "ADR-027 elargi (choix de l'operateur, 2026-10-17) : le pole RH (Nora, Camille, Sacha, Ines, Basile) peut raisonner avec Claude (son abonnement) ; tout autre cloud reste interdit" }
  ]
}
```

*(Reasons are kept in French, as in the file: the operator chose Claude, by subscription, for CV writing and for the HR department; any other cloud stays forbidden.)*

Accepting a risk means **writing it here**, and the operator's seal signs it.

**Be careful about what this file does.** It classifies and it records: the registry ties each output to an agent and flags a gap. It does not block a call. Blocking comes from the brain policy, in §6.

---

## 6. Brains and engine as data (sealed, ADR-144)

Extract of `policies/cerveau.json`, without the provider details:

```json
{
  "rh_safe": ["ollama", "claude"],
  "locked": { "infra": "ollama", "archiviste": "ollama", "post-mortem": "ollama", "red-team-lead": "ollama", "recon-specialist": "ollama", "exploit-analyst": "ollama", "conseil-ethique": "ollama" }
}
```

- `locked`: these profiles only run on the local model. The lock outranks any team choice.
- `rh_safe`: the private HR department and the Store agents reading a named schedule only run on these providers. Otherwise **the turn is refused**, with no fallback.

Extract of `policies/executeur.json`:

```json
{
  "engines": ["agent-natif"],
  "default": "agent-natif",
  "regimes": {
    "conversation": { "maxTurns": 8,  "contextTokens": 20000, "memoryEntries": 8,  "skills": false, "reasoning": "none" },
    "mission":      { "maxTurns": 12, "contextTokens": 32000, "memoryEntries": 12, "skills": true,  "reasoning": "low" },
    "atelier":      { "maxTurns": 40, "contextTokens": 48000, "memoryEntries": 12, "skills": true,  "reasoning": "medium" }
  },
  "terminal": { "timeoutMs": 120000, "maxOutputChars": 20000,
                "deny": { "pwsh": ["-EncodedCommand", "Invoke-Expression", "iex ", "Start-Process", "Invoke-WebRequest", "iwr ", "irm ", "curl ", "reg ", "schtasks", "$HOME", "%USERPROFILE%", "%APPDATA%"],
                          "bash": ["curl ", "wget ", "sudo ", "crontab", "eval ", "$HOME", "~/"] } }
}
```

**The limit, written in the code itself.** This deny list reads a command's text. A command built at run time (concatenation, encoding, a script written then launched) gets past the filter. This is not an isolation boundary: everything runs under my Windows account. The real bounds are the token, the worktree and the approved diff; this filter is defence in depth.

---

## 7. Untrusted data

Anything an agent reads that I did not write myself (web page, file, message from another agent, Lounge thread, card from the eye) is injected into its context under an explicit envelope, with inner tags neutralised:

```xml
<donnees_non_fiables source="page web lue par Iris">
…content…
</donnees_non_fiables>
```

The operator's instruction and the agent's memory are never concatenated with that content: they are separate blocks. And whatever that content says, any tool call it might trigger goes through the turn's capability token (§1): the defence does not rest on the model's vigilance, but on what it **cannot** do.

---

Next: [**ADR-081, full text**](adr-081-capability-tokens.md) · [**ADR-105, full text**](adr-105-write-transactions.md) · [**Design & architecture**](design.md)
