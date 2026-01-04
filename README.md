# Homelab
This contains the descriptions and scripts relating to my homelab setup.

The goal of this project is to gain hands-on experience with
Linux systems, networking, remote administration, and service orchestration
by building and maintaining a real, continuously running server.


## Goals

- Centralized NAS
- Shared printer access
- Practice remote administration (SSH)
- Prepare for containerization (Docker / Kubernetes)
- Understand real-world system failures and recovery
- Learning Linux systems and networking

## Hardware description

- **CPU**: Intel i3-4130 (4) @ 3.400GHz 
- **GPU**: Intel 4th Generation Core Processor Family 
- **RAM**: 8GB
- **Storage**:
    - 4TB HDD (primary)
    - 500GB HDD (secondary)

## Operating System

- **Ubuntu Server (LTS)**

All services currently run on a single physical host.

---

## Current services

- Samba-based NAS
- CUPS + HPLIP enabled network-wide printer access
- Tailscale for remote access
- Pi-Hole server set up for network-wide ad blocking

## Planned additions

- Docker
- kubernetes (k3s)
- JellyFin for streaming movies and shows
- NextCloud for having a personal cloud
- Graphana for metrics and montioring

## Networking

- Static LAN IP configured : 192.168.29.x
- Remote access via WireGuard VPN (using tailscale)

## Status

This is an evolving project. The services are added incrementally, with an emphasis on stability, documentation, and understanding the underlying systems.

## Contributing

Any suggestion on services or strategies to improve the server are welcome :)