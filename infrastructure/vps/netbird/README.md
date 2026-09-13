# NetBird control plane

Self-hosted NetBird management, signal, relay and dashboard, running on the [Hetzner VPS](../README.md).

---

## Components


| Component | Purpose |
|-----------|---------|
| **traefik** | Traefik Reverse Proxy |
| **dashboard** | UI Dashboard for Control Plane |
| **netbird-server** | Combo of Management + Signal + Relay + STUN |
| **proxy** | Expose internal services to the internet |
| **crowdsec** | Optional security plugin |


## Deployment

Docker Compose. Files in this directory:

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Deployment definition |
| `config.yaml` | Combined server configuration (management, signal, relay, STUN) |
| `dashboard.env.example` | Example .env for **dashboard** service |
| `proxy.env.example` | Example .env for **proxy** service |


Edit .env files if needed.

```bash
cp dashboard.env.example dashboard.env
cp proxy.env.example proxy.env
# fill in the real values, then
docker compose up -d
```

---

## Runbooks

- [Update the control plane](../../../docs/runbooks/netbird/update-control-plane.md)
- [Enroll a new peer](../../../docs/runbooks/netbird/enroll-a-peer.md)
- [Expose a homelab service over the overlay](../../../docs/runbooks/netbird/expose-a-service.md)

---

## TODO

- [ ] Define a Backup strategy
