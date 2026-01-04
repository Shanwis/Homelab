# Printer Server (CUPS)

This document describes the setup of a network-accessible printer server
using **CUPS (Common UNIX Printing System)** on an Ubuntu Server host.

The printer is connected via USB and shared across the local network
and remotely via secure access mechanisms.

---

## Goals

- Centralize printer access
- Avoid installing printer drivers on every client
- Enable printing from:
  - Local LAN devices
  - Remote devices via SSH
- Keep the setup simple, stable, and secure

---

## Hardware

- Printer: **HP Inkjet 310 series**
- Connection: USB (connected directly to server)

---

## Software Stack

- **CUPS** – Print server and job manager
- **HPLIP** – HP printer drivers and utilities
- **OpenSSH** – Remote print triggering

---

## Architecture Overview

Client (LAN / Remote)   
        |   
        | (SSH / IPP)   
        |   
        v   
Ubuntu Server   
├── CUPS (port 631)     
├── HPLIP drivers   
└── USB-connected printer

The server acts as a **print proxy**:
clients send documents, and CUPS handles
spooling, driver processing, and printing.

---

## Installation Summary

### Install CUPS

```bash
sudo apt update
sudo apt install -y cups
```

Enable and start the service:

```bash
sudo systemctl enable cups
sudo systemctl start cups
```

Install HP Drivers (HPLIP)
```bash
sudo apt install -y hplip hplip-gui
```

Verify detection:

```bash
hp-info
```

### User Permissions

Allow the primary user to manage printers:

```bash
sudo usermod -aG lpadmin $USER
```

(Log out and back in for changes to apply.)

### Network Access Configuration

CUPS is configured to allow access from the local network.

Key changes in /etc/cups/cupsd.conf:

* **Listen on all interfaces**:

    Change Listen 631 to Port 631

* **Allow access from local subnet**:

    Add Allow @LOCAL like 

    ```nginx
    <Location />
    # Allow shared printing...
    Order allow,deny
    Allow @LOCAL
    </Location>
    ```
    For /, /admin/conf and /admin/log

    After changes:

    ```bash
    sudo systemctl restart cups
    ```
#### Accessing the CUPS Interface

From any device on the LAN:

```ngnix
Copy code
http://<server-ip>:631
```

This web interface is used to:

* Add printers
* Manage queues
* View jobs
* Run test prints

#### Remote Printing via SSH
For remote use (e.g., from college), printing is done
by triggering CUPS over SSH.

Example: One-line remote print

```bash
ssh shanwis@<tailscale-ip> "lp -d HP_Inkjet_310" < document.pdf
```

This approach:
* Avoids exposing CUPS to the internet

* Requires no printer setup on clients

* Works from anywhere SSH access is available

## Security Considerations
* No public ports exposed

* Remote access via encrypted SSH

* Printer accessible only to authenticated users

* LAN access limited to trusted subnet

## Features which could be added:

* AirPrint over VPN, but it seems unnecessary
* Print job automation when file added to a folder

## Notes
This setup follows traditional Unix printing practices
and is intentionally conservative to ensure reliability.

