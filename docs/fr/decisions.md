<p align="right"><a href="../en/decisions.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Décisions clés

Le projet compte 148 décisions d'architecture écrites (ADR). En voici vingt qui montrent le mieux ma façon de raisonner : chacune part d'un problème réel, souvent découvert en testant ou en auditant mon propre travail.

---

### 1. Construire autour d'un moteur existant, puis savoir en sortir
**Problème :** j'avais commencé à écrire mon propre moteur d'agents : beaucoup de code, peu de garanties.
**Décision :** dès le deuxième jour, j'ai pivoté vers un moteur open source (Hermes Agent) et je me suis concentré sur ce qui lui manquait : l'organisation, les règles, la vérification, l'interface. L'ancien code a été gelé derrière des adaptateurs plutôt que réécrit.
**Depuis :** six semaines plus tard, j'ai retiré ce moteur (décision 15). Le choix de départ reste bon : il m'a permis de construire tout le reste d'abord.
**Pourquoi c'est important :** savoir quand *ne pas* construire est une compétence d'ingénieur. Savoir quand un choix a cessé d'être juste en est une autre.

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
**Depuis :** la même règle tient au Magasin : Ombeline a le web et aucune donnée d'équipe ; Thomas et Claire ont le planning nominatif et pas le web.
**Pourquoi c'est important :** c'est ainsi qu'on se défend contre l'*injection de prompt*, l'un des principaux risques des agents IA aujourd'hui.

### 7. Un cerveau local, choisi par la mesure
**Problème :** les modèles cloud coûtent de l'argent et font sortir des données ; mon portable n'a pas de carte graphique.
**Décision :** sur le PC fixe, un modèle local choisi après en avoir comparé plusieurs sur des tâches d'agent (9/10). Claude quand le travail l'exige.
**Depuis :** le cerveau se choisit depuis l'interface, pour l'équipe, un pôle ou un agent. Le choix vient d'une liste fermée de modèles vérifiés. Le portable, lui, passe par Claude ; le fournisseur gratuit a disparu avec l'ancien moteur. La Red Team, Hugo, Hélène et Aurèle sont verrouillés en local.
**Pourquoi c'est important :** coût, confidentialité et performance ont été arbitrés avec des données, pas des suppositions, et le choix final reste le mien.

### 8. Rendre le travail visible
**Problème :** Jarod a un jour répondu « Je lance une recherche… » sans appeler le moindre outil.
**Décision :** l'interface montre chaque étape réelle en direct, la raison pour laquelle une tâche s'est arrêtée, et signale toute action seulement *annoncée*. Les livrables arrivent dans un dossier dédié sur mon Bureau.
**Pourquoi c'est important :** la confiance vient de la transparence, pas d'une roue qui tourne.

### 9. Les capacités sensibles sont toujours sous contrôle manuel
**Problème :** un mode « Auto » qui approuve seul les tâches à risque moyen est pratique, mais ne doit jamais couvrir des actions de sécurité offensive.
**Décision :** toute tâche de la Red Team est forcée au niveau de risque maximal, quel que soit le point d'entrée. Elle attend toujours mon clic, et un conseiller éthique relit chaque plan. Ce pôle tourne sur le cerveau local, verrouillé par une politique scellée.
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
**Décision :** les notes de mémoire proposées passent par une file « en attente » : je valide ou je refuse, et mon geste applique la note. Ce qui concerne le pôle ou moi-même attend toujours. Une note écrite pendant un tour qui a eu le web attend aussi.
**Pourquoi c'est important :** l'équipe peut apprendre sans jamais dériver hors de ce que j'ai approuvé.

### 14. Faire parler les agents entre eux — dans un cadre
**Problème :** un arbre de délégation strict ne laisse jamais émerger de coordination entre agents ; mais les laisser discuter librement est un risque.
**Décision :** un « Salon » où les agents échangent sur le cerveau **local**, bornés par une politique : tickets, mentions, rétrospective hebdomadaire, marche sans surveillance, et ouverture d'un chantier en Coworking **uniquement sur ma décision**.
**Pourquoi c'est important :** on obtient le bénéfice d'une équipe qui se coordonne, sans renoncer au contrôle ni à la traçabilité.

### 15. Remplacer le moteur d'agents (ADR-144)
**Problème :** une mise à jour amont faisait passer les tours par une passerelle isolée par profil, pas par tâche : un tour pouvait hériter de l'identité d'un autre. Mes parades ont limité le système à un tour à la fois. Le 27 octobre, à 211 profils, la passerelle mettait 3 à 5 minutes à démarrer pour un délai fixe de 60 secondes : plus aucun tour ne passait.
**Décision :** une boucle d'agent dans le dépôt, avec deux familles de cerveaux (API et client Claude officiel), une seule chaîne de garde pour tous les appels d'outils, des cerveaux et un moteur décrits en données.
**Preuve :** même question : 98 267 jetons et 58 s avant, 4 509 à 9 406 jetons et ~5,5 s après. Tour du directeur : 25 594 jetons et 66 s avant, 19 330 et 9 s après. Porte de sécurité 36/36, un vrai tour vert par pôle (9/9).
**Prix assumé :** plus de mises à jour amont ; tout tourne sous mon compte Windows, le terminal est borné mais ce n'est pas une isolation du système. La borne réelle est le worktree et le diff approuvé.
**Pourquoi c'est important :** les contrats, les tests et le noyau ne dépendaient pas du moteur. C'est ce qui a rendu le changement possible en une nuit.

### 16. Des bureaux de spécialistes, sans pouvoir (ADR-140, ADR-149)
**Problème :** je voulais une flotte de spécialistes sans écrire un prompt par besoin et sans exploser les jetons.
**Décision :** 161 prompts open source (VoltAgent, licence MIT) importés par un programme déterministe, depuis une source épinglée, en 9 bureaux avec un manager chacun. Chaque fichier est scellé : modifié, il est refusé. Le code présélectionne les candidats, le manager choisit et contrôle. Ces agents n'écrivent, n'exécutent ni ne délèguent, quel que soit leur jeton. Depuis le 3 novembre, les agents de pôle peuvent les consulter, sauf le RH privé et l'Audit externe.
**Preuve :** une délégation complète passe de 95 100 à ≈ 63 400 jetons, côté agents de 43 500 à ≈ 11 500.
**Pourquoi c'est important :** un prompt importé est une donnée qu'on a relue. Il ne décide d'aucun droit.

### 17. La forge : un prompt se juge sur des cas de test (ADR-141)
**Problème :** chaque prompt était écrit à la main et jugé au goût. Un modèle qui se note lui-même converge vers du joli, pas vers du juste.
**Décision :** trois agents sans outil : Anselme écrit les cas de test puis les versions, Colin joue le candidat, Honorine note sur une grille fixe. Le code décide, et ajoute toujours au moins un cas d'injection : obéir à la consigne cachée plafonne la note. Rien n'est déployé sans ma relecture, et le dépôt se fait en brouillon.
**Pourquoi c'est important :** on applique à un prompt ce qu'on applique au code : un test rejouable plutôt qu'un avis.

### 18. L'œil voit, le cerveau raisonne (ADR-139)
**Problème :** une image jointe arrivait au directeur, qui tient les outils de délégation. Un texte caché dans une capture devenait une consigne, et aucun prompt ne détecte ça de façon fiable : le modèle lit avant de pouvoir juger.
**Décision :** Lucie lit l'image, sans aucun outil, et rend une fiche au schéma imposé. Le directeur reçoit la fiche comme donnée non fiable, jamais l'image, et n'a pas d'outil pour la faire regarder de nouveau. Secrets masqués avant toute sortie.
**Limites :** un code QR n'est pas décodé. Si l'œil suit un cerveau hébergé, l'image part chez ce fournisseur, comme dans une conversation Claude ; un réglage le garde en local.
**Pourquoi c'est important :** la frontière n'est pas « cet agent est-il prudent », c'est « d'où vient l'autorité ». Ma consigne est un ordre, le contenu d'une image est une donnée.

### 19. Un modèle par difficulté, choisi par le code (ADR-150)
**Problème :** une veille coûtait le même modèle qu'une décision d'architecture. La mesure sur 83 vrais tours a corrigé mon intuition : le vrai poste n'était pas le directeur, c'était Léo (28 %).
**Décision :** un mode « Automatique » : veille sur Haiku, code et directeur sur Sonnet, planning, architecture et relecture sur Opus. Le palier dépend du rôle, sans appel au modèle, et ne fait que monter. Une réponse trop faible est reprise une fois, un cran au-dessus. Rien ne change tant que je ne le choisis pas.
**Honnêteté :** le rejeu donne −15 % sur ces deux semaines. C'est une projection aux prix publics, pas une mesure, et la qualité de la veille sur Haiku reste à juger.
**Pourquoi c'est important :** mesurer d'abord a évité une erreur : mon premier réglage aurait augmenté la consommation.

### 20. Le Magasin : le code calcule, les agents relisent, je valide (ADR-148)
**Problème :** gérer un commerce alimentaire de proximité, avec de vrais plannings soumis au droit du travail. Une erreur touche des personnes.
**Décision :** planning, heures et VHP sont calculés par du code (8 règles, plus la cohérence du registre). Six agents en lecture seule proposent et relisent ; un planning ne se valide que par mon geste, et sans alerte bloquante. Aucune donnée réelle de l'équipe dans le dépôt, le checkpoint bloque les noms réels. Les agents qui lisent un planning nominatif n'ont pas le web et ne tournent que sur le modèle local ou Claude.
**Ce qui n'allait pas :** la première version a été déclarée validée avec plusieurs intégrations factices. Un audit l'a relevé, le document du pôle dit depuis ce qui n'est pas branché, et l'extraction automatique depuis le logiciel de gestion du magasin n'existe pas : elle passe par un import de relevés.
**Pourquoi c'est important :** dans un métier réel, dire ce qui ne marche pas vaut plus que d'annoncer ce qui marche.

---

Suite : [**Conception & architecture**](conception.md) · [**L'équipe**](equipe.md) · [**Une journée avec Jarod**](parcours.md)
