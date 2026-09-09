# Network Topology

How the network is physically wired and logically routed: devices, links, trunks, traffic flow and remote access.

VLAN-level detail (subnets, DHCP scopes, firewall policy, which device belongs where) lives in [vlans.md](vlans.md).This file covers the shape of the network, that one covers the rules applied to it.

---

## Diagram

[![Network topology](diagrams/topology.svg)](diagrams/topology.svg)

Source: [`diagrams/topology.excalidraw`](diagrams/topology.excalidraw)

---

## Design principles

A single OPNsense box (`freya`) holds the gateway address for every VLAN and routes all inter-VLAN traffic. The switches only tag and forward frames. This puts every inter-VLAN decision in one firewall rule set instead of spreading it across switch ACLs.

**No inbound ports from the internet.** Remote access runs over a WireGuard (NetBird) overlay. 

---

## Devices

| Host | Hardware | Role | Management address |
|------|----------|------|--------------------|
| freya | Lenovo ThinkCentre M920q | OPNsense: routing, firewall, NAT, DHCP, VLAN gateways | 10.10.0.1 |
| core-switch | Onti ONT-S508CL-8S | Core switch: 802.1Q trunk from `freya` | 10.10.0.2 |
| access-switch | GoodTop ZX310S-8T2XS | Access switch: end-device ports | 10.10.0.3 |
| unifi-u7-lite | Ubiquiti U7 Lite | Wireless AP, SSID per subnet using PSSK | 10.10.0.9 |
| thor | Lenovo ThinkCentre M710q | Proxmox VE node | 10.10.0.235 |
| odin | Lenovo ThinkCentre M710q | Proxmox VE node | 10.10.0.85 |
| loki | Lenovo ThinkCentre M710q | Proxmox VE node | 10.10.0.158 |
| homeassistant | Home Assistant Green Box | Hosts Home Assistant OS and communicates with IoT devices | 10.10.30.4 |

Guests running on the Proxmox cluster are listed in [vlans.md](vlans.md) alongside the subnet they sit in.

---

## Physical links

| From | Port | To | Port | Mode | VLANs | Bandwidth
|------|------|----|------|------|-------|-------|
| ISP's ONT | 0 | freya | em0 (WAN) | access | - | 1 Gbps |
| freya | ixl0 (LAN) | core-switch | 1 | trunk | all tagged, mgmt untagged | 10 Gbps |
| core-switch | 8 | access-switch | 10 | trunk | all tagged, mgmt untagged | 10 Gpbs |
| access-switch | 7 | U7 Lite AP | 0 | trunk | 10, 20, 40 tagged, mgmt untagged | 2.5 Gpbs |
| access-switch | 1 | thor | nic0 | trunk | mgmt untagged, 30 tagged | 1 Gbps |
| access-switch | 2 | odin | nic0 | trunk | mgmt untagged, 30 tagged | 1 Gbps |
| access-switch | 3 | loki | nic0 | trunk | mgmt untagged, 30 tagged | 1 Gbps |

The Proxmox nodes take a trunk rather than an access port so guests can be optionally attached to any VLAN by setting a tag on the virtual NIC, with untagged (native) - mgmt subnet. The bridge is VLAN-aware on each node.

---

## Traffic flow

A frame from a client in VLAN 10 to a service in VLAN 30 takes this path:
1. Client sends to its gateway, `10.10.10.1`, which is `freya`.
2. The access switch tags the frame with VLAN 10 and forwards it up the trunk.
3. The core switch passes it through to `freya`.
4. `freya` evaluates the firewall rules for VLAN 10 to VLAN 30.
5. If allowed, it routes the packet and sends it back down the trunk tagged as VLAN 30.
6. The switch delivers it to the destination port.

Everything crossing a VLAN boundary passes `freya` twice on the wire. Within a VLAN, the switch forwards locally and `freya` is not involved.

---

## Wireless

The U7 Lite sits on a trunk port and maps clients to a specific VLAN using PSSK technology. You essentially have a single SSID, with multiple passwords grant you access to a specific subnet.

The UniFi controller (UniFi OS) runs self-hosted on the `unifios01` host, a VM on the Proxmox cluster in Management VLAN. The AP is adopted by the controller but keeps forwarding traffic if the controller is down, so an offline controller is not an outage.

---

## Name resolution

Authoritative and recursive DNS runs on `technitium01` and `technitium02` in a cluster mode. These are two LXCs on separate cluster nodes so a single node reboot does not take resolution with it.
Query logs are written to MariaDB on `podman01`.

DHCP is served by `freya` per VLAN, handing out both Technitium instances as resolvers (on default port `53`).

Internal zone (Search Domain): `home.kruk.sh`. Records and the naming convention are covered below.

---

## Remote access

Remote access uses a NetBird overlay rather than a port forwarding.

- `netbird01`, an LXC in Management subnet, runs as a **routing peer** and advertises internal subnets to the mesh.
- Remote peers and `netbird01` both register outbound to the NetBird control plane, which handles authentication and signaling only.
- Overlay subnet: `100.64.0.0/10`.
- Advertised routes: Management, Trusted, IoT, Services.
- Access between peers and advertised subnets is governed by NetBird ACLs, not by the OPNsense rule set.

The result is that the WAN interface has no open inbound ports at all.

---

## Naming convention

Physical hosts take Norse names (`freya`, `thor`, `odin`, `loki`). Virtual and replaceable hosts take functional names with a numeric suffix (`podman01`, `technitium01`, `netbird01`, `unifios01`).

---

## Planned

- [ ] Second uplink between switches for redundancy using LAGG
- [ ] Second uplink between `freya` and core-switch for redundancy using LAGG
- [ ] Export switch configs into [`network/switches/`](../../network/switches/)
