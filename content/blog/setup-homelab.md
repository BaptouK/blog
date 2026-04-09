+++
title = "Mon setup homelab"
date = 2026-03-28
description = "Comment j'ai monté mon homelab avec Docker et Proxmox."
[taxonomies]
tags = ["homelab", "docker", "linux", "selfhost"]
+++

Après des mois à hésiter, j'ai enfin monté mon homelab. Voici mon setup actuel.

## Hardware

Un mini PC avec 16 Go de RAM et un SSD de 500 Go. Rien de fou, mais suffisant pour commencer.

## Stack logicielle

- **Proxmox** comme hyperviseur
- **Docker** pour les conteneurs applicatifs
- **Traefik** comme reverse proxy
- **Portainer** pour la gestion visuelle

## Services hébergés

| Service | Usage |
|---------|-------|
| Nextcloud | Stockage fichiers |
| Vaultwarden | Mots de passe |
| Uptime Kuma | Monitoring |

Le plus satisfaisant ? Ne plus dépendre des services cloud pour mes données personnelles.
