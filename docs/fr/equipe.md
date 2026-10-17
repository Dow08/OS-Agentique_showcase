<p align="right"><a href="../en/team.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# L'équipe

37 agents, organisés comme une entreprise, en 8 pôles. Chaque agent a une carte d'identité (son « âme ») qui fixe sa mission, ses règles et s'il a le droit d'écrire.

**La règle d'or :** les managers lisent, décident et délèguent. Seuls certains exécutants (marqués ✍️) ont le droit d'écrire, et uniquement dans les limites d'une tâche approuvée.

```mermaid
flowchart TD
    H["<b>Hermes</b> · Directeur"]
    H --> DEV["<b>Ada</b> · Développement"]
    H --> RH["<b>Nora</b> · RH & emploi"]
    H --> SEC["<b>Alix</b> · Sécurité défensive"]
    H --> RED["<b>Strike</b> · Red Team"]
    H --> TIK["<b>TIK</b> · Commerce digital"]
    H --> COM["<b>Victor</b> · Commercial"]
    H --> SG["<b>Margot</b> · Secrétariat"]
    H --> AUD["<b>Garance</b> · Audit externe"]
    H -.-> IR["<b>Iris</b> · Navigatrice web"]
    DEV --> A1["Architecte"] & A2["Linus ✍️"] & A3["Grace"] & A4["Testeur ✍️"] & A5["Auditeur ISO"]
    RH --> B1["Camille ✍️"] & B2["Sacha"] & B3["Léo"] & B4["Inès"] & B5["Basile"]
    SEC --> C1["Mira ✍️"] & C2["Elias"] & C3["Owen"]
    RED --> D1["Spectre ✍️"] & D2["Breach ✍️"] & D3["Aegis"]
    TIK --> E1["Noé"] & E2["Jules"] & E3["Lina"] & E4["Maya"] & E5["Hugo"] & E6["Clara"]
    COM --> G1["Théo"] & G2["Chloé"]
    SG --> F1["Hélène"] & F2["Aurèle"]
    AUD --> I1["Sable"]
```

---

## 🎯 Hermes, le directeur

Mon **seul interlocuteur**. Il écoute (voix ou clavier), décide de ce qu'il faut faire, confie le travail au bon pôle et me rend compte. Il ne fait jamais le travail lui-même et ne peut que *lire* : l'agent le plus exposé, celui à qui je parle, est aussi le moins puissant.

---

## 💻 Développement : Ada

| Agent | Rôle |
|---|---|
| **Ada** (manager) | transforme une demande en plan et vérifie que le résultat est complet et testé |
| **Architecte** | conçoit l'approche technique *avant* que quiconque écrive du code |
| **Linus** ✍️ | écrit et corrige le code ; indique précisément les fichiers touchés |
| **Grace** | relit les changements : bugs, failles, tests manquants, classés par gravité. Ne modifie jamais rien |
| **Testeur** ✍️ | lance les tests et rapporte, sans rien « corriger » pour les faire passer |
| **Auditeur ISO** | contrôle le projet au regard de la norme de sécurité ISO 27001 |

*L'Architecte, le Testeur et l'Auditeur ISO ont été conçus par la propre « usine à agents » du système, puis testés et validés par moi avant leur activation.*

---

## 👥 RH & emploi : Nora

| Agent | Rôle |
|---|---|
| **Nora** (manager) | pilote la recherche d'emploi et le travail sur le CV ; quand une information manque, elle **me la demande** au lieu de l'inventer |
| **Léo** | fait la veille des offres publiques chaque matin. Il ne connaît que les critères de recherche, **jamais** mes données personnelles |
| **Inès** | note chaque offre au regard de mon profil et en discute avec moi. Elle lit mon profil mais n'a **aucun accès au web** |
| **Camille** ✍️ | analyse une annonce et rédige un CV adapté, uniquement à partir de faits réels |
| **Sacha** | lit le CV comme un recruteur : « passe-t-il le premier tri, et pourquoi ? » |
| **Basile** | mentor carrière : tient une feuille de route de reconversion (étapes, certifications, bilan hebdomadaire). Chaque proposition est validée par moi ; ses recherches web passent par Iris, sur une liste fermée de sites |

*La séparation entre Léo (le web, sans données personnelles) et Inès (les données personnelles, sans le web) est voulue : une annonce piégée ne peut jamais amener un agent à divulguer mes informations.*

---

## 🛡️ Sécurité défensive : Alix

| Agent | Rôle |
|---|---|
| **Alix** (manager) | planifie le travail de sécurité et **garde l'accès** à mes données personnelles dans tout le système |
| **Mira** ✍️ | détection et réponse à incident : analyse journaux et alertes, écrit des règles de détection et des rapports |
| **Elias** | audits de conformité, en lecture seule : ISO 27001, NIST, RGPD |
| **Owen** | prépare et documente les tests d'intrusion que je mène : périmètre, règles d'engagement, checklists, trames de rapport |

*Le pôle s'appuie sur une bibliothèque locale de 759 guides défensifs (détection, forensique, réponse à incident, durcissement, conformité).*

---

## 🔴 Red Team : Strike

Émulation d'adversaire, **à visée défensive**, sous supervision stricte.

| Agent | Rôle |
|---|---|
| **Strike** (manager) | dirige la cellule et planifie les missions autorisées |
| **Spectre** ✍️ · **Breach** ✍️ | analystes en appui des évaluations de sécurité autorisées ; leur droit d'écriture n'existe que dans une tâche approuvée, « noyau seulement » : lancés hors tâche, ils n'ont aucun outil |
| **Aegis** | **conseiller éthique** : relit chaque plan sous l'angle de l'autorisation, du périmètre, de la légalité, de la proportionnalité et de la protection des données. Il conseille, n'agit jamais |

**Garde-fous intégrés :**
- toute tâche de ce pôle est **forcée au niveau de risque maximal**, quelle que soit la demande ;
- elle **attend donc toujours mon approbation manuelle** ; le mode « Auto » ne peut jamais l'approuver ;
- une action réelle est toujours un geste que j'accomplis moi-même, sur une cible que j'ai l'autorisation de tester ;
- le périmètre autorisé est un fichier que **moi seul** écris ; le directeur ne peut que proposer un programme réel que je lui ai donné, et **jamais** une mission : le pôle ne se délègue pas.

---

## 🛒 Commerce digital : TIK

Un **pôle produit complet** pour construire et faire vivre un vrai site.

| Agent | Rôle |
|---|---|
| **TIK** (manager) | pilote le pôle, cadre les missions et les budgets |
| **Noé** | designer du site ; propose des maquettes, n'écrit pas dans le dépôt du site |
| **Jules** | rédacteur web, sans accès au web : il écrit à partir des faits sourcés par la mission ; l'article est écrit par le code, sur une branche |
| **Lina** | analyste SEO |
| **Maya** | chargée d'études de marché |
| **Hugo** | analyste d'infrastructure du site |
| **Clara** | juriste : socle légal, conformité |

*Frontière stricte « web ⊕ privilège » : un agent a le web **ou** un privilège serveur, jamais les deux dans le même geste. Chaque fait est sourcé ; un plafond de publication borne ce qui peut partir en ligne.*

---

## 🗂️ Secrétariat : Margot

| Agent | Rôle |
|---|---|
| **Margot** (manager) | tâches administratives récurrentes (factures, pièces) ; n'expose que des agrégats |
| **Hélène** | archiviste : lecture locale des pièces, uniquement |
| **Aurèle** | post-mortems : rapports d'incident au format fixe, générés par le code à partir du registre, validés par moi |

*En conditions réelles depuis octobre : la collecte des pièces dans la messagerie est faite par une extension de navigateur (du code, pas un agent), chaque pièce est lue en local, classée dans le Drive selon des règles, et tout cas douteux arrive « à mon attention ». Le directeur ne voit jamais une pièce, un nom ni un montant : seulement des comptes.*

---

## 📞 Commercial : Victor

Une **mission commerciale réelle** (une carte cadeau locale) : recruter des commerces qui l'acceptent, vendre des lots à des acheteurs.

| Agent | Rôle |
|---|---|
| **Victor** (manager) | pilote le pôle et rédige la synthèse ; ne voit que des comptes |
| **Théo** | prospecteur : repère des commerces et organisations sur le web public, **ne reçoit ni ne collecte jamais une coordonnée** |
| **Chloé** | copilote : prépare appels, visites et messages, **sans web** |

*Le CRM, les relances et le plan du jour sont du code, pas un agent. Un brouillon d'e-mail peut être préparé ; **rien n'est jamais envoyé par le système**, c'est moi qui envoie. Une opposition est un drapeau vérifié sur tout chemin de sollicitation.*

---

## 🔎 Audit externe : Garance

Des revues d'exposition publique **passives** (pages publiques, certificats, registres), sur mes propres actifs ou sous mandat écrit.

| Agent | Rôle |
|---|---|
| **Garance** (manager) | cadre les missions ; le périmètre est un fichier que j'écris moi-même |
| **Sable** | reconnaissance passive : lit des sources publiques, jamais le navigateur ni le terminal |

*Chaque constat est validé par le code contre le journal des lectures : un constat sans preuve est rejeté. Elias (pôle sécurité) donne un avis de conformité par mission.*

---

## 🧭 Iris, la navigatrice web

Le **seul** agent autorisé à piloter un navigateur, toujours **à la vue de l'opérateur** et derrière un garde-fou déterministe. Isoler la navigation dans un seul agent surveillé évite qu'une page piégée ne détourne le reste de l'équipe.

---

## 🤝 Coworking

Pas un agent mais un **atelier partagé** : je fixe un objectif, l'équipe y travaille, et je suis un fil d'activité lisible, je réponds aux questions et j'approuve les étapes. Son journal est chaîné par empreintes : toute falsification est détectée.

---

Suite : [**Une journée avec Hermes**](parcours.md) · [**Conception & architecture**](conception.md) · [**Décisions clés**](decisions.md)
