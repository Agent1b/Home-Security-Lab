# 🏠 Home Network Security Lab

> A fully segmented, monitored home network built from scratch for cybersecurity research, threat detection practice, and hands-on learning.

---

## 🎯 Purpose

This lab serves as my personal training ground for:
- Practicing network security monitoring and threat detection
- Testing security tools and configurations in a real environment
- Learning firewall management, network segmentation, and traffic analysis
- Running CTF challenges and malware research in isolated segments
- Developing and testing my own security tools safely

---

## 🏗️ Network Architecture

```
                          ┌─────────────┐
                          │   Internet   │
                          └──────┬──────┘
                                 │
                          ┌──────▼──────┐
                          │  ISP Router  │
                          └──────┬──────┘
                                 │  192.168.x.0/24
                     ┌───────────▼───────────┐
                     │   Raspberry Pi 4 (4GB) │
                     │   Pi-hole + Unbound    │
                     │   DNS Filtering/Privacy│
                     │   192.168.x.2          │
                     └───────────┬───────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              │                  │                  │
   ┌──────────▼──────┐  ┌────────▼────────┐  ┌─────▼──────────┐
   │  Ubuntu Server   │  │  Work PC        │  │  Other Devices  │
   │  192.168.x.x     │  │  Windows        │  │  Phones, etc.   │
   │  Nextcloud+Samba │  │  Samba client   │  │  Pi-hole DNS    │
   └──────────────────┘  └─────────────────┘  └────────────────┘
                                 │
                          ┌──────▼──────┐
                          │  Tailscale   │
                          │  VPN Mesh    │
                          │  Remote Access│
                          └─────────────┘
```

---

## 🖥️ Hardware Inventory

| Device | Role | Specs | OS |
|--------|------|-------|----|
| Raspberry Pi 4 | DNS Filtering / Network Security | 4GB RAM, 64GB SD | Raspberry Pi OS 64-bit (Trixie) |
| Custom Home Server | Cloud Storage / File Server | 238GB SSD (OS) + 931GB HDD (Storage) | Ubuntu Server 24.04 LTS |
| Work PC | Daily driver + security research | Windows 11 | Windows 11 |

---

## 🔥 Firewall Configuration

**Firewall Software:** UFW (Uncomplicated Firewall) on both Ubuntu Server and Raspberry Pi

### Network Layout

| Device | IP | Subnet | Role |
|--------|-----|--------|------|
| ISP Router | 192.168.x.1 | 192.168.x.0/24 | Gateway |
| Raspberry Pi | 192.168.x.2 | 192.168.x.0/24 | DNS + Security |
| Ubuntu Server | 192.168.x.x | 192.168.x.0/24 | Storage + Services |

### Ubuntu Server Firewall Rules

| Port | Protocol | Source | Service |
|------|----------|--------|---------|
| SSH | TCP | Local network only | SSH (local only) |
| HTTP | TCP | Local network only | Nextcloud web |
| SMB | TCP | Local network only | Samba file sharing |
| SMB | TCP | Local network only | Samba file sharing |
| Admin | TCP | Local network only | Cockpit admin panel |
| All | All | Tailscale interface | Tailscale VPN |

### Raspberry Pi Firewall Rules

| Port | Protocol | Source | Service |
|------|----------|--------|---------|
| SSH | TCP | Local network only | SSH (local only) |
| DNS | TCP/UDP | Local network only | DNS (Pi-hole) |
| HTTP | TCP | Local network only | Pi-hole web admin |
| Admin | TCP | Local network only | Cockpit admin panel |
| All | All | Tailscale interface | Tailscale VPN |

### Key Security Decisions

- All services locked to local network only — zero internet exposure
- No open ports on ISP router (no port forwarding)
- Remote access exclusively via Tailscale encrypted mesh VPN
- SSH access restricted to local subnet on both machines
- Samba share requires authenticated user (no guest access)

---

## 📊 Monitoring & Detection

### Tools Deployed

| Tool | Purpose | Where It Runs |
|------|---------|---------------|
| Pi-hole | DNS-level ad/malware/tracker blocking | Raspberry Pi 4 |
| Unbound | Private recursive DNS resolver (no upstream dependency) | Raspberry Pi 4 |
| Cockpit | System monitoring, logs, service management | Ubuntu Server + Pi |
| FileBrowser | Web-based file manager for Pi filesystem | Raspberry Pi 4 |
| Fail2ban | Brute force & intrusion prevention | Ubuntu Server |
| PSAD | Port scan detection via UFW logs | Ubuntu Server |
| CrowdSec | Community-driven threat intelligence & IP blocking | Ubuntu Server |
| Tailscale | Zero-trust encrypted VPN mesh | Both machines |

### DNS Security (Pi-hole + Unbound)

Pi-hole acts as the network-wide DNS resolver with three blocklists active:

| Blocklist | Purpose | Domains Blocked |
|-----------|---------|-----------------|
| StevenBlack/hosts | Ads, trackers | ~82,000 |
| hagezi/pro.txt | Comprehensive ads, tracking, telemetry | ~500,000 |
| hagezi/tif.txt | Malware, botnets, phishing, threat intelligence | ~200,000 |

**Unbound** resolves DNS directly against root servers — no queries sent to ISP, Google, or Cloudflare. Full DNS privacy for every device on the network.

### What I Monitor

- All DNS queries across the network via Pi-hole dashboard
- Failed SSH authentication attempts (Fail2ban)
- Port scan attempts (PSAD + UFW logging)
- Known malicious IPs (CrowdSec community intelligence)
- System resources, services, and logs (Cockpit)

---

## 💾 Storage Server

**Hardware:** Custom PC — 238GB SSD (OS drive) + 931GB HDD (storage drive)
**OS:** Ubuntu Server 24.04 LTS (bare metal, no Docker)
**Storage:** 931GB HDD mounted at `/mnt/storage`, partitioned as:
- `/mnt/storage/nextcloud-data` — Nextcloud user data
- `/mnt/storage/workshare` — Samba network share

### Services Running

| Service | Port | Purpose |
|---------|------|---------|
| Nextcloud | HTTP | Self-hosted Google Drive alternative |
| Samba | SMB | Windows network drive (mapped as Z:\) |
| Cockpit | Admin port | Web-based server management |
| Apache2 | HTTP | Web server for Nextcloud |
| MariaDB | Local only | Nextcloud database |

### Storage Strategy

- **Nextcloud** for personal cloud storage, photo backup, file sync
- **Samba share** (100GB) mapped as network drive on Windows work PC
- OS and data on separate physical drives for clean separation
- Static IP configured via Netplan for reliable local access

---

## 🔒 Remote Access — Tailscale

Instead of exposing ports to the internet, the entire lab uses **Tailscale** for secure remote access:

- Zero port forwarding on ISP router
- Encrypted WireGuard tunnels between all devices
- Access Nextcloud, Cockpit, Pi-hole admin from anywhere in the world
- Works on Windows, Linux, Android, iOS

---

## 🧠 Skills Demonstrated

This home lab project demonstrates hands-on experience with:

- **Network Engineering:** Static IP configuration, DNS architecture, network-wide filtering
- **Linux Administration:** Ubuntu Server + Raspberry Pi OS, service configuration, UFW firewall, Netplan, fstab, systemd
- **Security:** Intrusion detection (Fail2ban, PSAD, CrowdSec), DNS-level threat blocking, zero-exposure network design
- **Self-Hosted Services:** Nextcloud (bare metal), Samba, Pi-hole, Unbound, Cockpit, FileBrowser
- **Privacy:** Private recursive DNS with Unbound, no upstream DNS dependency, VPN-only remote access
- **System Administration:** Multi-drive storage management, PHP tuning, Apache configuration, MariaDB

---

## 📈 Roadmap

- [x] Ubuntu Server with Nextcloud (bare metal)
- [x] Samba network share for Windows
- [x] Pi-hole + Unbound private DNS
- [x] UFW firewall on both machines
- [x] Tailscale remote access
- [x] Fail2ban + PSAD + CrowdSec intrusion detection
- [x] Cockpit server management on both machines
- [ ] Homepage dashboard (central app launcher)
- [ ] Automated Nextcloud backups (rsync/borgbackup)
- [ ] Grafana + Prometheus monitoring dashboards
- [ ] Wazuh SIEM for centralized alerting
- [ ] Jellyfin media server
- [ ] Add dedicated router (TP-Link/GL.iNet) for VLAN segmentation
- [ ] Honeypot on guest network

---

## 🔗 Related Projects

- [Custom RAT — PoC Research](link-to-rat-repo) — Offensive security research conducted in this lab
- [Custom Payload Research](link-to-payload-repo) — Payload development and testing

---

## 📬 Contact

**Michael Baazov**  
[LinkedIn](https://www.linkedin.com/in/michael-baazov-87417823b/) | [GitHub](https://github.com/Agent1b) | michbz@proton.me

---

*This lab is continuously evolving. I believe the best way to learn cybersecurity is by building, breaking, and defending real systems.*
