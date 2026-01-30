# Pihole (DNS sinkhole)

Pi-hole deployed via Docker as the primary DNS resolver for the LAN, providing network-wide ad and tracker blocking.

## Current flow of network

Devices -> Router -> Pi-Hole -> Upstream DNS (currently google/cloudflare, future unbound)

## Requirements

Disable systemd-resolved stub resolver to free port 53

Port 53 (DNS) must be free on the host because Pi-hole container binds directly to host port 53.

In ubuntu:
```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
sudo rm /etc/resolv.conf
```

Now we must set the upstream DNS manually in the /etc/resolv.conf file as docker inherits host DNS as in the file

In /etc/resolv.conf

```bash
nameserver 1.1.1.1
nameserver 8.8.8.8
```

## Getting it started 

Create a directory in the /opt of the server, I have named it pihole-data

```bash
cd /opt
mkdir pihole-data
```

Create a docker compose file as given in this folder

```bash
cd /opt/pihole-data
touch docker-compose.yml
```

Create a data directory and two other directories for saving everything about the container

```bash
mkdir /opt/pihole-data/data
cd /opt/pihole-data/data
mkdir etc-dnsmasq.d  
mkdir etc-pihole
```

Now run it using

```bash
docker compose up -d
```

Check if running using

```bash
docker ps
```
you will see a new container running pihole with the web dashboard in localhost:8080

## Setting in Pi-hole

Go to settings -> DNS and set upstream DNS: Google by ticking IPv4 checkboxes

Then change mode to expert and change to permit all origins

## Current Blocklists:
- Steven Black
- OISD
- AdGuard
- EasyPrivacy

Reason:
Balanced blocking with low false positives for Indian ISP + OTT + banking usage.

## Debugging

**Issue**: Blocklists not downloading
Error: DNS resolution unavailable inside container

**Root Cause**:
Docker inherited systemd stub resolver

**Fix**:
Recreated resolv.conf manually
Restarted Docker

***
**Issue**: Set Pi-Hole as the primary DNS in router but ads still coming in network

**Root Cause**:
IPv6 addresses provided by the router bypassing Pi-hole entirely

**Fix**:
Disable IPv6 in devices/router
Router may not allow, so set in device

***

## Verification

```bash
docker exec -it pihole ping google.com
docker exec -it pihole pihole -g
dig google.com @PIHOLE_IP
```

You can understand if any issues are there with these commands and their results

## Future scope

* Change Upstream DNS to unbound
* Access the DNS server through tailscale and make it primary DNS for my devices

## Known Limitations

- IPv6 DNS currently disabled to avoid bypass
- Single Pi-hole instance (no redundancy)
- Depends on host DNS configuration correctness

⚠ If Pi-hole container stops, DNS resolution for the entire network may fail.
Recovery: docker restart pihole

## Quick Rebuild Checklist

1. Disable systemd-resolved
2. Fix /etc/resolv.conf
3. Start Docker
4. docker compose up -d
5. Set router DNS → Pi-hole IP
6. Run pihole -g
