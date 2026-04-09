+++
title = "My homelab setup"
date = 2026-03-28
description = "How I built my homelab with Docker and Proxmox."
[taxonomies]
tags = ["homelab", "docker", "linux", "selfhost"]
+++

After months of hesitation, I finally built my homelab. Here's my current setup.

## Hardware

A mini PC with 16 GB of RAM and a 500 GB SSD. Nothing crazy, but enough to get started.

## Software stack

- **Proxmox** as hypervisor
- **Docker** for application containers
- **Traefik** as reverse proxy
- **Portainer** for visual management

## Hosted services

| Service | Usage |
|---------|-------|
| Nextcloud | File storage |
| Vaultwarden | Passwords |
| Uptime Kuma | Monitoring |

The most satisfying part? No longer depending on cloud services for my personal data.
