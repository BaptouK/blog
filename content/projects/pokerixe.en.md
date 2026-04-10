+++
title = "PokeRixe"
description = "Turn-based Pokémon battle website, developed with Angular (front-end) and Java Spring (back-end)."
weight = 1

[taxonomies]
tags = ["angular", "typescript", "java", "spring", "docker", "mariadb", "github-actions", "ci-cd", "sonarqube", "compodoc", "echarts", "fullstack", "devops", "pokemon"]

+++

## Overview

Pokérixe is a turn-based Pokémon battle website.

The project is developed in **2 stages**:

1. **Front-end** — developed solo since early 2026.
2. **Back-end** — end-of-year project (2nd year), due the first week of May 2026.

---

## Front-end

| Links | |
|---|---|
| Source code | [GitHub](https://github.com/BaptouK/PokeRixe) |
| Demo | [pokerixe.baptouk.live](https://pokerixe.baptouk.live/) |

**Framework:** Angular — chosen for its event management, routing and reactivity needed for a dynamic game interface.

### Technologies

| Technology | Version | Usage |
|---|---|---|
| [Angular](https://angular.io/) | 21 | Front-end framework |
| TypeScript | ~5.9 | Main language |
| Custom CSS | — | Styles (mobile responsive included) |
| [ECharts](https://echarts.apache.org/) + [ngx-echarts](https://xieziyu.github.io/ngx-echarts/) | 6 / 21 | Stats charts |
| [RxJS](https://rxjs.dev/) | ~7.8 | Async stream management |

### Available features

- Pokédex browser (with type filter)
- Individual Pokémon view (stats)
- Available moves for each Pokémon
- User login / registration (mock)
- Team management (slots, moves, order) (mock)
- Search and join a game (mock)
- Full team CRUD (mock)
- Interactive battle interface (mock)

{% admonition(type="info", title="Mock?") %}
**Mock** means the feature is simulated on the front-end with fake data, without any real back-end call. This allows developing and testing the interface independently from the server.
{% end %}

---

## Back-end

### Technologies

- **Java Spring** — main REST API framework
- **MariaDB** — relational database
- **Redis** — caching and session management

### Planned features

#### User management

*Coming soon*

#### Battle management

*Coming soon*

---

## DevOps & Quality

*These tools apply to the entire project (front-end and back-end).*

| Technology | Version | Usage |
|---|---|---|
| [Compodoc](https://compodoc.app/) | ^1.2 | Front-end code documentation |
| [SonarQube](https://www.sonarsource.com/products/sonarqube/) | — | Code quality |
| [GitHub Actions](https://github.com/features/actions) | — | CI/CD |

#### Documentation

I use **Compodoc** to automatically generate technical documentation for the front-end from TypeScript code comments.

The documentation is available online: [baptouk.github.io/PokeRixe](https://baptouk.github.io/PokeRixe/)

#### Code quality with SonarQube

**SonarQube** analyzes the code on every push to detect potential bugs, bad practices and measure test coverage. Results are automatically sent to the SonarQube server via CI.

#### CI/CD via GitHub Actions

The GitHub Actions pipeline orchestrates the entire workflow on every push:

1. **Build** — compilation and project verification
2. **SonarQube analysis** — sending metrics to the server
3. **Docker build & push** — building the image and publishing to the registry
4. **Deployment** — notifying the server to pull the new image
5. **Documentation** — Compodoc generation and deployment to GitHub Pages

#### Deployment with Docker

Each project service (front, back, database, cache) runs in its own Docker container. Images are published to a dedicated **Docker registry** and pulled by the server on every update.
