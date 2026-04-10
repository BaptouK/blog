+++
title = "Homelab"
description = "Infrastructure selfhost avec Proxmox, Docker et Traefik pour héberger mes services personnels."
weight = 1
date = 2026-04-09

[taxonomies]
tags = ["docker", "linux", "selfhost", "proxmox", "traefik"]

[extra]
toc = true
mermaid = true
+++

Mon homelab, c'est un vieux PC portable recyclé en serveur qui tourne 24/7 et héberge tous les services que j'utilise au quotidien. L'objectif : apprendre en faisant, garder la main sur mes données, et avoir un terrain de jeu pour expérimenter.

## Le matériel

Rien d'extravagant : un ancien laptop reconverti.

| Composant | Spec |
|-----------|------|
| Type | PC portable (recyclé) |
| RAM | 16 Go |
| Stockage | 250 Go SSD |
| OS | Proxmox VE |

Un seul serveur, pas de NAS, pas de RAID. Simple et suffisant pour mes besoins actuels.

## Architecture

Proxmox virtualise tout. L'infra se découpe en trois briques :

| VM / Conteneur | Rôle | Type |
|----------------|------|------|
| **LXC WireGuard** | Tunnel VPN pour sécuriser les accès distants | Conteneur LXC |
| **VM Webserver** | Docker + Traefik, héberge tous les services web | Machine virtuelle |
| **VM Minecraft** | Serveur Minecraft dédié | Machine virtuelle |

## Schéma réseau

{% mermaid() %}
flowchart TD
    Internet(["Internet"])
    CF(["Cloudflare DNS"])
    Router(["Box / Routeur"])

    subgraph proxmox["Proxmox VE"]
        LXC_WG["LXC WireGuard"]

        subgraph vm_web["VM Webserver"]
            Traefik["Traefik - Reverse Proxy"]
            Dockhand["Dockhand - Orchestration"]

            subgraph containers["Conteneurs Docker"]
                Blog["Site Web"]
                Vault["Vaultwarden"]
                Outline["Outline"]
                Sonar["SonarQube"]
                Autres["Autres services"]
            end
        end

        VM_MC["VM Minecraft"]
    end

    Internet -->|Requete HTTPS| CF
    CF -->|DNS resolu| Router
    Router -->|Port forwarding| Traefik
    Traefik -->|Route par domaine| containers


    Internet -->|Connexion joueurs| Router
    Router -->|Port 25565| VM_MC
    Router -->|Connexion vpn | LXC_WG

{% end %}

## La stack en détail

### Proxmox VE

[Proxmox](https://www.proxmox.com/) gère la virtualisation. Il permet d'isoler les services dans des VMs et conteneurs LXC, et de tout administrer depuis une interface web.

[WireGuard](https://www.wireguard.com/) tourne dans un **conteneur LXC** plutôt qu'une VM complète — c'est plus léger et largement suffisant pour un tunnel VPN. Il me permet d'accéder à tous mes services internes depuis n'importe où, comme si j'étais sur mon réseau local.

### Traefik — le reverse proxy

[Traefik](https://traefik.io/) est le point d'entrée de tous les services web. Il :

- **Route les requêtes** vers le bon conteneur Docker en fonction du nom de domaine
- **Gère les certificats TLS** automatiquement via Let's Encrypt
- **S'intègre nativement avec Docker** : il détecte les nouveaux conteneurs et crée les routes automatiquement grâce aux labels

Couplé à **Cloudflare DNS**, les sous-domaines pointent vers mon IP et Traefik dispatche le trafic.

### Dockhand — l'orchestration

[Dockhand](https://github.com/torn-s/dockhand) orchestre les conteneurs Docker sur la VM Webserver. Il simplifie le déploiement et la gestion de l'ensemble des services.

### Les services

| Service | Usage | 
|---------|-------|
| [**Blog / Zola**](https://www.getzola.org/) | Ce blog, hébergé en selfhost | 
| [**Vaultwarden**](https://github.com/dani-garcia/vaultwarden) | Gestionnaire de mots de passe (compatible Bitwarden) | 
| [**Outline**](https://www.getoutline.com/) | Wiki / base de connaissances |
| [**SonarQube**](https://www.sonarsource.com/fr/products/sonarqube/) | Analyse de qualité de code | 
| [**Dockhand**](https://dockhand.pro/) | Interface d'orchestration Docker |
| [**FossFlow**](https://stan-smith.github.io/FossFLOW/) | Site web de création diagramme réseau | 

## Réseau et exposition

Tout tourne sur le **même réseau local**, pas de VLAN pour l'instant.

Le flux pour accéder à un service depuis Internet :

1. L'utilisateur tape `service.domaine.tld`
2. **Cloudflare DNS** résout le domaine vers mon IP publique
3. Le **routeur** forward le port 443 vers Traefik
4. **Traefik** lit le header `Host` et route vers le bon conteneur Docker
5. Le service répond à travers la chaîne inverse

Les services internes restent accessibles uniquement depuis le réseau local ou via **WireGuard**.

## Ce qui manque (pour l'instant)

- **Backups** — Pas encore en place. Prévu : snapshots Proxmox + sauvegarde des volumes Docker
- **Monitoring** — Aucune supervision. Uptime Kuma ou Grafana/Prometheus sont possible.
- **RAID / NAS** — Un seul disque, pas de redondance. Un NAS dédié serait le bienvenu.
- **Segmentation réseau** — Tout est sur le même réseau. Des VLANs isoleraient mieux les services exposés

## Ce que ça m'apporte

Ce homelab est autant un outil du quotidien qu'un labo d'apprentissage. Je l'utilise tous les jours (mots de passe, wiki, code quality) et chaque ajout est l'occasion de creuser un sujet : réseau, sécurité, conteneurisation, CI/CD.

Le principe : **si un service peut tourner chez moi, il tourne chez moi.**
