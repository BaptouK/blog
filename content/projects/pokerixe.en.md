+++
title = "PokeRixe"
description = "Architecture and implementation of a real-time Pokémon battle server with AI."
weight = 2
[taxonomies]
tags = ["java", "spring-boot", "websocket", "mongodb", "jwt", "ollama", "docker", "prometheus", "grafana", "devops"]

[extra]
toc = true
mermaid = true
+++

## Overview

PokeRixe is a **real-time turn-based Pokémon battle game**. The backend orchestrates battles via WebSocket, manages authentication, enriches data with PokéAPI, and provides AI analysis of matches.

**Link to the site:** [pokerixe.baptouk.live](https://pokerixe.baptouk.live)

**Source code**: [github.com/orgs/Pokerixe/repositories](https://github.com/orgs/Pokerixe/repositories)


---

## Technical Stack

| Layer | Technology |
|---|---|
| **Language** | Java 21 |
| **Framework** | Spring Boot 4.0 |
| **Database** | MongoDB 7 |
| **Real-time** | WebSocket + MessagePack |
| **Authentication** | Spring Security + JWT (HttpOnly cookies) |
| **AI** | Spring AI + Ollama |
| **External API** | PokéAPI |
| **CI/CD** | GitHub Actions + GHCR |
| **Containerization** | Docker |
| **Monitoring** | Prometheus, Grafana, Loki, Node Exporter, cAdvisor |

---

## General Architecture

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

## Real-time Battles via WebSocket

### General Concept

Battles operate in **synchronized turn-by-turn** fashion:

1. **Each player sends their action** (attack + target) via WebSocket
2. **The server waits for both actions** before calculating
3. **Once both are received**, the server executes the battle logic
4. **Result is sent to both clients** with the updated state

### Handling Special Turns

The main challenge was managing **transitions between Pokémon**:

- When a Pokémon faints, the owner must **select the next one** (switch action)
- **The other player waits** in the current turn — their previous action is preserved
- Once the switch completes, the turn resumes with the previously waiting player acting first (new Pokémon has no action in this turn)

**Result:** A state machine on the server that handles multiple scenarios:
- Normal turn (2 simultaneous actions)
- Switch turn (1 action switches Pokémon, 1 action waits)
- Recovery turn (after switch, action order changes)

### Damage Calculation

Damage applies classic Pokémon mechanics:

- **Offensive stat**: Attack or Special Attack (depending on move type)
- **Defensive stat**: Defense or Special Defense (depending on move type)
- **STAB** (Same Type Attack Bonus): +50% if the move type matches the Pokémon's type
- **Type effectiveness**: ×2 or ×0.5 based on type matchup (PokéAPI data)
- **Simplified formula**: `(((2/5 * Level + 2) * Power * StatOff / StatDef) / 50) + 2) * STAB * SuperEff`

---

## Authentication and Security

### JWT with HttpOnly Cookies

- **Sign-up/Login**: Spring Security + credential validation
- **JWT generation**: Token stored in **HttpOnly cookie** (not accessible to JavaScript)
- **Validation per request**: Spring Security filters tokens via a `JwtAuthenticationFilter`
- **Refresh tokens**: Periodic rotation to limit token lifetime

**Advantage**: Protection against XSS (token inaccessible to JS) while preserving HTTP independence.

---

## PokéAPI Integration

### Data Retrieval

- **Spring WebClient**: Reactive, non-blocking HTTP client
- **Application-side cache**: Pokémon data (stats, types, moves) are cached in-memory after first call
- **Enriched DB data**: Teams store the Pokémon ID; complete stats are fetched once and reused

### Performance Consideration

PokéAPI can be slow. Eventually, a distributed cache (Redis) or periodic syncing would be ideal, but in MVP it's handled.

---

## AI Analysis with Ollama

### Post-battle Analysis

At the end of each battle, the backend calls **Ollama** (local LLM model) to analyze the match:

```
Input → Turn history (Pokémon, moves, damage, type matchups)
Process → Generic prompt + battle context
Output → Overall score (1-10) + per-turn scores + strategic advice
```

### Data Sovereignty

**Key choice**: Ollama is self-hosted on the infrastructure. No cloud API calls → battle data stays private.

**MVP trade-off**: The prompt is basic (no fine-tuning), but sufficient for relevant feedback.

---

## Orchestration with Docker Compose

The project runs on **two separate docker-compose files**:

### `docker-compose.yml` (Application)

```yaml
Services:
- mongo-pokerixe      → MongoDB (battle data, users, teams)
- mongodb-exporter    → Expose MongoDB metrics to Prometheus
- backend-pokerixe    → Spring Boot (API + WebSocket)
- frontend-pokerixe   → Angular (static files)
- nginx-pokerixe      → Internal reverse proxy + Traefik labels
```

**Networks**:
- `http`: Exposed to Traefik (public HTTPS)
- `pokerixe-bridge`: Internal (backend ↔ frontend, nginx ↔ app)
- `default`: General communication

### `monitoring/docker-compose.yml` (Observability)

```yaml
Services:
- prometheus          → Scrape metrics (Spring Boot, MongoDB, Node, cAdvisor)
- node-exporter       → Host CPU, memory, disk
- grafana             → Dashboards
- loki                → Log storage
- promtail            → Log collector (from Docker containers)
- cadvisor            → Container metrics
```

**Interaction**: Prometheus scrapes all sources. Grafana consumes Prometheus (metrics) + Loki (logs).

---

## Monitoring and Observability

### Prometheus

Scrapes endpoints exposed by:
- **Spring Boot Actuator**: `/actuator/prometheus` (heap, GC, HTTP requests, active WebSocket connections)
- **MongoDB Exporter**: `/metrics` (documents, indexes, operations)
- **Node Exporter**: System metrics (CPU, RAM, disk, network)
- **cAdvisor**: Resource consumption per Docker container

### Grafana

Dashboards:
- **Spring Application**: Latency, error rates, heap memory, active connections
- **MongoDB**: Operations/sec, connections, collection sizes
- **Infrastructure**: CPU, memory, disk I/O, network
- **Logs**: HTTP requests, errors, WebSocket events (via Loki)

### Loki + Promtail

- **Promtail**: Collector that scrapes JSON logs from Docker containers
- **Loki**: Log storage optimized for high volume (compression, label indexing)
- **Grafana integration**: Logs queryable directly in dashboards

**Advantage**: Easy correlation between metrics (Prometheus) and logs (Loki).

---

## Deployment and CI/CD

### GitHub Actions

On each push to `main`:

1. **Build** → Project compilation, tests (when implemented 😅)
2. **SonarQube Analysis** → Bug detection, code smells
3. **Docker Build** → Backend image creation
4. **Push GHCR** → Publish to GitHub Container Registry
5. **Deployment** → Signal server to fetch new image
6. **Restart** → `docker-compose pull && docker-compose up -d`

### Security

- **Traefik + Cloudflare**: HTTPS with Let's Encrypt certificates (DNS-01 wildcard)
- **JWT HttpOnly**: Secure client-side sessions
- **Spring Security**: Filtering of unauthenticated requests

---

## Key Learnings

### 1. WebSocket and Turn Logic

The biggest complexity was **synchronizing both players** while managing transitions to fainted Pokémon. Initially, I had a simple 2-state machine (wait for 1 action, wait for 2 actions), but it quickly became complicated with Pokémon switches.

**Solution**: Shifted to a model where each player has independent state (`waitingForAction`, `switched`, `ready`), with transition logic dependent on turn context.

### 2. Ollama Cache and Spring AI

Ollama can be slow if the model isn't in GPU memory. I learned the importance of **profiling response times** and calling AI **after battles** rather than in real-time.

### 3. Docker Networking

Traefik ↔ Nginx ↔ Spring Boot is 3 layers of reverse proxy. Good news: it scales. Bad news: **debugging HTTP headers** becomes trickier. Hence the importance of structured logging.

### 4. IntelliJ IDEA and Corrupted Caches

Multiple times, IntelliJ refused to compile for no apparent reason. **Solution**: `File > Invalidate Caches > Restart`. Small win of the day...

### 5. Lack of Tests

Due to time constraints, the project has **no unit or integration tests**. It's a weakness for a production project (even MVP). To integrate into future workflow.

---

## Conclusion

PokeRixe backend is a **fullstack project touching many domains**: business logic (Pokémon battles), real-time (WebSocket), security (JWT), AI (Ollama), and observability (Prometheus/Grafana).

The main learning: **well-architected systems require thinking beyond code** — networking, monitoring, deployment, secret management. This took the most time, and it's what makes the project "ready for real life".

**Source code**: [github.com/orgs/Pokerixe/repositories](https://github.com/orgs/Pokerixe/repositories)