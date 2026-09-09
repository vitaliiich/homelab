# VLANs and Segmentation

The VLANs, what belongs in each, and the rules governing traffic between them.

Physical wiring, trunks VLANs device inventory live in [topology.md](topology.md). This file covers policy: subnets, DHCP ranges, and who is allowed to talk to whom.

---

## VLANs

| VLAN | Name | Subnet | Gateway | Tagging | Purpose |
|------|------|--------|---------|---------|---------|
| untagged | Management | 10.10.0.0/24 | 10.10.0.1 | untagged (native) | Hypervisors, switches, AP, infrastructure control planes |
| 10 | Trusted | 10.10.10.0/24 | 10.10.10.1 | tagged | Personal workstations, phones, tablets |
| 20 | IoT | 10.10.20.0/24 | 10.10.20.1 | tagged | Cameras, smart plugs, TVs, Zigbee devices |
| 30 | Services | 10.10.30.0/24 | 10.10.30.1 | tagged | Self-hosted application workloads |
| 40 | Guest | 10.10.40.0/24 | 10.10.40.1 | tagged | Guest devices, anything untrusted |


### Addressing


| Segment | Static range | DHCP pool | Reservations |
|---------|--------------|-----------|--------------|
| Management | .2 - .40 | .41 - .245 | PVE Nodes, switches, DNS servers, `netbird01`, `unifios01`, UniFi AP |
| Trusted | .2 - .40 | .41 - .245 | - |
| IoT | .2 - .40 | .41 - .245 | SmartTV (because of OPNsense alias for mDNS service rules) |
| Services | .2 - .40 | .41 - .245 | All service hosts |
| Guest | .2 - .40 | .41 - .245 | - |

DHCP is served per interface by `freya`. Both Technitium instances are handed out as resolvers on every subnet.

---

## What lives where

### Management (untagged)

| Host | Type | Notes |
|------|------|-------|
| `freya` | bare metal | OPNsense, gateway for all segments |
| `thor` / `odin` / `loki` | bare metal | Proxmox VE cluster |
| core-switch / access-switch | bare metal | Switch management interfaces |
| U7 Lite AP | bare metal | AP |
| `technitium01` / `technitium02` | LXC | Authoritative and recursive DNS Cluster |
| `unifios01` | VM | UniFi OS Server |
| `netbird01` | LXC | NetBird routing peer |

### Trusted (VLAN 10)

Laptops, phones, desktops. Can access everything, but Management subnet.

### IoT (VLAN 20)

Smart TV(s), IoT Devices (e.g. Aqara Sensors, Smart Air Purifier, Smart Humidifier, etc). Devices here are assumed to be untrustworthy.

### Services (VLAN 30)

| Host | Type | Runs |
|------|------|------|
| `podman01` | VM | MariaDB, Karakeep |
| `homeassistant` | bare metal | Home Assistant OS |

### Guest (VLAN 40)

Random guests' devices. Internet only.

---

## Inter-VLAN policy

Default is **deny**. Anything not listed below is dropped and logged.

Rows are the source, columns the destination:

| From \ To | Mgmt | Trusted | IoT | Services | Guest | WAN |
|-----------|------|---------|-----|----------|-------|-----|
| **Management** | - | allow | allow | allow | allow | allow |
| **Trusted** | deny | - | allow | allow | allow | allow |
| **IoT** | deny | deny | - | deny | deny | **allow** |
| **Services** | deny | deny | allow | - | deny | allow |
| **Guest** | deny | deny | deny | deny | - | allow |


### Explicit exceptions


| Source | Destination | Ports | Reason |
|--------|-------------|-------|--------|
| all segments | `technitium01`, `technitium02` | 53 tcp/udp | DNS/Name resolution |
| IoT | 224.0.0.251 | 5353 udp | Allow Cast (Chromecast) |
| IoT (SmartTV host) | Trusted subnet | 32768:61000 udp | Allow Cast (Chromecast) |


---

## Adding a segment

1. Create the VLAN interface on `freya`, assign the gateway address.
2. Add a DHCP scope and set the resolvers.
3. Tag the VLAN on the trunk ports between `freya`, core-switch and access-switch.
4. Tag it on the AP trunk if it needs wireless, and new subnet into Networks and PSSK list in the SSID config.
5. Add firewall rules. Start with deny-all, then add exceptions one at a time (e.g. Technitium DNS Servers).
6. Update the matrix above, [topology.md](topology.md), and the [docs/network/diagrams/topology.svg](Excalidraw diagram).
7. Verify from a client in the new segment: gateway reachable, DNS resolving, and inter-VLAN traffic blocked in both directions.
