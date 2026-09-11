# Hardware

Everything piece of hardware in the lab: from a compute node to a PDU.
What it is, where it came from, what did it cost, and how it's mounted.

---

## Design constraints

The lab was built around three constraints that explain most of the choices below.

**Price** Lenovo Tiny desktops are plentiful on the second-hand market, have expandable SODIMM RAM, sip power, and cost a fraction of equivalent new hardware. Three nodes for cluster quorum was the target, so per-unit price mattered more than per-unit performance.

**10 inch, not 19.** A standard 19 inch rack does not fit in an apartment, which constrains every other choice: switches must be half-width, and nothing deep or loud is an option. The DeskPi Rackmate is probably the most popular 10 inch rack solution. 

**Quiet and low power.** The rack lives in living space, so it runs 24/7 within earshot.

---

## Pricing

>**Note! All the prices listed below are as of Q1-Q2 2026**.

### Nodes

| Component | Qty | Source | Condition | Unit price | Shipping | Total |
|-----------|-----|--------|-----------|------------|----------|-------|
| Lenovo ThinkCentre M920q (`freya`): Intel Core i5-8500T/8 Gb DDR4/256 Gb NVMe | 1 | eBay US | used | $139.99 | $17.63 | $157.62 |
| Lenovo ThinkCentre M710q (`thor`, `odin`, `loki`): Intel Core i7-6700T/32 Gb DDR4/No Drive | 3 | eBay US | used | $105.41 | $42.27 | $443.04 |
| NVMe drives (256 Gb) | 3 | olx.ua | used | $29.46 | $0.61 | $90.21 |
| Intel X710-DA2 LP FC 2-port SFF+ PCI-E card | 1 | eBay US | used | $24.97 | $3.41 | $28.38 |
| Lenovo ThinkCentre M920q PCIe riser + backplate | 1 | Aliexpress | new | $18.34 | $0.00 | $18.34 |
| Lenovo 65w PSU (square tip) | 4 | Rozetka.ua | used | $42.52 | $0.00 | $42.52 |
| Lenovo PSU Power Cord (C8 Type) | 4 | Prom.ua | new | $1.03 | $0.51 | $6.16 |

### Network

| Component | Qty | Source | Condition | Unit price | Shipping | Total |
|-----------|-----|--------|-----------|------------|----------|-------|
| ONTi ONT-S508CL-8S (`core-switch`) | 1 | Aliexpress | new | $78.84 | $0.00 | $78.84 |
| GoodTop ZX310S-8T2XS (`access-switch`) | 1 | Aliexpress | new | $64.05 | $0.00 | $64.05 |
| Ubiquiti U7 Lite AP | 1 | suri.com.ua | new | $120.53 | $0.00 | $120.53 |
| Ubiquiti UACC-PoE+-2.5G PoE+ Adapter | 1 | enko.ua | new | $23.29 | $2.12 | $25.41 |
| Unifi U7 Lite Desktop Stand | 1 | prom.ua | new | $7.12 | $0.00 | $7.12 |
| GoodTop 10G SFP+ DAC (0.25m) | 1 | Aliexpress | new | $8.81 | $0.00 | $8.81 |
| OPTFOCUS 10G SFP+ DAC (0.5m) | 1 | Aliexpress | new | $7.26 | $0.00 | $7.26 |
| CAT6a RJ-45 Patch cables | 15 | Aliexpress | new | $1.89 | $0.00 | $28.35 |

### Rack, power and KVM

| Component | Qty | Source | Condition | Unit price | Shipping | Total |
|-----------|-----|--------|-----------|------------|----------|-------|
| DeskPi RackMate T2 | 1 | deskpi.com | new | $168.10 | $49.52 | $217.62 |
| DeskPi RackMate Mini PC Rack Shelf, 1U | 4 | Aliexpress | new | $31.40 | $0.00 | $125.60 |
| DeskPi 2U Plate | 1 | Aliexpress | new | $15.38 | $0.00 | $15.38 |
| DeskPi 0.5U Brush Panel | 1 | Aliexpress | new | $13.92 | $0.00 | $13.92 |
| 6-socket power strip | 1 | Aliexpress | new | $36.00 | $0.00 | $36.00 |
| 5-socket power strip | 1 | Aliexpress | new | $30.65 | $0.00 | $30.65 |
| Unnlink 4K DisplayPort to HDMI Adapter | 3 | Aliexpress | new | $5.76 | $0.00 | $17.28 |
| Navceker KVM Switch HDMI 4x1 4K 240Hz | 1 | Aliexpress | new | $68.18 | $0.00 | $68.18 |
| HDMI Cable (1m) | 4 | Rozetka.ua | new | $2.02 | $0.00 | $8.08 |

### Total

| Category | Cost |
|----------|------|
| Compute | $786.27 |
| Network | $340.37 |
| Rack | $532.71 |
| **Total** | **$1659.35** |

---

## Rack layout

Photos of the finished build: [`images/`](images/)

---

## Planned

- [ ] UPS sized for a graceful shutdown
- [ ] NAS Build for Proxmox centralized storage and backups
