+++
title = "Homelab"
description = "Self-hosted infrastructure with Proxmox, Docker and Traefik to host my personal services."
weight = 1
date = 2026-04-09

[taxonomies]
tags = ["docker", "linux", "selfhost", "proxmox", "traefik"]

[extra]
toc = true
mermaid = true
+++

My homelab is an old recycled laptop running 24/7, hosting all the services I use daily. The goal: learn by doing, keep control over my data, and have a playground to experiment.

## Hardware

Nothing fancy: a repurposed old laptop.

| Component | Spec |
|-----------|------|
| Type | Laptop (recycled) |
| RAM | 16 GB |
| Storage | 250 GB SSD |
| OS | Proxmox VE |

A single server, no NAS, no RAID. Simple and sufficient for my current needs.

## Architecture

Proxmox virtualizes everything. The infrastructure is split into three building blocks:

| VM / Container | Role | Type |
|----------------|------|------|
| **LXC WireGuard** | VPN tunnel to secure remote access | LXC Container |
| **VM Webserver** | Docker + Traefik, hosts all web services | Virtual Machine |
| **VM Minecraft** | Dedicated Minecraft server | Virtual Machine |

## Network diagram

{% mermaid() %}
flowchart TD
    Internet(["Internet"])
    CF(["Cloudflare DNS"])
    Router(["Box / Router"])

    subgraph proxmox["Proxmox VE"]
        LXC_WG["LXC WireGuard"]

        subgraph vm_web["VM Webserver"]
            Traefik["Traefik - Reverse Proxy"]
            Dockhand["Dockhand - Orchestration"]

            subgraph containers["Docker Containers"]
                Blog["Website"]
                Vault["Vaultwarden"]
                Outline["Outline"]
                Sonar["SonarQube"]
                Autres["Other services"]
            end
        end

        VM_MC["VM Minecraft"]
    end

    Internet -->|HTTPS request| CF
    CF -->|DNS resolved| Router
    Router -->|Port forwarding| Traefik
    Traefik -->|Route by domain| containers


    Internet -->|Player connection| Router
    Router -->|Port 25565| VM_MC
    Router -->|VPN connection| LXC_WG

{% end %}

## Stack in detail

### Proxmox VE

[Proxmox](https://www.proxmox.com/) handles virtualization. It isolates services in VMs and LXC containers, and lets me manage everything from a web interface.

[WireGuard](https://www.wireguard.com/) runs in an **LXC container** rather than a full VM — lighter and more than enough for a VPN tunnel. It lets me access all my internal services from anywhere, as if I were on my local network.

### Traefik — the reverse proxy

[Traefik](https://traefik.io/) is the entry point for all web services. It:

- **Routes requests** to the right Docker container based on the domain name
- **Manages TLS certificates** automatically via Let's Encrypt
- **Integrates natively with Docker**: it detects new containers and creates routes automatically via labels

Combined with **Cloudflare DNS**, subdomains point to my IP and Traefik dispatches the traffic.

### Dockhand — orchestration

[Dockhand](https://github.com/torn-s/dockhand) orchestrates Docker containers on the Webserver VM. It simplifies deployment and management of all services.

### Services

| Service | Usage |
|---------|-------|
| [**Blog / Zola**](https://www.getzola.org/) | This blog, self-hosted |
| [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden) | Password manager (Bitwarden-compatible) |
| [**Outline**](https://www.getoutline.com/) | Note-taking / Online collaboration |
| [**SonarQube**](https://www.sonarsource.com/products/sonarqube/) | Code quality analysis |
| [**Dockhand**](https://dockhand.pro/) | Docker orchestration interface |
| [**FossFlow**](https://stan-smith.github.io/FossFLOW/) | Network diagram creation tool |

## Network and exposure

Everything runs on the **same local network**, no VLANs for now.

The flow to access a service from the Internet:

1. The user types `service.domain.tld`
2. **Cloudflare DNS** resolves the domain to my public IP
3. The **router** forwards port 443 to Traefik
4. **Traefik** reads the `Host` header and routes to the right Docker container
5. The service responds through the reverse chain

Internal services remain accessible only from the local network or via **WireGuard**.

## What's missing (for now)

- **Backups** — Not in place yet. Planned: Proxmox snapshots + Docker volume backups
- **Monitoring** — No supervision. Uptime Kuma or Grafana/Prometheus are options.
- **RAID / NAS** — Single disk, no redundancy. A dedicated NAS would be welcome.
- **Network segmentation** — Everything is on the same network. VLANs would better isolate exposed services.

## What I get out of it

This homelab is both a daily tool and a learning lab. I use it every day (passwords, wiki, code quality) and each addition is an opportunity to dig into a topic: networking, security, containerization, CI/CD.

The principle: **if a service can run at home, it runs at home.**
