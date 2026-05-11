+++
title = "PokeRixe — Backend"
description = "Architecture et implémentation du serveur de combats Pokémon en temps réel avec IA."
weight = 2
[taxonomies]
tags = ["java", "spring-boot", "websocket", "mongodb", "jwt", "ollama", "docker", "prometheus", "grafana", "devops"]

[extra]
toc = true
mermaid = true
+++

## Présentation

PokeRixe est un jeu de combats Pokémon **tour par tour en temps réel**. Le backend orchestre les affrontements via WebSocket, gère l'authentification, enrichit les données avec PokéAPI, et propose une analyse IA des parties.

**Lien vers le site :** [pokerixe.baptou.live ](https://pokerixe.baptouk.live)

---

## Stack technique

| Couche | Technologie |
|---|---|
| **Langage** | Java 21 |
| **Framework** | Spring Boot 4.0 |
| **Base de données** | MongoDB 7 |
| **Temps réel** | WebSocket + MessagePack |
| **Authentification** | Spring Security + JWT (cookies HttpOnly) |
| **IA** | Spring AI + Ollama |
| **API externe** | PokéAPI |
| **CI/CD** | GitHub Actions + GHCR |
| **Conteneurisation** | Docker |
| **Monitoring** | Prometheus, Grafana, Loki, Node Exporter, cAdvisor |

---

## Architecture générale

{% mermaid() %}
    graph TB
        Client["Client Angular<br/>(pokerixe.baptouk.live)"]
        Traefik["Traefik<br/>(Reverse Proxy + TLS)"]
        Nginx["Nginx<br/>(Load balancing interne)"]
        
        subgraph "Backend"
            Spring["Spring Boot 4.0<br/>(API REST + WebSocket)"]
            Security["Spring Security<br/>+ JWT"]
            PokeAPI["PokéAPI Client<br/>(Spring WebClient)"]
        end
        
        subgraph "Data & Storage"
            Mongo["MongoDB 7<br/>(Combats, Utilisateurs, Équipes)"]
            MongoExp["MongoDB Exporter"]
        end
        
        subgraph "IA"
            SpringAI["Spring AI<br/>(Abstraction)"]
            Ollama["Ollama<br/>(Analyse locale)"]
        end
        
        Client -->|HTTPS| Traefik
        Traefik --> Nginx
        Nginx -->|REST + WebSocket| Spring
        Spring -->|Auth| Security
        Spring -->|Query| Mongo
        Spring -->|HTTP| PokeAPI
        Spring -->|Analyze| SpringAI
        SpringAI -->|Request| Ollama
        
        Mongo --> MongoExp
        
{% end %}


---

## Combats en temps réel via WebSocket

### Concept général

Les combats fonctionnent en **tour par tour synchronisé** :

1. **Chaque joueur envoie son action** (attaque + cible) via WebSocket
2. **Le serveur attend les deux actions** avant de calculer
3. **Une fois les deux reçues**, le serveur exécute la logique de combat
4. **Résultat envoyé aux deux clients** avec l'état actualisé

### Gestion des tours spéciaux

Le défi principal a été de gérer les **transitions entre Pokémon** :

- Quand un Pokémon meurt, le joueur propriétaire doit **choisir le prochain** (action "switch")
- **L'autre joueur attend** dans le tour courant — son action précédente n'est pas perdue
- Une fois le switch effectué, le tour reprend avec l'ancien joueur qui agit d'abord (nouveau Pokémon dépourvu d'action dans ce tour)

**Résultat :** Une machine à états côté serveur qui gère plusieurs scénarios :
- Tour normal (2 actions simultanées)
- Tour switch (1 action change de Pokémon, 1 action attend)
- Tour recovery (après switch, action ordre change)

### Calcul des dégâts

Les dégâts appliquent les mécaniques Pokémon classiques :

- **Stat offensif** : Attaque ou Attaque Spé (selon le type d'attaque)
- **Stat défensif** : Défense ou Défense Spé (selon le type d'attaque)
- **STAB** (Same Type Attack Bonus) : +50% si le type de l'attaque match le type du Pokémon
- **Super efficacité** : ×2 ou ×0.5 selon le matchup type (données PokéAPI)
- **Formule simplifiée** : `(((2/5 * Level + 2) * Power * StatOff / StatDef) / 50) + 2) * STAB * SuperEff`

---

## Authentification et sécurité

### JWT avec HttpOnly cookies

- **Inscription/Connexion** : Spring Security + validation des identifiants
- **Génération JWT** : Token stocké en **HttpOnly cookie** (non accessible au JavaScript)
- **Validation à chaque request** : Spring Security filtre les tokens via un `JwtAuthenticationFilter`
- **Refresh tokens** : Rotation périodique pour limiter la durée de vie

**Avantage :** Sécurité contre XSS (token inaccessible au JS) tout en préservant la décorellation HTTP.

---

## Intégration PokéAPI

### Récupération des données

- **WebClient Spring** : Client HTTP réactif et non-bloquant
- **Cache côté application** : Les données Pokémon (stats, types, attaques) sont cachées en mémoire après premier appel
- **Données enrichies en DB** : Les équipes stockent l'ID du Pokémon; les stats complètes sont récupérées une seule fois et réutilisées

### Considération perf

PokéAPI peut être lente. À terme, un cache distributé (Redis) ou une synchro périodique serait idéale.

---

## Analyse IA avec Ollama

### Post-combat analysis

À la fin de chaque combat, le backend appelle **Ollama** (modèle LLM local) pour analyser la partie :

```
Input → Historique des tours (Pokémon, attaques, dégâts, type matchups)
Process → Prompt générique + contexte du combat
Output → Score global (1-10) + scores par tour + conseil stratégique
```

### Souveraineté des données

**Choix clé :** Ollama est auto-hébergé sur l'infrastructure. Aucun appel à une API cloud → les données de combat restent privées.

**Trade-off MVP** : Le prompt est basique (pas de fine-tuning), mais suffit pour des retours pertinents.

---

## Orchestration avec Docker Compose

Le projet tourne sur **deux docker-compose distincts** :

### `docker-compose.yml` (Application)

```yaml
Services:
- mongo-pokerixe      → MongoDB (données combats, users, équipes)
- mongodb-exporter    → Exposition des métriques MongoDB à Prometheus
- backend-pokerixe    → Spring Boot (API + WebSocket)
- frontend-pokerixe   → Angular (static files)
- nginx-pokerixe      → Reverse proxy interne + Traefik labels
```

**Réseaux** :
- `http` : Exposé à Traefik (HTTPS public)
- `pokerixe-bridge` : Interne (backend ↔ frontend, nginx ↔ app)
- `default` : Communication générale

### `monitoring/docker-compose.yml` (Observabilité)

```yaml
Services:
- prometheus          → Scrape des métriques (Spring Boot, MongoDB, Node, cAdvisor)
- node-exporter       → CPU, mémoire, disque du host
- grafana             → Tableaux de bord
- loki                → Stockage des logs
- promtail            → Collecteur de logs (depuis les conteneurs Docker)
- cadvisor            → Métriques des conteneurs
```

**Interaction** : Prometheus scrape toutes les sources. Grafana consomme Prometheus (métriques) + Loki (logs).

---

## Monitoring et observabilité

### Prometheus

Scrape les endpoints exposés par :
- **Spring Boot Actuator** : `/actuator/prometheus` (heap, GC, requêtes HTTP, WebSocket actifs)
- **MongoDB Exporter** : `/metrics` (documents, index, opérations)
- **Node Exporter** : Métriques système (CPU, RAM, disque, network)
- **cAdvisor** : Consommation de ressources par conteneur Docker

### Grafana

Dashboards :
- **Application Spring** : Latency, error rates, heap memory, active connections
- **MongoDB** : Opérations/sec, connexions, taille des collections
- **Infrastructure** : CPU, memory, disk I/O, network
- **Logs** : Requêtes HTTP, erreurs, WebSocket events (via Loki)

### Loki + Promtail

- **Promtail** : Collector qui scrape les logs JSON des conteneurs Docker
- **Loki** : Base de données optimisée pour les logs volumineux (compression, indexation par labels)
- **Intégration Grafana** : Logs queryables directement en dashboard

**Avantage** : Corrélation facile entre métriques (Prometheus) et logs (Loki).

---

## Déploiement et CI/CD

### GitHub Actions

À chaque push sur `main` :

1. **Build** → Compilation du projet, tests (quand implémenter 😅)
2. **Analyse SonarQube** → Détection de bugs, code smells
3. **Build Docker** → Construction de l'image backend
4. **Push GHCR** → Publication sur GitHub Container Registry
5. **Déploiement** → Signal au serveur pour récupérer la nouvelle image
6. **Redémarrage** → `docker-compose pull && docker-compose up -d`

### Sécurité

- **Traefik + Cloudflare** : HTTPS avec certificats Let's Encrypt (DNS-01 wildcard)
- **JWT HttpOnly** : Sessions sécurisées côté client
- **Spring Security** : Filtrage des requêtes non authentifiées

---

## Apprentissages clés

### 1. WebSocket et logique de tour

La plus grosse complexité a été de **synchroniser les deux joueurs** en gérant les transitions vers des Pokémon morts. Initiallement, j'avais une simple machine à états 2 états (attendre 1 action, attendre 2 actions), mais ça s'est vite compliqué avec les switch Pokémon.

**Solution** : Passage à un modèle où chaque joueur a un état indépendant (`waitingForAction`, `switched`, `ready`), avec une logique de transition qui dépend du contexte du tour.

### 2. Cache Ollama et Spring AI

Ollama peut être lent si le modèle n'est pas en mémoire GPU. J'ai appris l'importance de **profiler les temps de réponse** et d'appeler l'IA **après le combat** plutôt qu'en temps réel.

### 3. Docker networking

Traefik ↔ Nginx ↔ Spring Boot, c'est 3 couches de reverse proxy. Bonne nouvelle : ça scale. Mauvaise nouvelle : **debug des headers HTTP** devient trickier. D'où l'importance des logs structurés.

### 4. IntelliJ IDEA et caches corrompus

Plusieurs fois, IntelliJ refusait de compiler sans raison apparente. **Solution** : `File > Invalidate Caches > Restart`. Petite victoire du jour...

### 5. Absence de tests

Faute de temps, le projet n'a **pas de tests unitaires ni d'intégration**. C'est un point faible pour un projet en prod (même en MVP). À intégrer dans le workflow futur.

---

## Conclusion

PokeRixe backend est un projet **fullstack qui touche à beaucoup de domaines** : logique métier (combats Pokémon), temps réel (WebSocket), sécurité (JWT), IA (Ollama), et observabilité (Prometheus/Grafana).

L'apprentissage principal : **bien architected systems demandent de penser au-delà du code** — réseaux, monitoring, déploiement, gestion des secrets. C'est ce qui a pris le plus de temps, et c'est aussi ce qui rend le projet "prêt pour la vraie vie".

**Code source** : [github.com/orgs/Pokerixe/repositories](https://github.com/orgs/Pokerixe/repositories)