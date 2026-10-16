<p align="right"><a href="../en/build-log.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Comment ça a été construit

Un mois, 301 commits, 112 décisions écrites, et pas une ligne de code tapée à la main : tout a été produit par des agents IA (Claude Code, ponctuellement Cursor) que j'ai dirigés. Cette page raconte **dans quel ordre** les briques sont apparues, **pourquoi** chacune est née, et **comment** j'ai tenu le projet.

Les numéros renvoient aux ADR (*Architecture Decision Records*) du dépôt privé : une décision = une page, avec son contexte, le choix, les alternatives écartées et la preuve.

---

## Jour 1 — 15 septembre : écrire avant de coder

Le projet commence par une décision de principe : **un OS d'agents, pas un empilement de scripts** (ADR-001). Avant la moindre fonctionnalité, huit documents de conception :

| Document | Ce qu'il fixe |
|---|---|
| Carte d'architecture | huit plans (interaction, intention, planificateur, agents, fournisseurs, exécuteurs, coworking, auto-amélioration), chacun projeté sur ce qui existe déjà |
| Contrats | les interfaces entre plans, validées avant tout code (ADR-006) |
| Machines à états | la vie d'une tâche, d'une approbation, d'un exécuteur |
| Modèle de menaces | ce qu'un agent peut faire de mal, et ce qui l'en empêche |
| Plan de test | tests isolés, tests de contrat pour tout outil externe |

Deux choix fondateurs : construire **autour** d'un moteur open source (Hermes Agent) plutôt que d'en écrire un (ADR-003), et faire des *hooks* natifs du moteur mon **bus d'événements** (ADR-004). Un identifiant de tâche unique corrèle tous les journaux (ADR-005).

## Semaine 1 — le cœur, puis la voix

- **Le cœur d'orchestration** : une tâche est une machine à états ; l'exécution ne part qu'à la transition « prête » (ADR-007) ; le risque effectif est le **maximum entre ce qui est déclaré et ce que la tâche peut réellement faire** (ADR-015) ; « terminé » exige que toutes les vérifications déclarées passent (ADR-016).
- **Les exécuteurs** : Claude Code, Cursor, un shell à liste blanche, et le moteur lui-même ; sortie bornée, arbre de processus réellement tué (ADR-018).
- **La sécurité de base** : filtre de secrets avant toute écriture, jamais un prompt en argument shell, validation de forme contre l'injection de commande (ADR-014, ADR-019).
- **Le planificateur** (chaîne séquentielle, pas un graphe arbitraire, ADR-010), le **Coworking** (ADR-012) et le **moteur de frictions** qui n'apprend que de signaux réellement détectables (ADR-013).
- **Un seul interlocuteur** : le directeur, en lecture seule (ADR-021, ADR-022).
- **Jarvis** : la conversation vocale locale, par la même porte que tout le reste (ADR-025), puis le lancement d'un clic et les accusés vocaux (ADR-029).

## Semaine 2 — des pôles qui font du vrai travail, et voir l'équipe

- **Pôle RH** : le CV Creator comme outil de travail, données personnelles jamais sur GitHub (ADR-026) ; cerveau local d'abord, évaluation déterministe, reprise automatique par Claude si la réponse ne suffit pas (ADR-027) ; le **circuit CV** (Camille rédige, Sacha relit, livraison seulement sans point bloquant, ADR-038) ; puis la **recherche d'emploi** en cinq lots : critères, veille, tri, miroir Notion, automatisation (ADR-042 → 049), avec l'invariant « web ⊕ données personnelles » (ADR-045) et des questions à l'opérateur plutôt que des inventions (ADR-047).
- **Pôle sécurité défensive** : connaissance documentaire, aucune exécution offensive (ADR-031) ; le pôle offensif forcé au niveau de risque maximal (ADR-032, ADR-033).
- **Voir l'équipe** : suivi en direct détaillé (ADR-034), alertes de dérive déterministes (ADR-035), matrice des accès (ADR-036), profilage « qui a fait quoi, avec quels outils, pour combien » (ADR-039), tâches orphelines soldées (ADR-040).
- **Coworking** façon atelier partagé (ADR-043), **empreinte d'action** : une approbation est liée à ce qui a été approuvé (ADR-044), puis fusion conversation + Coworking en un seul accueil (ADR-055).
- **Sélecteur de cerveau** : local ou Claude, pour toute l'équipe, en un clic (ADR-056) ; **Iris**, seule à piloter un navigateur (ADR-057).

## Semaine 3 — le noyau, le SOC, le Salon

Un **audit interne du runtime agentique** (29 septembre) a été le tournant : il listait les primitives qui manquaient pour que les règles soient *imposées* et non *affichées*. La réponse a été le noyau :

- **Kernel nommé**, la tâche comme objet système, `deny` et `RETRYING` réels (ADR-077) ; **identité d'appelant** : opérateur, jeton local ou agent, avec des droits différents (ADR-078) ; **arrêt au niveau de l'outil** sur tous les profils (ADR-079) ; **processus enfants** sans aucun nom de secret hérité (ADR-080) ; **jetons de capacité** vérifiés à chaque appel d'outil (ADR-081) ; **ordonnanceur unique** pour tous les processus (ADR-083).
- **SOC local** : Security Control Plane adossé au noyau, broker à catalogue, élévation UAC par action (ADR-061) ; approbation à risque élevé sous **Windows Hello** (ADR-063) ; audit chaîné, sceau, arrêt d'urgence, mode sûr (ADR-069) ; registre des sorties des agents (ADR-071) ; Sysmon avec configuration maison (ADR-072).
- **Le directeur délègue, il ne fait pas** : plus de web, de fichiers ni de terminal pour lui, un serveur MCP comme seule porte (ADR-070).
- **Salon d'agents** : fil visible, notes de mémoire proposées (ADR-084), chantiers proposés (ADR-088), tickets et mentions sur le cerveau local (ADR-090), rétro hebdomadaire et marche sans surveillance (ADR-096, ADR-097). **Mémoires en attente** (ADR-086, ADR-089).
- **Pôle Commerce digital** : cadre et socle déterministe (ADR-082), passerelle hébergeur en lecture seule (ADR-085), le pôle et sa frontière « web ⊕ privilège » (ADR-087), site et pipeline (ADR-092), contenu à portes (ADR-093), opportunités et juriste (ADR-095).

## Semaine 4 — industrialiser, puis redessiner

- **Panneau d'administration du serveur** : shell d'administration par recettes, journal chaîné, console SSH (ADR-102) ; **veille de marché autonome**, sources fermées, rien de publié sans moi (ADR-103).
- **Gate frontend** : un système de design écrit, un lint de design, Playwright sur une instance isolée aux données fictives, captures à trois largeurs ; rien ne fusionne sans la gate verte (ADR-104).
- **Transactions d'écriture et politique en données** : worktree isolé, diff à seconde approbation, règles de risque scellées et simulables (ADR-105).
- **Quatre nouveaux pôles ou rôles** en un chantier : Commercial (ADR-106), Audit externe (ADR-107), post-mortems (ADR-108), mentor carrière (ADR-109) ; **Secrétariat en conditions réelles** (ADR-113).
- **Refonte orbitale de Jarvis** : prototypes HTML validés d'abord, puis anneau, hexagone, cockpits par pôle, barre unifiée, vue Équipe, cockpit GRC, vue Systèmes (ADR-112) : sept lots livrés en deux jours, chacun construit par une session et vérifié par une autre.
- **Un incident, un post-mortem, une règle** : une clé de noyau surchargée dans une instance de vérification avait signé le vrai journal d'audit ; désormais le journal refuse une clé étrangère et toute signature inattendue doit être attestée par moi (ADR-114).

---

## Ce qui a changé de cap en route

| Ce que je croyais | Ce que j'ai appris | La trace |
|---|---|---|
| La sécurité peut vivre dans l'interface | une règle non imposée par le système est une politesse | audit du 29 septembre → noyau (ADR-077 → 083) |
| Un modèle local suffit partout | il suffit pour le Salon, la Red Team et le secrétariat ; le directeur et le pôle développement ont besoin de plus | sélecteur de cerveau, cerveaux fixes par agent (ADR-056) |
| Une approbation suffit | une approbation sans empreinte approuve n'importe quoi ; une écriture sans transaction est irréversible | ADR-044, ADR-105 |
| Les agents peuvent écrire leur mémoire | ils proposent, je valide ; sinon la mémoire dérive | ADR-064, ADR-086 |
| Le directeur peut « faire un peu » | dès qu'il fait, il travaille sans le contexte de l'agent compétent | ADR-070 |
| Les captures d'écran prouvent l'interface | seulement si elles sont faites sur des données fictives et régénérées par la gate | ADR-104 |

---

## La méthode, concrètement

1. **Un brief** : ce que je veux, en français, à la voix ou au clavier.
2. **Une porte** : WHAT · WHY · INTERFACE · SECURITY · TEST · ROLLBACK, écrite par l'agent, relue par moi. Pas de code avant.
3. **Un document de conception** pour les gros chantiers, découpé en **lots** ; chaque lot est livré vert (tests, lint, gate) avant le suivant.
4. **Une session par lot, dans son worktree** ; deux sessions n'écrivent jamais les mêmes fichiers. Un **vérificateur distinct** relit, lance la gate, mesure à l'écran sur une instance isolée.
5. **Un ADR** par décision, écrit dans le même lot que le code.
6. **Un checkpoint horaire** qui commite et pousse, avec un détecteur de secrets et de données privées.
7. **Des audits** réguliers (application, runtime, UX) et un **post-mortem** par incident.

Ce que je retiens pour un employeur : le vibe coding ne remplace pas la méthode, il la rend **indispensable**. Ce dépôt montre une méthode qui a tenu un mois, sur ≈ 95 000 lignes, sans incident de sécurité non traité.

---

Suite : [**Conception & architecture**](conception.md) · [**L'équipe**](equipe.md) · [**Décisions clés**](decisions.md)
