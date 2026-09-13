# Enable mDNS between Trusted and IoT

**When:** a device in VLAN 10 needs to discover a device in VLAN 20 (casting to a TV, AirPlay, a network printer, HomeKit)
**Risk:** opens a permanent hole in the Trusted/IoT boundary.
**Time:** 20-30 min including testing
**Last verified:** Sep 10, 2026

---

## Why this is needed

mDNS is how devices announce themselves on a local network without a DNS server. A TV advertises "I am a Chromecast" to the multicast group `224.0.0.251` on UDP 5353 to anyone.

Putting the TV and the laptop in separate subnets means the laptop never hears the announcement and the cast button does not appear.

Fixing this needs two separate things:
1. **Discovery.** A repeater on `freya` that listens for mDNS on one interface and re-announces it on the other.
2. **Connectivity.** Firewall rules allowing the actual unicast traffic.

---

## Before you start

- [ ] Note which specific devices need to work. You would want to harden down the actual OPNsense firewall rule to the specific device(s) if possible for better security.
- [ ] Back up the OPNsense config (*System > Configuration > Backups > Download*)

---

## Steps

### 1. Install the repeater plugin
**System > Firmware > Plugins**, install `os-mdns-repeater`.

### 2. Configure the repeater
**Services > mDNS Repeater**

| Setting | Value |
|---------|-------|
| Enable | yes |
| Interfaces | Trusted (VLAN 10), IoT (VLAN 20) |


Apply and confirm the service is running.

### 3. Create Aliases
Create 2 **Aliases**:

| Name | Type | Content | Description |
|---------|-------|-------|-------|
| Cast_Media | Port(s) | 32768:61000 | Ports for Cast devices (UDP) |
| Cast_Media | Host(s) | {IP_OF_YOUR_SMART_TV} | Cast devices under IoT Subnet |


### 4. Allow multicast through the firewall
**Firewall > Rules [new] > Trusted**, and add the following rules:

| Interface | Action | Protocol | Source | Source Port | Destination | Destination Port |
|-------|-------|-------|-------|-------|-------|-------|
| IOT | Pass | UDP | Network `IOT` | any | Single Host `224.0.0.251` | Single Port `5353` |
| IOT | Pass | UDP | Alias `Chromecasts` | any | Network `TRUSTED` | Alias `Cast_Media` |

