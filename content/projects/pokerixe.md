+++
title = "PokeRixe"
description = "Site de combat de Pokémons tour par tour, développé en Angular (front) et Java Spring (back)."
weight = 1

[taxonomies]
tags = ["angular", "typescript", "java", "spring", "docker", "mariadb", "github-actions", "ci-cd", "sonarqube", "compodoc", "echarts", "fullstack", "devops", "pokemon"]

+++

## Présentation

Pokérixe est un site de combat de Pokémons tour par tour.

Le projet est développé en **2 étapes** :

1. **Le Front-end** — développé seul depuis début 2026.
2. **Le Back-end** — sujet du projet de fin d'année (2ème année), à rendre la première semaine de Mai 2026.

---

## Front-end

| Liens | |
|---|---|
| Code source | [GitHub](https://github.com/BaptouK/PokeRixe) |
| Démo | [pokerixe.baptouk.live](https://pokerixe.baptouk.live/) |

**Framework :** Angular — choisi pour la gestion des événements, le routing et la réactivité nécessaires à une interface de jeu dynamique.

### Technologies


| Technologie | Version | Usage |
|---|---|---|
| [Angular](https://angular.io/) | 21 | Framework front-end |
| TypeScript | ~5.9 | Langage principal |
| CSS custom | — | Styles (responsive mobile inclus) |
| [ECharts](https://echarts.apache.org/) + [ngx-echarts](https://xieziyu.github.io/ngx-echarts/) | 6 / 21 | Graphiques de statistiques |
| [RxJS](https://rxjs.dev/) | ~7.8 | Gestion des flux asynchrones |

### Fonctionnalités disponibles

- Consultation du Pokédex (avec filtre par type)
- Consultation de chaque Pokémon (stats)
- Attaques disponibles pour chaque Pokémon
- Connexion / Inscription utilisateur (mock)
- Gestion d'équipe (slots, mouvements, ordre) (mock)
- Recherche et rejoindre une partie (mock)
- CRUD complet des équipes (mock)
- Interface de combat interactive (mock)

{% admonition(type="info", title="Mock ?") %}
**Mock** signifie que la fonctionnalité est simulée côté front-end avec des données fictives, sans appel à un vrai back-end. Cela permet de développer et tester l'interface sans dépendre du serveur.
{% end %}

---

## Back-end

### Technologies

- **Java Spring** — framework principal de l'API REST
- **MariaDB** — base de données relationnelle
- **Redis** — cache et gestion des sessions

### Fonctionnalités prévues

#### Gestion des utilisateurs

*À venir*

#### Gestion des combats

*À venir*

---

## DevOps & Qualité

*Ces outils s'appliquent à l'ensemble du projet (front et back).*

| Technologie | Version | Usage |
|---|---|---|
| [Compodoc](https://compodoc.app/) | ^1.2 | Documentation du code Front-end |
| [SonarQube](https://www.sonarsource.com/products/sonarqube/) | — | Qualité du code |
| [GitHub Actions](https://github.com/features/actions) | — | CI/CD |


#### Documentation

J'utilise **Compodoc** pour générer automatiquement la documentation technique du front-end à partir des commentaires du code TypeScript.

La documentation est accessible en ligne : [baptouk.github.io/PokeRixe](https://baptouk.github.io/PokeRixe/)

#### Analyse qualité avec SonarQube

**SonarQube** analyse le code à chaque push pour détecter les bugs potentiels, les mauvaises pratiques et mesurer la couverture de tests. Les résultats sont envoyés automatiquement au serveur SonarQube via la CI.

#### CI/CD via GitHub Actions

Le pipeline GitHub Actions orchestre l'ensemble du workflow à chaque push :

1. **Build** — compilation et vérification du projet
2. **Analyse SonarQube** — envoi des métriques au serveur
3. **Build & push Docker** — création de l'image et publication dans le registry
4. **Déploiement** — notification du serveur pour qu'il récupère la nouvelle image
5. **Documentation** — génération Compodoc et déploiement sur GitHub Pages

#### Déploiement avec Docker

Chaque service du projet (front, back, base de données, cache) tourne dans son propre conteneur Docker. Les images sont publiées dans un **registry Docker** dédié et récupérées par le serveur lors de chaque mise à jour.

