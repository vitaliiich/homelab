# VPS

This is the only machine in this repo not physically in the apartment. So far it is only being used to expose my apps hosted on my local infrastructure to the Internet, without punching any holes (ports) in my firewall (OPNsense) through NetBird Reverse Proxy feature. Moreover, my current ISP uses a **CGNAT** technology, which makes exposing any services to the public internet even more complicated, as i do not "own" any reliable single IP address for a somewhat-prolonged period of time.

---

## Host


| Parameter | Name |
|---|---|
| Provider | Hetzner |
| Region | Falkenstein, DE |
| Plan | CX23 |
| vCPU / RAM / disk | 2 vCPU/4 GB/40 GB NVME |
| OS | Debian 13 |
| Monthly cost | €3.77 |


---

## What runs here


| Service | Purpose | Detail |
|---------|---------|--------|
| NetBird control plane | Self-hosted NetBird admin portal for my NetBird network | [netbird/](netbird/README.md) |


---

## Hardening

Baseline applied to the host:
- **SSH:** pubkey auth only, root login disabled, password auth off
- **Firewall:** `ufw`, default deny inbound, only the ports listed below opened
- **fail2ban:** enabled for `ssh`, *maxretry = 5*, *bantime = 1h*
- **Updates:** `unattended-upgrades` for auto-installing security patches, *Automatic-Reboot-Time* set to daily at 04:00
- **User:** single non-root user with `sudo`

### Open ports


| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | any | SSH |
| 80 | TCP | any | HTTP |
| 443 | TCP | any | HTTPS |
| 3478 | UDP | any | STUN/TURN for NAT |


---

## Provisioning

Currently configured manually.

---

## Runbooks

- [Update the NetBird control plane](../../docs/runbooks/netbird/update-control-plane.md)
- [Expose a homelab service over the overlay](../../docs/runbooks/netbird/expose-a-service.md)
- [Enroll a new peer](../../docs/runbooks/netbird/enroll-a-peer.md)

---

## TODO

- [ ] Ansible role for base hardening
- [ ] Ansible role for the NetBird deployment
- [ ] Define a Backup strategy
