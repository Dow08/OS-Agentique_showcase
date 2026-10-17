<p align="right"><a href="../en/adr-105-write-transactions.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# ADR-105 — Transactions d'écriture et politique en données (texte intégral)

Second ADR publié en entier. Il règle deux trous du noyau : une tâche d'écriture approuvée écrivait directement dans mes fichiers (désormais : un **worktree isolé**, un diff, une **seconde approbation liée à l'empreinte du diff**), et les règles de risque vivaient dans le code (désormais : un **fichier scellé** que l'on peut lire, simuler et tester, et qui ne peut que relever le risque, jamais l'abaisser).

---

## ADR-105 — Kernel : transactions d'écriture (P1-2) et politique en données (P1-4)

**Statut** : accepté, livré le 2026-10-15. **Choix de l'opérateur** : « on finit le noyau P1-2 et P1-4 » (après revue du projet : consolider le noyau avant toute extension). Audit `docs/architecture/07-agentic-runtime-audit.md` §9.4, §9.5, P1-2, P1-4.

**Contexte** — Deux trous restaient au kernel. (1) L'approbation protégeait l'ACTION, pas l'ÉTAT : une tâche d'écriture approuvée écrivait directement dans l'arbre de l'opérateur, puis la vérification constatait après coup ; un échec laissait les fichiers modifiés. Le worktree n'existait que pour la file autonome du Coworking. (2) La politique était dispersée en TypeScript ; aucune règle ne se lisait, se simulait ni se testait comme une donnée.

### P1-2 — transactions d'écriture
- **WHAT** : `src/core/kernel/transaction.ts` (décision pure, ouverture, diff, intégration, rollback, entretien) orchestré par `task-manager.ts`. Séquence : PRECHECK → SNAPSHOT (worktree git depuis HEAD, `%LOCALAPPDATA%\Hermes\worktrees\<id>`, branche `hermes/q-<id>`) → APPLY (l'executor reçoit le worktree comme `workspace.root`) → VERIFY (sur le résultat isolé) → DIFF (Impact Preview) → APPROVE (empreinte du diff) → COMMIT (`integratePatch` : empreintes des fichiers de l'opérateur, `git apply --check`, apply, aucun commit) ou ROLLBACK (worktree jeté).
- **Éligibilité** (`transactionDecision`, pure) : `allowEdits` sans `readOnly`, executor avec accès fichiers ET dans `policies/transactions.json` (`hermes-agent`, `claude-code`, `cursor`, `shell`), dossier dans un dépôt git avec au moins un commit. Hors transaction, par construction : jeux d'outils `cv`/`salon`, étapes Coworking (un arbre partagé par la mission, intégré par le Coworking), tâche avec `branch` (planificateur), dossier hors git. Une écriture directe le dit : `rollback: { kind: "none", detail }`.
- **Machine à états** : deux transitions ajoutées — `VERIFYING → WAITING_APPROVAL` (diff prêt) et `WAITING_APPROVAL → COMPLETED` (diff intégré). COMPLETED reste inatteignable sans VERIFYING.
- **Seconde approbation** : `approveTask` sur une transaction `awaiting-integration` intègre au lieu d'exécuter ; `actionDigest` = empreinte du patch (ce que l'interface transmet) ; `expectedDigest` différent → `ApprovalInvalidatedError` ; patch altéré sur disque → refus ; fichiers de l'opérateur modifiés depuis le diff → rien n'est appliqué, la tâche reste en attente. Risque `high` → Windows Hello sur l'empreinte du diff. `approval.phase: "diff"` trace la décision. `denyTask` jette le worktree.
- **Dry run** : `--dry-run` / `dryRun: true` (dans l'enveloppe d'action) = transaction arrêtée après le diff, patch conservé dans `memory/tasks/<id>/` (ignoré par git), jamais intégré. Refus franc à la création si la tâche n'est pas isolable.
- **Impact Preview** (sans LLM) : fichiers, lignes +/−, octets, outils réellement appelés pendant le tour ; borné (`maxFiles`, `maxPatchBytes`), troncature signalée. Jarvis › Tâches : aperçu, empreinte, boutons « Intégrer » / « Jeter » ; CLI : `printTask`.
- **Entretien** : `npm run hermes -- transactions [prune]` ; un worktree dont la tâche est terminée ou orpheline est retiré, jamais un diff en attente ni un dossier de la file autonome (aucune transaction rattachée). Le disque fait foi.
- **Journal** : `transaction.opened`, `transaction.diff`, `transaction.integrated`, `transaction.rolled_back`, `transaction.integration_refused`, `transaction.pruned`.

### P1-4 — politique en données
- **WHAT** : `src/core/kernel/policy-data.ts` + `policies/risk.json` (scellé : `policies/` est déjà dans `security/seal-scope.txt`). Deux effets seulement : `floor` (relève à `medium` ou `high`, jamais `low`) et `deny` (refus motivé via `registerDenyRule` du PolicyDecisionPoint). Conditions fermées : `executor`, `profile`, `requestedBy`, `allowEdits`, `readOnly`, `hasParent`, `toolProfile`, `workspaceInside` (alias `@repo`, `@hermes-home`), `instructionMatches` (≤ 200 caractères). Parseur strict (clé inconnue, `when` vide, regex invalide, id en double = refus).
- **Règles fixes intouchables** : `src/core/policy.ts` inchangé (git_commit et high → approbation, medium → approbation). La donnée ne peut qu'ajouter des planchers ou des refus : « risque monotone ».
- **Fermé** : `policies/risk.json` illisible → toute tâche refusée, motif clair ; `doctor` « kernel (politique en données) » en `fail`. Fichier absent → aucune règle de données (le socle fixe reste entier).
- **Simulation** : `explainPolicy` est LA fonction d'enqueueTask (déclaré → dérivé → plancher du profil → règles de données) ; `npm run hermes -- policy simulate "…"` l'appelle avec le vrai PolicyDecisionPoint et la décision de transaction, sans créer de tâche. `policy show` liste les règles.
- **Règles livrées** : `formulation-destructive` (écriture + `rm -rf`, push forcé, reset dur, `git clean -f`, `Remove-Item -Recurse`… → high, donc Hello) ; `profils-hermes-proteges` (aucune écriture sous `HERMES_HOME/profiles` : ils se déploient par scripts, sous le sceau).

- **INTERFACE** : `Task.transaction`, `Task.dryRun`, `TaskApproval.phase` ; `EnqueueParams.dryRun` ; HTTP `dryRun` dans `POST /api/tasks` (un agent ne peut pas le demander) ; CLI `task --dry-run`, `policy [show|simulate]`, `transactions [prune]` ; `doctor` « kernel (politique en données) », « kernel (transactions) » ; `KERNEL_INTERFACES` += `WorkspaceTransaction`, `PolicyData`.
- **SECURITY** : l'arbre de l'opérateur n'est jamais touché par une tâche isolée avant son geste ; l'approbation porte sur ce qui a été écrit (constaté), pas sur ce qui était annoncé ; le jeton de capacité d'un tour (ADR-081) est émis avec le worktree comme racine, donc la PEP borne l'écriture à lui. Tests : `HERMES_WORKTREES_DIR` isolé dans `tests/helpers/isolated-memory.ts`.
- **TEST** : `tests/kernel-transaction.test.ts` (14 : décision pure, Impact Preview, intégration, refus, rollback sur vérification échouée, vérification sur le résultat isolé, dry run, refus de dry run, diff vide, fichiers de l'opérateur modifiés, empreinte et patch altérés, mode direct, `approveDiff: never`, entretien) ; `tests/kernel-policy-data.test.ts` (11 : parseur, monotonie, règles fixes, chaque règle du fichier réel, test de couverture des règles, plancher et refus par la vraie porte, fichier illisible fermé, simulation = réel) ; e2e `panels.spec.ts` « diff en attente » (3 largeurs, axe, « Intégrer » transmet l'empreinte du diff).
- **ROLLBACK** : `"mode": "direct"` dans `policies/transactions.json` (comportement d'avant, testé) ; vider `rules` de `policies/risk.json`. Les deux fichiers sont scellés : re-sceller après modification.

**Preuve réelle (2026-10-15)** : tâche `coder` (cerveau `anthropic`), dépôt jetable du scratchpad. Consigne approuvée → tour réel dans `hermes/q-2c01296e` (outil `write_file`) → vérification `file:notes/preuve.md` sur le résultat isolé → diff 1 fichier +1/−0 en attente ; dépôt de l'opérateur vérifié intact à ce moment → approbation du diff → fichier présent, aucun commit, aucun worktree ni branche restant. 19 événements au journal, de `task.created` à `task.completed`. `npm test` **1348/1348**, deux passages consécutifs ; `npm run ui:gate` vert, 59 tests navigateur. Constat en cours de route : la décision de transaction lançait deux processus git pour TOUTE tâche d'écriture (étapes Coworking comprises), ce qui ralentissait les suites longues sous charge (échecs intermittents du Coworking, passant seuls) ; git n'est plus lancé que si la décision en dépend.

**Écarts assumés et suite** :
- `git apply` suit `core.autocrlf` (true au niveau système sur ce poste) : le fichier intégré prend les fins de ligne d'un checkout, comme l'intégration du Coworking.
- Le dashboard 4242 doit être redémarré pour servir ce code (geste de l'opérateur : jamais sans son accord). L'interface (`ui/dist`) est déjà reconstruite.
- Pas encore : transaction par mission Coworking (un worktree par espace, intégration au « accept ») ; saga pour les effets non fichier (Notion, e-mail : les executors TIK ont déjà `beginEffect`/`finishEffect`) ; politiques `network`/`autonomy` (P2-3, P2-10) ; non-régression sur les décisions passées du journal ; écran Jarvis pour `policy simulate`.

---

> Texte **reproduit tel quel** depuis le registre des décisions du dépôt privé (`DECISIONS.md`), sans retouche : noms de fichiers, chiffres de tests et limites assumées compris. Les chemins cités sont ceux du dépôt privé.
