# Remote Access (Tailscale)

This document describes the remote access setup for the home server using **Tailscale**, a peer-to-peer encrypted networking solution built on WireGuard.

The goal is to securely access the server and its services from anywhere without exposing ports to the public internet.

## Goals

- Secure remote access from outside the home network
- No port forwarding on the router
- Minimal maintenance
- Device-based access control
- Support for SSH, file access, and service management
- Safe access for multiple users (family)

## Why Tailscale

Traditional approaches (port forwarding, public SSH) were avoided due to security risks and maintenance overhead.

Tailscale was chosen because it provides:

- End-to-end encrypted connections
- Automatic NAT traversal
- Peer-to-peer networking using WireGuard
- Identity-based access control
- Works behind home routers without configuration

## Conceptual Overview

Tailscale creates a **private overlay network** (called a **tailnet**) between authorized devices.

Laptop / Phone (remote)     
|   
| Encrypted WireGuard tunnel    
|   
v   
Home Server

Each device receives a private IP address in the `100.x.x.x` range, which is only reachable by other authorized devices.


## Installation Summary

### Install on Server (Ubuntu)

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

After login, the server receives a Tailscale IP:

Example: 100.81.135.106

Install on Client Devices:
* Laptop (college)
* Phone
* Optional: family devices

Each device logs in using its own account.

### SSH Access
Once connected to Tailscale, the server is reachable from anywhere via SSH:

```bash
ssh shanwis@100.81.135.106
```
Optionally, Tailscale-managed SSH can be enabled:

```bash
sudo tailscale up --ssh
```

This allows SSH authentication using Tailscale identity instead of passwords.

## Remote Service Usage
### NAS Access
Files accessed via SSH (scp, rsync, sftp)

Samba access possible through either smbclient or directly through files applications

Example:
```
smbclient //100.x.x.x/NAS -U shanwis
```

or 

```
smb://100.x.x.x/NAS
```

### Printer Access
Remote printing via SSH-triggered CUPS commands

Example:
```bash
ssh shanwis@100.81.135.106 "lp -d HP_Inkjet_310" < document.pdf
```

### Administration
Full server management via SSH
* No services exposed publicly

## Multi-User Access (Sharing)
Family members can access the server without sharing accounts.

* Each user has their own Tailscale account
* The server device is explicitly shared
* Access can be revoked at any time

This ensures:

* Clear access boundaries
* No shared credentials
* Easy management

## Security Model
* No inbound ports opened on router
* All traffic encrypted end-to-end
* Access restricted to authorized devices
* Device-level visibility and revocation
* Server is invisible to the public internet

## Features to be added in the future
* Make use of WireGaurd itself or Headscale to go fully open source

## Notes
Tailscale provides a clean abstraction for secure remote access while maintaining a strong security posture. I will try to migrate to headscale or use just the WireGuard VPN in the future to make it more open source inclined
