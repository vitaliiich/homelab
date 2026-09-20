# Upgrade the NetBird control plane

Follows the [official upgrade guide](https://docs.netbird.io/selfhosted/maintenance/upgrade) and [backup guide](https://docs.netbird.io/selfhosted/maintenance/backup), with the verification steps this deployment needs.

**When:** once every 2 weeks, only on weekends. Or after each major CVE published affecting NetBird's components.
**Risk:** remote access to the lab is down during the restart. Established tunnels survive briefly, but no peer can re-register while the server is down.
**Time:** 15-30 min
**Last verified:** Sep 20, 2026

> **Do not run this while away from home.**
> If the upgrade fails there is no second path to the lab, and the only way to fix the server is the VPS provider's console.

---

## 1. Back up

**Config files:**
```bash
mkdir backup
cp docker-compose.yml dashboard.env config.yaml backup/
```

**Databases:**
```bash
docker compose stop netbird-server
docker compose cp -a netbird-server:/var/lib/netbird/ backup/
docker compose start netbird-server
```

---

## 2. Pull new Docker images

```bash
docker compose pull netbird-server dashboard proxy
```

---

## 3. Recreate the containers from new images

```bash
docker compose up -d --force-recreate netbird-server dashboard proxy
```

---

## 4. Verify

**Services**

```bash
docker compose ps
docker compose logs --since 5m | grep -i -E 'error|fatal|panic|warn'
```
