<p align="right"><a href="../en/policies-and-schemas.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Politiques et schémas : ce qui tourne réellement sous les captures

Le code source reste privé. Mais les **règles** qu'il applique sont des données, et ces données peuvent être montrées. Tout ce qui suit est reproduit depuis le dépôt privé : la structure d'un jeton de capacité, l'ordre de décision du point d'application, et les fichiers de politique **scellés** (modifier l'un d'eux impose de le relire, puis de re-sceller, sous Windows Hello).

---

## 1. Le jeton de capacité d'un tour (ADR-081)

Émis par le noyau pour **chaque tour** d'un agent, signé **Ed25519**. Le hook qui l'applique tourne dans un autre processus et ne détient que la clé publique : il peut vérifier, jamais fabriquer.

```json
{
  "sub":   "coder",
  "task":  "3f9c…",
  "mode":  "write",
  "root":  "<dossier de travail de la tâche : le worktree isolé>",
  "allow": [],
  "deny":  ["<jonctions défaites, zones interdites>"],
  "exp":   1759250000,
  "jti":   "a1b2…"
}
```

| Champ | Sens |
|---|---|
| `sub` | le profil d'agent pour lequel le jeton vaut ; un jeton d'un autre profil est refusé |
| `task` | la tâche du noyau qui a émis le tour |
| `mode` | `read` (aucune écriture, aucun terminal) ou `write` (borné à `root`) |
| `root` | la racine d'écriture ; chemins réels (liens et jonctions résolus), jamais un préfixe de nom |
| `allow` / `deny` | sous-périmètres permis ou interdits à l'intérieur de `root` |
| `exp` | 20 minutes ; un jeton n'est pas révocable individuellement, il expire (l'arrêt d'urgence coupe tout) |
| `jti` | identifiant unique, journalisé avec `capability.issued` |

**Ordre de décision du point d'application**, à chaque appel d'outil, avant l'effet :

1. Zones privées : interdites aux outils fichiers pour **tout** profil, jeton ou pas (y compris lister un dossier qui les contient).
2. Jeton présenté mais invalide, expiré ou émis pour un autre profil → refus.
3. Pas de jeton : un profil « noyau seulement » (le pôle offensif) n'a **aucun** outil ; les autres gardent leur comportement par défaut.
4. Jeton `read` : ni `write_file`, ni `patch`, ni `terminal`, ni `process_manage`, ni `execute_code`.
5. Jeton `write` : chaque chemin touché, résolu, doit être dans `root`, dans `allow` s'il est déclaré, jamais sous `deny`. Le mode multi-fichiers caché de `patch` est lu : ses chemins comptent. Un chemin invérifiable refuse.

Un refus est journalisé **sans les arguments** de l'outil (jamais le contenu d'un fichier ni d'une commande) :

```json
{"at":"2026-10-16T21:14:03.512Z","profile":"coder","tool":"write_file","code":"outside_root","task":"3f9c…"}
```

---

## 2. Politique des capacités (scellée)

```json
{
  "kernelOnly": ["red-team-lead", "recon-specialist", "exploit-analyst", "pentest-prep", "conseil-ethique"]
}
```

Les cinq profils du pôle offensif n'ont **aucun outil** hors d'une tâche lancée par le noyau, donc passée par la politique de risque et par Windows Hello. Un test exige que tout profil dont la fiche porte un plancher de risque « élevé » figure dans cette liste.

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
      "reason": "aucune tâche n'écrit dans les profils des agents (fiche, config, hooks) : ils se déploient par scripts, sous le sceau"
    }
  ]
}
```

Chaque règle a son test ; une règle sans test fait échouer la suite. La même fonction sert la vraie porte et la simulation : `policy simulate "…"` explique pourquoi une tâche serait relevée ou refusée, sans la créer.

---

## 4. Transactions d'écriture (scellée, ADR-105)

```json
{
  "mode": "isolated",
  "executors": ["hermes-agent", "claude-code", "cursor", "shell"],
  "approveDiff": "always",
  "maxPatchBytes": 2097152,
  "maxFiles": 500
}
```

Toute tâche d'écriture d'un de ces exécuteurs, dans un dépôt git, travaille dans un **worktree dédié**, jamais dans mes fichiers. Son diff m'est présenté avec son empreinte ; l'intégrer est une seconde approbation. `mode: "direct"` est le retour arrière, testé.

---

## 5. Classification des sorties par profil (scellée, ADR-071)

Chaque profil porte une classe de données ; la classe décide si un cerveau hébergé a le droit de voir ce que l'agent manipule. Un hook temps réel applique la règle avant l'appel.

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
    { "kind": "llm", "executor": "hermes", "agents": ["rh", "cv-builder", "recruteur", "analyste-carriere", "mentor-carriere"], "destination": "Anthropic", "reason": "choix de l'opérateur : le pôle RH peut raisonner avec Claude (son abonnement) ; tout autre cloud reste interdit" }
  ]
}
```

Accepter un risque, c'est **l'écrire ici**, et le sceau de l'opérateur le signe. Le pôle RH (mes données personnelles) est classé SECRET : seul le modèle local, ou Claude par exception explicite, peut le servir ; aucun autre fournisseur, jamais.

---

## 6. Données non fiables

Tout ce qu'un agent lit sans que ce soit moi qui l'aie écrit (page web, fichier, message d'un autre agent, fil du Salon) est injecté dans son contexte sous une enveloppe explicite, dont les balises internes sont neutralisées :

```xml
<donnees_non_fiables source="page web lue par Iris">
…contenu…
</donnees_non_fiables>
```

La consigne de l'opérateur et la mémoire de l'agent ne sont jamais concaténées à ce contenu : ce sont des blocs distincts. Et quoi que dise ce contenu, l'appel d'outil qu'il pourrait provoquer passe par le jeton de capacité du tour (§1) : la défense ne repose pas sur la vigilance du modèle, mais sur ce qu'il **ne peut pas** faire.

---

Suite : [**ADR-081, texte intégral**](adr-081-jetons-de-capacite.md) · [**ADR-105, texte intégral**](adr-105-transactions-ecriture.md) · [**Conception & architecture**](conception.md)
