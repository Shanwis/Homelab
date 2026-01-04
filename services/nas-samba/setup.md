# Network Attached Storage (NAS)

This document describes the setup of a **Network Attached Storage (NAS)**
service using **Samba (SMB/CIFS)** on an Ubuntu Server host.

The NAS provides shared storage for devices on the local network and
secure access from remote locations.

---

## Goals

- Centralized file storage for home network
- Cross-platform compatibility (Linux, Windows, macOS)
- Simple permission model
- Reliable disk mounting
- Secure remote access (via SSH / VPN)
- Avoid vendor lock-in

## Hardware

- Primary Storage: **4TB HDD**
- Secondary Storage: **500GB HDD**
- Connection: SATA (internal)


## Software Stack

- **Samba** – SMB/CIFS file sharing
- **Linux filesystem mounts** – Persistent disk mounting
- **OpenSSH** – Remote file access (scp / sftp)
- **Tailscale** – Secure remote network access (optional)


## Directory Layout

All NAS data is stored under `/srv/nas`.

/srv/nas    
├── public  
├── backups     
└── media

Additional disks are mounted under `/srv/media` and may be bind-mounted into the NAS structure if required.

## Disk Mounting

Disks are mounted persistently using `/etc/fstab`, as shown below:

```nginx
# /etc/fstab: static file system information.
UUID=XXXXXXXXXXXXXXXX /srv/media/part1 ntfs-3g uid=1000,gid=1000,unmask=002,nofail 0 0
UUID=XXXXXXXXXXXXXXXX /srv/media/part2 ntfs-3g uid=1000,gid=1000,unmask=002,nofail 0 0
UUID=XXXXXXXXXXXXXXXX /srv/media/part3 ntfs-3g uid=1000,gid=1000,unmask=002,nofail 0 0
/srv/media/part1 /srv/nas/media/part1 none bind,nofail 0 0
/srv/media/part2 /srv/nas/media/part2 none bind,nofail 0 0
/srv/media/part3 /srv/nas/media/part3 none bind,nofail 0 0
```


### Example mount point

/srv/media/disk1

Mounting strategy:
- Disks are mounted at boot
- Filesystems checked automatically
- Data remains accessible even if services restart

## Permissions Model

Ownership and permissions are intentionally simple.

### Ownership

```bash
sudo groupadd nasuser
sudo chown -R $USER:nasuser /srv/nas
```

### Permissions
```bash
sudo chmod -R 775 /srv/nas
```

Meaning:

* Owner: full access
* Group: read/write access
* Others: read-only (or restricted via Samba)

This aligns Linux filesystem permissions with Samba access.

## Samba Configuration Overview

Samba is configured to expose specific directories as shares.

### Key principles

* Each share maps directly to a filesystem directory. 
* Authentication uses local Linux users
* Guest access disabled (unless explicitly required)
* Permissions enforced at filesystem level

Check out the smb.conf file to see the configuration used in my setup.

## Access Methods
### Local Network (LAN)
Access via:

```bash
\\<server-ip>\NAS
```

#### Supported clients:
* Windows
* Linux
* macOS

## Remote Access (Secure)

Remote access is intentionally not exposed publicly.

Options:

* **SSH (scp, sftp, rsync)**
* **VPN-based access (Tailscale)**

Example:

```bash
scp file.txt shanwis@<tailscale-ip>:/srv/nas/public/
```

## Security Considerations

* No public SMB exposure
* Access restricted to authenticated users
* Remote access via encrypted channels only
* Filesystem permissions enforced at OS level

## Features which can be added
* Backups must be made
* More consideration into how the files are to be shared

## Notes
This NAS setup favors:
* Simplicity
* Predictability
* Unix-native tooling

It is intentionally conservative to avoid data loss
and permission complexity.