<p align="right"><a href="../en/policies-and-schemas.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Politiques et schémas : ce qui tourne réellement sous les captures

Le code source reste privé. Mais les **règles** qu'il applique sont des données, et ces données peuvent être montrées. Tout ce qui suit est reproduit depuis le dépôt privé, à l'état du 4 novembre 2026 : la structure d'un jeton de capacité, l'ordre de décision du point d'application, et les fichiers de politique **scellés** (modifier l'un d'eux impose de le relire, puis de re-sceller, sous élévation).

---

## 1. Le jeton de capacité d'un tour (ADR-081)

Émis par le noyau pour **chaque tour** d'un agent, signé **Ed25519**. Depuis l'ADR-144, il est vérifié dans la **chaîne de garde unique** du moteur natif : dans la boucle d'agent, ou dans le serveur MCP qui sert mes outils au client Claude. Les deux appellent la même fonction. La chaîne vérifie, elle ne fabrique rien.

```json
{
  "sub":   "coder",
  "task":  "3f9c…",
  "mode":  "write",
  "root":  "<dossier de travail de la tâche : le worktree isolé>",
  "allow": [],
  "deny":  ["<jonctions défaites, zones interdites>"],
  "hosts": ["<hôtes web permis pour ce tour, s'il y en a>"],
  "exp":   1759250000,
  "jti":   "a1b2…"
}
```

| Champ | Sens |
|---|---|
| `sub` | le profil d'agent pour lequel le jeton vaut ; un jeton d'un autre profil est refusé |
| `task` | la tâche du noyau qui a émis le tour ; un jeton d'une autre tâche est refusé |
| `mode` | `read` (aucune écriture, aucun terminal, lecture bornée au dossier de travail) ou `write` (borné à `root`) |
| `root` | la racine d'écriture ; chemins réels (liens et jonctions résolus), jamais un préfixe de nom |
| `allow` / `deny` | sous-périmètres permis ou interdits à l'intérieur de `root` |
| `hosts` | bornes web du tour, lues dans le jeton vérifié, jamais dans le contexte de l'agent |
| `exp` | 20 minutes ; un jeton n'est pas révocable individuellement, il expire (l'arrêt d'urgence coupe tout) |
| `jti` | identifiant unique, journalisé avec `capability.issued` |

**La chaîne de garde**, à chaque appel d'outil, avant l'effet :

1. L'outil fait-il partie du jeu du tour ? Sinon, il n'existe pas pour ce tour.
2. Normalisation des arguments, sans extension de droit.
3. Validation par le schéma JSON de l'outil.
4. Chemin réseau (UNC) refusé.
5. Arrêt d'urgence : aucun outil, quel que soit le jeton.
6. Décision du noyau (ci-dessous), jeton vérifié à chaque appel.
7. L'outil, borné à 120 secondes.
8. Registre des sorties : profil, outil, session, jamais les arguments.
9. Sortie bornée à 20 000 caractères et secrets masqués, avant que le texte n'atteigne le modèle ou un journal.

**L'ordre de décision du noyau :**

1. Agent importé d'un bureau : jamais d'écriture ni d'exécution, quel que soit le jeton.
2. Zones privées : interdites aux outils fichiers pour **tout** profil, jeton ou pas (y compris lister un dossier qui les contient).
3. Jeton présenté mais invalide, expiré ou émis pour un autre profil → refus.
4. Pas de jeton : un profil « noyau seulement » n'a **aucun** outil.
5. Jeton `read` : ni `write_file`, ni `patch`, ni `terminal`, ni `process_manage`, ni `execute_code`. La lecture reste dans le dossier de travail et ne touche jamais un fichier de secrets.
6. Jeton `write` : chaque chemin touché, résolu, doit être dans `root`, dans `allow` s'il est déclaré, jamais sous `deny`. Le mode multi-fichiers de `patch` est lu : ses chemins comptent. Un chemin invérifiable refuse.

Un refus est journalisé **sans les arguments** de l'outil (jamais le contenu d'un fichier ni d'une commande) :

```json
{"at":"2026-10-16T21:14:03.512Z","profile":"coder","tool":"write_file","code":"outside_workspace","task":"3f9c…"}
```

---

## 2. Politique des capacités (scellée)

```json
{
  "kernelOnly": ["red-team-lead", "recon-specialist", "exploit-analyst", "pentest-prep", "conseil-ethique", "oeil", "forgeron", "forgeron-juge", "forgeron-banc"]
}
```

Ces neuf profils n'ont **aucun outil** hors d'une tâche lancée par le noyau : les cinq du pôle offensif, Lucie (l'œil) et les trois de la forge. Un test exige que tout profil dont la fiche porte un plancher de risque « élevé » figure dans cette liste.

---

## 3. Politique de risque en données (scellée, ADR-105)

Les règles fixes restent dans le code et ne sont pas négociables ici : un commit git ou un risque élevé → approbation ; un risque moyen → approbation ; faible → s'exécute. Le fichier ne peut que **relever** le risque (`floor`) ou **refuser** (`deny`), jamais abaisser. Illisible, il ferme tout : aucune tâche ne part.

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

La seconde règle date de l'ancien moteur. Il n'y a plus de profil par agent ; elle reste en place, sans objet aujourd'hui.

Chaque règle a son test ; une règle sans test fait échouer la suite. La même fonction sert la vraie porte et la simulation : `policy simulate "…"` explique pourquoi une tâche serait relevée ou refusée, sans la créer.

---

## 4. Transactions d'écriture (scellée, ADR-105)

```json
{
  "mode": "isolated",
  "executors": ["hermes-agent", "agent-natif", "claude-code", "cursor", "shell"],
  "approveDiff": "always",
  "maxPatchBytes": 2097152,
  "maxFiles": 500
}
```

Toute tâche d'écriture d'un de ces exécuteurs, dans un dépôt git, travaille dans un **worktree dédié**, jamais dans mes fichiers. Son diff m'est présenté avec son empreinte ; l'intégrer est une seconde approbation. `mode: "direct"` est le retour arrière, testé. `hermes-agent` n'est plus qu'un alias d'`agent-natif`, accepté à la porte.

---

## 5. Classification des sorties par profil (scellée, ADR-071)

Chaque profil porte une classe de données. Extrait :

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

Accepter un risque, c'est **l'écrire ici**, et le sceau de l'opérateur le signe.

**Attention à ce que ce fichier fait.** Il classe et il constate : le registre rattache chaque sortie à un agent et signale un écart. Il ne bloque pas un appel. Le blocage vient de la politique des cerveaux, au §6.

---

## 6. Cerveaux et moteur en données (scellés, ADR-144)

Extrait de `policies/cerveau.json`, sans le détail des fournisseurs :

```json
{
  "rh_safe": ["ollama", "claude"],
  "locked": { "infra": "ollama", "archiviste": "ollama", "post-mortem": "ollama", "red-team-lead": "ollama", "recon-specialist": "ollama", "exploit-analyst": "ollama", "conseil-ethique": "ollama" }
}
```

- `locked` : ces profils ne tournent que sur le modèle local. Le verrou passe avant tout choix d'équipe.
- `rh_safe` : le pôle RH privé et les agents du Magasin qui lisent un planning nominatif ne tournent que sur ces fournisseurs. Sinon, **le tour est refusé**, sans repli.

Extrait de `policies/executeur.json` :

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

**La limite, écrite dans le code lui-même.** Cette liste de refus lit le texte d'une commande. Une commande construite à l'exécution (concaténation, encodage, script écrit puis lancé) passe ce filtre. Ce n'est pas une frontière d'isolation : tout tourne sous mon compte Windows. Les vraies bornes sont le jeton, le worktree et le diff approuvé ; ce filtre est de la défense en profondeur.

---

## 7. Données non fiables

Tout ce qu'un agent lit sans que ce soit moi qui l'aie écrit (page web, fichier, message d'un autre agent, fil du Salon, fiche de l'œil) est injecté dans son contexte sous une enveloppe explicite, dont les balises internes sont neutralisées :

```xml
<donnees_non_fiables source="page web lue par Iris">
…contenu…
</donnees_non_fiables>
```

La consigne de l'opérateur et la mémoire de l'agent ne sont jamais concaténées à ce contenu : ce sont des blocs distincts. Et quoi que dise ce contenu, l'appel d'outil qu'il pourrait provoquer passe par le jeton de capacité du tour (§1) : la défense ne repose pas sur la vigilance du modèle, mais sur ce qu'il **ne peut pas** faire.

---

Suite : [**ADR-081, texte intégral**](adr-081-jetons-de-capacite.md) · [**ADR-105, texte intégral**](adr-105-transactions-ecriture.md) · [**Conception & architecture**](conception.md)
