<p align="right"><a href="../en/decisions.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Décisions clés

Le projet compte 98 décisions d'architecture écrites (ADR). En voici quatorze qui montrent le mieux ma façon de raisonner : chacune part d'un problème réel, souvent découvert en testant ou en auditant mon propre travail.

---

### 1. Construire autour d'un moteur existant plutôt que d'en réécrire un
**Problème :** j'avais commencé à écrire mon propre moteur d'agents : beaucoup de code, peu de garanties.
**Décision :** dès le deuxième jour, j'ai pivoté vers un moteur open source (Hermes Agent) et je me suis concentré sur ce qui lui manquait : l'organisation, les règles, la vérification, l'interface. L'ancien code a été gelé derrière des adaptateurs plutôt que réécrit.
**Pourquoi c'est important :** savoir quand *ne pas* construire est une compétence d'ingénieur. Cela a fait gagner des semaines et gardé le système maintenable.

### 2. Un seul interlocuteur, qui ne peut que lire
**Problème :** plusieurs assistants (voix, chef de projet, messagerie…) commençaient à s'empiler.
**Décision :** un seul directeur. Il me parle, délègue, et n'a que des droits de lecture.
**Pourquoi c'est important :** l'agent le plus exposé à ce que je dis est aussi le moins puissant. La surface d'attaque diminue.

### 3. Le risque ne peut que monter
**Problème :** un audit croisé a montré que le risque d'une tâche était celui que le demandeur *déclarait*. Un agent pouvait qualifier de « faible risque » une tâche qui écrit des fichiers et éviter mon approbation.
**Décision :** le risque effectif est le plus élevé entre ce qui est déclaré et ce que la tâche peut réellement faire. Prouvé par des tests.
**Pourquoi c'est important :** j'audite mon propre travail. Une règle ne compte qu'une fois qu'un test prouve qu'elle est appliquée.

### 4. « L'IA dit que c'est fait » n'est pas une preuve
**Problème :** une tâche était considérée comme terminée dès que l'IA le disait.
**Décision :** chaque tâche déclare des contrôles (fichier créé, commande réussie, aucun secret divulgué), et tous doivent réussir avant qu'elle puisse être close.
**Pourquoi c'est important :** une IA peut se tromper avec aplomb. C'est la vérification qui la rend fiable.

### 5. Une approbation est liée à ce que j'ai approuvé
**Problème :** approuver par un simple identifiant ne garantissait pas que l'action exécutée était celle que j'avais vue.
**Décision :** chaque approbation est liée à une empreinte de l'action ; toute modification après mon clic l'invalide.
**Pourquoi c'est important :** c'est le principe de la signature d'un document. Cela ferme une faille classique entre le moment du contrôle et celui de l'exécution.

### 6. Jamais le web et mes données personnelles dans les mêmes mains
**Problème :** un agent RH avait à la fois accès au web et à mes données de CV. Une annonce piégée aurait pu lui ordonner de les envoyer à l'extérieur.
**Décision :** une séparation stricte. Léo navigue sur le web sans jamais voir mes données ; Inès lit mes données sans aucun accès au web.
**Pourquoi c'est important :** c'est ainsi qu'on se défend contre l'*injection de prompt*, l'un des principaux risques des agents IA aujourd'hui.

### 7. Un cerveau local, choisi par la mesure
**Problème :** les modèles cloud coûtent de l'argent et font sortir des données ; mon portable n'a pas de carte graphique.
**Décision :** un cerveau par machine. Sur le PC fixe, un modèle local choisi après en avoir comparé plusieurs sur des tâches d'agent (9/10, environ 5 s par échange simple). Sur le portable, un modèle cloud gratuit. Claude n'intervient qu'en repli explicite pour le travail exigeant sur le CV.
**Depuis :** le cerveau se choisit depuis l'interface, en local ou avec Claude (Opus 5.5, Sonnet 5, Haiku 4.5), pour toute l'équipe en un clic, et certains agents gardent un cerveau fixe (la Red Cell toujours en local).
**Pourquoi c'est important :** coût, confidentialité et performance ont été arbitrés avec des données, pas des suppositions, et le choix final reste le mien, tâche par tâche.

### 8. Rendre le travail visible
**Problème :** Hermes a un jour répondu « Je lance une recherche… » sans appeler le moindre outil.
**Décision :** l'interface montre chaque étape réelle en direct, la raison pour laquelle une tâche s'est arrêtée, et signale toute action seulement *annoncée*. Les livrables arrivent dans un dossier dédié sur mon Bureau.
**Pourquoi c'est important :** la confiance vient de la transparence, pas d'une roue qui tourne.

### 9. Les capacités sensibles sont toujours sous contrôle manuel
**Problème :** un mode « Auto » qui approuve seul les tâches à risque moyen est pratique, mais ne doit jamais couvrir des actions de sécurité offensive.
**Décision :** toute tâche de la Red Cell est forcée au niveau de risque maximal, quel que soit le point d'entrée. Elle attend toujours mon clic, et un conseiller éthique relit chaque plan.
**Pourquoi c'est important :** le confort ne passe jamais avant la sécurité. La règle est appliquée côté serveur, pas seulement dans l'interface.

### 10. Des scripts avant l'IA, et un contrat sur chaque dépendance
**Problème :** utiliser l'IA pour des tâches mécaniques est lent, coûteux et imprévisible ; une mise à jour d'outil peut casser une intégration sans bruit.
**Décision :** tout ce qui est mécanique est un simple script. Chaque outil externe dont dépend le système a un *test de contrat* qui échoue bruyamment si son comportement change.
**Pourquoi c'est important :** un système fiable utilise l'IA là où elle apporte de la valeur, et seulement là.

### 11. Déplacer la sécurité de l'écran vers le noyau
**Problème :** un audit de l'exécution (doc 07) a montré que trop de garde-fous vivaient dans l'interface. Ce qui démarrait « à côté » de l'orchestrateur échappait aux règles.
**Décision :** un noyau nommé par lequel **tout** passe, un ordonnanceur unique à l'état partagé entre processus, et une **identité d'appelant** sur chaque tour — le directeur lui-même parle au noyau en son nom.
**Pourquoi c'est important :** une règle de sécurité ne vaut que si le système l'impose. Au niveau du noyau, elle cesse d'être une politesse pour devenir une garantie.

### 12. Un jeton de capacité revérifié à chaque appel d'outil
**Problème :** vérifier les droits au début d'un tour laisse la porte ouverte à un agent qui élargit son pouvoir en cours de route.
**Décision :** le noyau accorde un jeton de capacité précis (quels outils, quel périmètre) **recontrôlé à chaque appel d'outil**, avec un arrêt d'urgence qui descend jusqu'au niveau d'un outil, et des processus enfants qui n'héritent d'aucun nom de secret.
**Pourquoi c'est important :** le principe du moindre privilège, vérifié en continu plutôt qu'une seule fois.

### 13. La mémoire s'apprend, mais sous ma validation
**Problème :** laisser des agents décider seuls de ce qu'ils retiennent, c'est les laisser réécrire leurs propres règles.
**Décision :** les notes de mémoire proposées passent par une file « en attente » : je valide ou je refuse, et c'est le moteur lui-même qui applique ma décision. Seul ce que l'opérateur demande de retenir sur lui s'écrit directement.
**Pourquoi c'est important :** l'équipe peut apprendre sans jamais dériver hors de ce que j'ai approuvé.

### 14. Faire parler les agents entre eux — dans un cadre
**Problème :** un arbre de délégation strict ne laisse jamais émerger de coordination entre agents ; mais les laisser discuter librement est un risque.
**Décision :** un « Salon » où les agents échangent sur le cerveau **local**, bornés par une politique : tickets, mentions, rétrospective hebdomadaire, marche sans surveillance, et ouverture d'un chantier en Coworking **uniquement sur ma décision**.
**Pourquoi c'est important :** on obtient le bénéfice d'une équipe qui se coordonne, sans renoncer au contrôle ni à la traçabilité.

---

Suite : [**Conception & architecture**](conception.md) · [**L'équipe**](equipe.md) · [**Une journée avec Hermes**](parcours.md)
