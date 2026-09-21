# Upgrade the Technitium DNS cluster

Rolling upgrade of `technitium01` and `technitium02` using the automated installer, per the [official install and update guide](https://blog.technitium.com/2017/11/running-dns-server-on-ubuntu-linux.html).

**When:** a new Technitium DNS Server release
**Risk:** low if done one node at a time. DNS stays up throughout, because DHCP hands out both resolvers and the untouched node keeps answering.
**Time:** 15-20 min
**Last verified:** 21 Sep 2026

---

## Cluster layout


| Node | Role | Address | LXC VMID | Proxmox host |
|------|------|---------|----------|--------------|
| `technitium01` | primary | 10.10.0.5 | 100 | `thor `|
| `technitium02` | secondary | 10.10.0.6 | 102 | `odin `|


---

## Before you start

- [ ] Read the changelog for everything between the installed version and the target: <https://github.com/TechnitiumSoftware/DnsServer/blob/master/CHANGELOG.md>. Look for runtime version bumps and config format changes.
- [ ] Note the current version on each node (web console, **About**)
- [ ] Confirm both nodes currently answer, so you know the baseline is healthy:
```bash
dig @10.10.0.5 google.com +short
dig @10.10.0.6 google.com +short
```

---

## 1. Back up

The backup is being done automatically once a day via **Proxmox Replication**

---

## 2. Upgrade the secondary (`technitium02`)

Confirm `technitium01` is answering before touching this node, since it will carry all the traffic:
```bash
dig @10.10.0.5 google.com +short
```

Open a shell for the `technitium02` LXC in Proxmox VE UI and log in.

Run the installer. The same script handles both install and upgrade:
```bash
curl -sSL https://download.technitium.com/dns/install.sh -o /tmp/technitium-install.sh
bash /tmp/technitium-install.sh
```

Watch the service come back up:
```bash
journalctl --unit dns --follow
```

### Verify the secondary
```bash
dig @10.10.0.6 google.com +short
```

- [ ] Service active, no errors in the journal
- [ ] Resolves external names
- [ ] Resolves internal `home.arpa` records
- [ ] Web console loads and shows the new version. If the UI misbehaves, hard-refresh with **Ctrl+F5** to drop cached scripts from the old version.
- [ ] Node shows as healthy in the cluster view

**Do not continue to the primary until every box above is checked.** A broken secondary is harmless while the primary is up. Two broken nodes is an outage.

---

## 3. Upgrade the primary (`technitium01`)

Repeat the same steps.

Run the same verification checklist as for the secondary.

---

## 4. Verify the cluster

From a client in any VLAN:

```bash
dig google.com
dig freya.home.kruk.sh
```

- [ ] Both nodes report the same version
- [ ] Cluster view shows both nodes healthy and in sync
- [ ] Query logs are still arriving in MariaDB on `podman01`.
