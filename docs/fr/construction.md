<p align="right"><a href="../en/build-log.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Comment ça a été construit

Sept semaines, 788 commits, 148 décisions écrites, ≈ 253 000 lignes de TypeScript, et pas une ligne de code tapée à la main : tout a été produit par des agents IA (Claude Code, ponctuellement Cursor) que j'ai dirigés. Cette page raconte **dans quel ordre** les briques sont apparues, **pourquoi** chacune est née, et **comment** j'ai tenu le projet.

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

Deux choix fondateurs : construire **autour** d'un moteur open source (Hermes Agent) plutôt que d'en écrire un (ADR-003), et faire des *hooks* natifs du moteur mon **bus d'événements** (ADR-004). Un identifiant de tâche unique corrèle tous les journaux (ADR-005). Le premier de ces choix a tenu six semaines : il a été défait en semaine 6.

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

## Semaine 5 — 17 au 23 octobre : le pouls, le coach, et une panne qui couve

- **Coach carrière** : Basile passe de la feuille de route à un vrai suivi en cinq sprints : plan d'objectifs et hypothèses à confirmer, liens officiels vérifiés par le code *avant* de m'être soumis, carte « Cette semaine », compétences et quiz, entretien blanc, candidatures et marché par métier, première prise en main (ADR-119 → 121, ADR-130, ADR-132 → 134).
- **Le Pouls** : un registre de routines plafonnées, un minuteur sur le PC, un relevé du serveur (ADR-123) ; un vérificateur nocturne des liens de la documentation et de la mémoire (ADR-122) ; un digest du matin rédigé par le directeur et vérifié par le code (ADR-125).
- **Le Cerveau** : une carte visuelle des agents et de la mémoire, en six vues (ADR-124).
- **Mesurer avant d'activer** : un banc compare les modes d'injection de la carte au directeur. Par défaut, elle n'est pas injectée (ADR-126).
- **La panne qui couve** : une mise à jour du moteur amont (17 octobre) fait passer les tours par une passerelle partagée, isolée par profil et non par tâche. Constaté le 19 octobre : un tour pouvait hériter de l'identité et du mode d'un autre. Une parade par relance à chaque tour (ADR-129), un essai d'exécution sous un compte Windows séparé (ADR-131). Elles tiennent, mais à un tour à la fois.

## Semaine 6 — 26 au 29 octobre : sortir du moteur

- **Cerveau v3** : le Dashboard devient l'accueil, avec le Second cerveau derrière lui (ADR-138).
- **L'œil** : une image jointe est lue par Lucie, un agent sans outil. Le directeur reçoit une fiche, jamais l'image. Une consigne cachée dans une capture reste une donnée (ADR-139).
- **Bureaux d'agents** : 161 spécialistes importés d'une bibliothèque open source (VoltAgent, licence MIT), en 9 bureaux avec un manager chacun. Importeur déterministe, source épinglée, fichiers scellés, aucune écriture (ADR-140).
- **La forge** : une boucle rédacteur → banc → juge produit prompts et skills, notés sur des cas de test, jamais déployés sans ma relecture (ADR-141).
- **Le coût d'un tour** : décomposition mesurée, poids retiré du prompt du directeur (ADR-142).

Le 27 octobre, tout s'arrête. L'activation des 152 profils de huit bureaux porte le total à 211. La passerelle met alors 3 à 5 minutes à démarrer, le moteur n'en attend que 60 secondes, sans réglage possible. Plus aucun tour ne passe, pour aucun agent, de 14 h 15 à 20 h 33.

Ma conclusion : le moteur ne m'apportait plus que la boucle modèle ↔ outils, un client MCP, la mémoire, les skills et la session. Tout le reste était déjà à moi. J'ai décidé de le remplacer (**ADR-144**) : tout sous mon compte Windows, Claude par son client officiel, cerveau interchangeable, interface inchangée.

Le plan a été écrit le 27, relu par une IA externe, puis construit dans la nuit par des sessions parallèles, lot par lot. Le 28 au matin, le code ne contenait plus Hermes Agent. **Jarod est resté le nom du directeur.**

| Mesure | Hermes Agent | Moteur natif |
|---|---|---|
| Même question, 5 questions de parité | 98 267 jetons, 58 s | 4 509 à 9 406 jetons, ~5,5 s |
| Tour du directeur, délégation comprise | 25 594 jetons, 66 s | 19 330 jetons, 9 s |
| Porte de sécurité du moteur | — | 36 cas sur 36 |
| Contrôle réel d'un tour par pôle | — | 9 sur 9 |

Ce que ça a coûté : les mises à jour amont et le fournisseur gratuit du portable. Le compte séparé pour les agents et son runner, essayés la semaine d'avant, ont été retirés : **tout tourne sous mon compte**, la borne réelle d'une écriture est le worktree et le diff approuvé. Le code pour Linux est écrit et testé sous Windows, mais **rien n'a encore tourné sur une vraie machine Linux**.

Dans la foulée : le mode chantier du sceau, un seul resceau à la validation (ADR-146).

## Semaine 7 — 2 au 4 novembre : le Magasin et l'audit qui le rattrape

- **Pôle Magasin** : l'exploitation d'un commerce alimentaire de proximité, dans le cockpit Commercial, avec six agents en lecture seule (ADR-148). Planning d'équipe et vente par heure payée (VHP), registre de sécurité et HACCP, rapprochement des pièces, veille locale, météo.
- **Un audit produit et UX, le jour même** : la première version était déclarée validée, alors que plusieurs intégrations étaient factices ou injoignables. L'audit a relevé 11 points « déclarés livrés mais faux ». J'ai fait réécrire le document du pôle pour dire le réel, puis reconstruire ces points : moteur de planning unique, VHP calculée sur de vraies heures, contrats partagés entre serveur et interface, réactivité, Dashboard en mur de widgets, palette de commandes, kit responsive.
- **Le planning, repris sur pièces** : il ne trouvait pas de solution sur une vraie semaine. Le code explore maintenant avec retour arrière, à graine fixe. Thomas et Claire relisent ensuite, et les semaines que je valide servent de modèle.
- **Les pôles consultent les bureaux** : tout agent de pôle peut poser une question à un spécialiste, sauf le RH privé et l'Audit externe (ADR-149).
- **Moins de jetons** : un aiguillage par palier (veille sur Haiku, code sur Sonnet, planning et architecture sur Opus), à activer par mon choix (ADR-150) ; une mission de bureau en un tour dans le cas courant, une sur cinq relue par le manager (ADR-151). Le rejeu sur 83 vrais tours donne −15 % : c'est une projection, pas une mesure.
- **Le cerveau local** : l'attente suit la progression réelle du modèle, 480 s au plus, et le préfixe stable est lu d'avance (ADR-153).

---

## Ce qui a changé de cap en route

| Ce que je croyais | Ce que j'ai appris | La trace |
|---|---|---|
| La sécurité peut vivre dans l'interface | une règle non imposée par le système est une politesse | audit du 29 septembre → noyau (ADR-077 → 083) |
| Un modèle local suffit partout | il suffit pour le Salon, la Red Team et le secrétariat ; le directeur et le pôle développement ont besoin de plus | sélecteur de cerveau (ADR-056), paliers (ADR-150) |
| Une approbation suffit | une approbation sans empreinte approuve n'importe quoi ; une écriture sans transaction est irréversible | ADR-044, ADR-105 |
| Les agents peuvent écrire leur mémoire | ils proposent, je valide ; sinon la mémoire dérive | ADR-064, ADR-086 |
| Le directeur peut « faire un peu » | dès qu'il fait, il travaille sans le contexte de l'agent compétent | ADR-070 |
| Les captures d'écran prouvent l'interface | seulement si elles sont faites sur des données fictives et régénérées par la gate | ADR-104 |
| Il ne faut pas réécrire un moteur | c'était vrai au jour 2 ; au bout de six semaines, le moteur n'apportait plus que sa boucle et il bloquait tout | ADR-003 → ADR-144 |
| Un compte Windows séparé bornerait les agents | l'essai a fonctionné, au prix de pannes de droits répétées ; j'ai choisi le compte unique, limite assumée | ADR-131 → ADR-144 |
| « Tests verts » veut dire « livré » | une fonction peut passer ses tests sur un faux et ne rien faire en vrai | audit du 2 novembre, ADR-148 |

---

## La méthode, concrètement

1. **Un brief** : ce que je veux, en français, à la voix ou au clavier.
2. **Une porte** : WHAT · WHY · INTERFACE · SECURITY · TEST · ROLLBACK, écrite par l'agent, relue par moi. Pas de code avant.
3. **Un document de conception** pour les gros chantiers, découpé en **lots** ; chaque lot est livré vert (tests, lint, gate) avant le suivant.
4. **Une session par lot, dans son worktree** ; deux sessions n'écrivent jamais les mêmes fichiers. Un **vérificateur distinct** relit, lance la gate, mesure à l'écran sur une instance isolée.
5. **Un ADR** par décision, écrit dans le même lot que le code.
6. **Un checkpoint horaire** qui commite et pousse, avec un détecteur de secrets et de données privées.
7. **Des audits** réguliers (application, runtime, UX) et un **post-mortem** par incident.

Ce que je retiens pour un employeur : le vibe coding ne remplace pas la méthode, il la rend **indispensable**. La preuve la plus nette est la semaine 7 : sans audit, une version « validée » serait restée en place. Le système a tenu un changement de moteur en une nuit parce que les contrats, les tests et le noyau ne dépendaient pas de lui.

---

Suite : [**Conception & architecture**](conception.md) · [**L'équipe**](equipe.md) · [**Décisions clés**](decisions.md)
