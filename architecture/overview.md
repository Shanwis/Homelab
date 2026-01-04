# System Architecture Overview

This document provides a high-level architectural overview of the home labserver, its services, and how clients interact with it both locally and remotely.

The design prioritizes:
- Simplicity
- Security
- Open standards
- Incremental extensibility

## High-Level Goals

- Centralize home services on a single always-on server
- Provide reliable LAN access
- Enable secure remote access from anywhere
- Avoid exposing services directly to the public internet
- Maintain clear separation of concerns between services


## Network Architecture

### Local Network (LAN)

Clients ── SMB / IPP / SSH ──► Server (192.168.29.12)

- Samba provides file sharing
- CUPS provides printer access
- Pi-hole provides DNS services
- All services are restricted to the LAN
- No public exposure


### Remote Access (Overlay Network)

Remote Client   
│   
│ Encrypted WireGuard Tunnel    
│ (Tailscale)   
▼
Server (100.81.135.106)

* Tailscale creates a private overlay network
* Each device receives a `100.x.x.x` IP
* Traffic is end-to-end encrypted
* NAT traversal handled automatically
* No inbound ports opened on router

## Service Architecture

### Core Services

┌────────────────────────────┐  
│ Ubuntu Server │   
│────────────────────────────│  
│ Samba → NAS │  
│ CUPS → Printer │  
│ Pi-hole → DNS Filtering │    
│ OpenSSH → Admin Access │  
│ Tailscale → Remote Access │
└────────────────────────────┘

Each service runs independently and communicates over standard protocols.


## Storage Architecture

Physical Disks  
│   
├── 4TB HDD ──► /srv/nas    
│ ├── public    
│ ├── backups   
│ └── media 
│   
└── 500GB HDD ─► /srv/media 

- Disks are mounted persistently via `/etc/fstab`
- Services consume storage via filesystem paths
- No abstraction layers (LVM, ZFS) used initially
- Emphasis on predictability and transparency


## Access Patterns

### File Access
- LAN: Samba (SMB)
- Remote: SSH (`scp`, `rsync`, `sftp`)

### Printing
- LAN: Native CUPS / AirPrint / IPP
- Remote: SSH-triggered CUPS jobs

### Administration
- SSH over Tailscale
- No web admin interfaces exposed publicly


## Security Model

- No public-facing services
- No port forwarding
- Authentication via:
  - Linux users
  - SSH keys
  - Tailscale identity
- Network isolation enforced by NAT + overlay VPN
- Least-privilege access for shared users

## Design Philosophy

This system intentionally avoids:
- Proprietary NAS operating systems
- Cloud-dependent services
- Over-abstraction

Instead, it focuses on:
- Unix-native tools
- Open protocols
- Explicit configuration
- Incremental complexity


## Future Expansion

Potential future additions:
- Dockerized services (Jellyfin, dashboards)
- Monitoring (Prometheus, Grafana)
- Automated backups
- Self-hosted VPN control plane (Headscale)
- Infrastructure-as-code documentation


## Summary

The home server functions as a **secure, centralized service node** that integrates the features I need and remote access while remaining simple, auditable, and extensible.

Each architectural decision is made with long-term maintainability in mind.