<p align="right"><a href="../en/walkthrough.md">English</a> · <b>Français</b> · <a href="../../README.fr.md">← Retour</a></p>

# Une journée avec Hermes

Le système réel, étape par étape. Chaque capture vient de l'interface réelle, avec de vraies réponses du modèle IA local. Rien n'est une maquette. Les données de démonstration sont fictives partout où des informations personnelles apparaîtraient.

---

## Étape 1 · Le réveiller

Un clic sur le raccourci du Bureau lance tout : le modèle IA, le moteur vocal, l'interface. Hermes attend son mot d'activation.

![Accueil : « Dites “Hermès, …” suivi de votre demande »](../../assets/jarvis-accueil.png)

> **Ce que ça montre :** un point d'entrée unique pour tout. Des suggestions aident à démarrer dès la première utilisation.

---

## Étape 2 · Simplement parler

Je lui demande de m'aider à structurer un projet : ici, le site web d'une boulangerie. Hermes répond en 15 secondes environ avec un plan structuré, puis propose de **déléguer** l'étape suivante au bon pôle (développement ou RH).

![Une vraie conversation avec Hermes](../../assets/jarvis-conversation.png)

> **Ce que ça montre :** le directeur réfléchit, puis oriente le travail. Il sait quel pôle fait quoi.

---

## Étape 3 · Une action m'attend

Je lui demande de *créer un fichier*. Cela modifie quelque chose sur mon PC : la tâche est classée **risque moyen** et s'arrête. L'orbe passe à l'ambre : **approbation requise**.

![L'action attend mon approbation](../../assets/jarvis-approbation.png)

> **Ce que ça montre :** l'humain dans la boucle. Rien de ce qui modifie ma machine ne s'exécute sans mon clic explicite.

---

## Étape 4 · Je garde la main

J'ai refusé cette action (ici, la même interface sur téléphone). Hermes confirme : **« Action refusée. »** Rien ne s'est exécuté.

<p align="center"><img src="../../assets/jarvis-mobile.png" alt="Action refusée, sur mobile" width="300"></p>

> **Ce que ça montre :** un refus est définitif, et l'interface fonctionne aussi bien sur téléphone.

---

## Étape 5 · Contrôler la santé du système

Tout ce que fait l'équipe est mesuré, lu dans les journaux du moteur : qui a travaillé quand, combien de temps, avec quels outils, combien de jetons et pour quel coût. Seul Claude, le repli cloud, coûte quelque chose : le cerveau local est gratuit.

![Qui a travaillé quand, par agent](../../assets/profiling-timeline.png)

![Activité par agent : temps, outils, jetons, coût](../../assets/profiling-agents.png)

> **Ce que ça montre :** l'observabilité. Rien n'est inventé ; chaque chiffre est traçable.

---

## Étape 6 · Le système se connaît

Avant chaque décision, le directeur reçoit une **carte réelle** de son équipe, de ses compétences et de ses outils, générée à partir de ce qui est réellement installé.

![Carte de l'équipe et des outils](../../assets/jarvis-carte.png)

> **Ce que ça montre :** l'agent ne devine pas ce qu'il sait faire ; on le lui dit, à partir de l'état réel de la machine.

---

## En coulisses

- toutes les heures, une sauvegarde automatique de tout le système ;
- une recherche de secrets avant chaque sauvegarde : un mot de passe ou une clé la **bloque** ;
- chaque matin, une veille des offres d'emploi qui tourne seule et prépare son résumé.

---

Suite : [**L'équipe**](equipe.md) · [**Conception & architecture**](conception.md) · [**Décisions clés**](decisions.md)
