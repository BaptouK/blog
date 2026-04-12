+++
title = "Jeu de la vie"
date = 2026-04-11
description = "Implémentation du Jeu de la vie de Conway en C++ avec la bibliothèque SFML."
[taxonomies]
tags = ["cpp", "sfml", "simulation"]
[extra]
toc = true
+++

Le Jeu de la vie est l'un de ces projets que tout développeur croise un jour ou l'autre. Inventé en 1970 par le mathématicien britannique John Horton Conway, ce n'est pas vraiment un jeu — il n'y a ni joueur, ni objectif, ni victoire. C'est un **automate cellulaire** : un système qui évolue tout seul, régi par quelques règles d'une simplicité déconcertante.


Voici le lien du repo de mon projet : [https://github.com/BaptouK/game_of_life](https://github.com/BaptouK/game_of_life)

## Comment ça fonctionne

Le monde se résume à une grille infinie de cellules, chacune pouvant être vivante ou morte. À chaque génération, l'état de chaque cellule est recalculé en fonction de ses huit voisines immédiates, selon quatre règles :

1. Une cellule vivante avec **moins de 2 voisines vivantes** meurt (sous-population).
2. Une cellule vivante avec **2 ou 3 voisines vivantes** survit.
3. Une cellule vivante avec **plus de 3 voisines vivantes** meurt (surpopulation).
4. Une cellule morte avec **exactement 3 voisines vivantes** naît (reproduction).

C'est tout. Quatre règles, et pourtant ces règles suffisent à faire émerger des structures d'une richesse inattendue : des oscillateurs, des vaisseaux qui se déplacent, des canons qui tirent indéfiniment des gliders...

## Le projet : C++ et SFML

J'ai choisi d'implémenter le Jeu de la vie en **C++** avec la bibliothèque **SFML** (Simple and Fast Multimedia Library) pour plusieurs raisons.

D'abord, je voulais consolider mes bases en C++ — gestion mémoire, structures de données, organisation du code en classes. Le Jeu de la vie est un terrain idéal : le problème est bien défini, la logique est claire, et la complexité vient de l'implémentation elle-même plutôt que des règles métier.

J'utilise SFML, c'est une bibliothèque multimédia pensée pour le C++, qui donne accès à une fenêtre graphique, la gestion des événements, le rendu 2D — le tout avec une API propre et accessible.


## Ce que j'ai appris

Implémenter cet automate m'a confronté à quelques questions concrètes :

- **Le patron MVC** : Ce projet était le premier ou j'ai utilisé un modèle de développement pour avoir un programme bine structuré, entre le Modèle, la Vue et le Controlleur.

- **Interaction en temps réel** : la pause, la vitesse, le dessin à la souris — gérer les événements SFML en parallèle de la simulation demande une boucle de jeu bien structurée.

## Résultat

Une fenêtre qui s'ouvre, une grille, et des cellules qui naissent et meurent à chaque tick. On peut dessiner avec la souris, mettre en pause, accélérer. Rien d'extraordinaire visuellement, mais voir un **planeur** (glider) traverser l'écran pour la première fois après avoir écrit soi-même les règles qui le font exister, c'est satisfaisant.

Le Jeu de la vie reste un classique pour une bonne raison : il illustre comment la complexité peut émerger de la simplicité. Et côté code, c'est un projet concret, complet, avec une boucle de rendu, de la logique, et de l'interaction — exactement ce qu'il faut pour progresser.
