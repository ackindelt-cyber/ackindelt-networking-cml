## Status

Initial planning document.

This document captures the current v1 topology and feature scope before VLANs, addressing, interface numbering, port-channel numbering, or device configuration are finalized.

Primary next step: build and link the topology in CML to confirm that the selected nodes support the required interface count.

---

## 1. Purpose

This project is a simulated enterprise campus network built in Cisco CML.

The v1 goal is to create a realistic but controlled enterprise campus foundation with:

- ISP simulation
- Customer edge router
- Routed firewall boundary
- Dual collapsed core/distribution switches
- Layer 2 access switches
- Redundant access uplinks
- LACP EtherChannels
- HSRP gateway redundancy
- STP and HSRP alignment
- Centralized DHCP simulation
- DHCP relay
- PAT through the firewall
- Dedicated management VLAN

V1 is not intended to include every possible enterprise feature. It is the base topology and core design.

---

## 2. Planned Devices

| Device | Suggested CML Node | Role |
|---|---|---|
| ISP1 | IOSv | Simulated ISP/provider router |
| EDGE1 | IOSv | Customer edge router |
| FW1 | ASAv | Routed firewall |
| CORE1 | IOSvL2 | Collapsed core/distribution switch |
| CORE2 | IOSvL2 | Collapsed core/distribution switch |
| ASW1 | IOSvL2 | Layer 2 access switch |
| ASW2 | IOSvL2 | Layer 2 access switch |
| ASW3 | IOSvL2 | Layer 2 access switch |
| INFRA1 | IOSv | Simulated centralized DHCP/infrastructure server |
| C1/C2/C3/C4/PRN1 | Alpine/Desktop | Endpoint simulation |

---

## 3. Link Map

This is the initial CML build target.

No VLANs, IP addresses, port-channel numbers, or interface numbers are assigned yet.

| Link Group | Required Links |
|---|---|
| ISP edge | ISP1 to EDGE1 |
| Edge firewall | EDGE1 to FW1 |
| Firewall core | FW1 to CORE1; FW1 to CORE2 |
| Core interconnect | Two links between CORE1 and CORE2 |
| ASW1 uplinks | Two links from ASW1 to CORE1; two links from ASW1 to CORE2 |
| ASW2 uplinks | Two links from ASW2 to CORE1; two links from ASW2 to CORE2 |
| ASW3 uplinks | Two links from ASW3 to CORE1; two links from ASW3 to CORE2 |
| Infrastructure server | ASW2 to INFRA1 |
| Endpoints | ASW1 to C1/C2; ASW2 to C3; ASW3 to C4/PRN1 |

---

## 4. Interface Count Estimate

| Device | Estimated Links Needed | Notes |
|---|---:|---|
| ISP1 | 1 | Link to EDGE1 |
| EDGE1 | 2 | Links to ISP1 and FW1 |
| FW1 | 3 | Links to EDGE1, CORE1, and CORE2 |
| CORE1 | 9 | FW1, CORE2 x2, ASW1 x2, ASW2 x2, ASW3 x2 |
| CORE2 | 9 | FW1, CORE1 x2, ASW1 x2, ASW2 x2, ASW3 x2 |
| ASW1 | 6 | CORE1 x2, CORE2 x2, C1, C2 |
| ASW2 | 6 | CORE1 x2, CORE2 x2, INFRA1, C3 |
| ASW3 | 6 | CORE1 x2, CORE2 x2, C4, PRN1 |
| INFRA1 | 1 | Link to ASW2 |

The topology should fit within standard IOSv/IOSvL2 interface limits, but this must be confirmed directly in CML during the initial build.

---

## 5. Device Role and Feature Scope

### ISP1

Role:

- Simulates upstream ISP/provider connectivity.
- Provides an external reachability target.
- Does not represent a full ISP design.

Included in v1:

- Basic routed connectivity to EDGE1.

---

### EDGE1

Role:

- Customer edge router between ISP1 and FW1.

Included in v1:

- Routed link to ISP1.
- Routed link to FW1.
- Static default route toward ISP1.
- Basic forwarding between FW1 and ISP1.

---

### FW1

Role:

- Routed firewall/security boundary between EDGE1 and the campus core.

Included in v1:

- Outside interface toward EDGE1.
- Inside connectivity toward CORE1 and CORE2.
- PAT for campus outbound traffic.
- Basic inside/outside stateful firewall behavior.
- Static routing.

Important validation item:

- Confirm whether the deployed ASAv image supports the intended redundant inside interface behavior.
- If unsupported, adjust the FW1-to-core connection design before moving into addressing/configuration.

---

### CORE1 and CORE2

Role:

- Dual collapsed core/distribution layer.

Included in v1:

- VLAN SVIs.
- Inter-VLAN routing.
- HSRP for VLAN default gateways.
- STP root and HSRP active alignment.
- Per-VLAN gateway/path preference.
- Two-link LACP trunk between CORE1 and CORE2.
- LACP trunk bundles to access switches.
- Dedicated firewall transit VLAN.
- HSRP VIP on the firewall transit VLAN for FW1 next hop.
- DHCP relay toward INFRA1.
- Static default route toward FW1.
- Management VLAN gateway.


---

### ASW1, ASW2, and ASW3

Role:

- Layer 2 access switches.

Included in v1:

- Layer 2 access switching only.
- Two-link LACP bundle to CORE1.
- Two-link LACP bundle to CORE2.
- Explicit access port assignments.
- Global PortFast default on access switches.
- Global BPDU Guard default on PortFast ports.
- Management SVI in VLAN 50.
- Default gateway pointing to VLAN 50 HSRP VIP.
- Dedicated native VLAN on trunks.
- Parking VLAN for unused ports.
- Shutdown unused ports.
- DTP disabled on trunks where supported.
- Allowed VLAN lists on trunks.
- Interface descriptions.

---

### INFRA1

Role:

- Simulated centralized infrastructure server.

Included in v1:

- IOSv-based DHCP server.
- DHCP pools for campus VLANs.
- DHCP default gateways set to HSRP VIPs.
- Connected to the server/infrastructure VLAN.

Future option:

- Replace INFRA1 with a custom Linux or Windows server image after the base network is stable.

---

## 6. Proposed VLAN Model

Final VLANs and addressing will be assigned later.

| VLAN | Name | Purpose |
|---:|---|---|
| 10 | USERS | Standard user endpoints |
| 20 | ADMIN | Admin / IT endpoints |
| 30 | SERVERS_INFRA | Servers and infrastructure services |
| 40 | PRINTERS_IOT | Printers, IoT, and operational devices |
| 50 | MGMT | Network device management |
| 900 | FW_TRANSIT | Firewall-to-core transit VLAN |
| 998 | NATIVE | Dedicated native VLAN for trunks |
| 999 | PARKING | Unused / disabled access ports |

VLAN 1 should not be used for endpoint access or intentional management.

---

## 7. Key Design Choices

| Area | V1 Decision |
|---|---|
| ISP redundancy | No |
| Edge router redundancy | No |
| Firewall appliance redundancy | No |
| Firewall inside path redundancy | Yes, pending ASAv support validation |
| Core redundancy | Yes |
| Core interconnect redundancy | Yes |
| Access uplink redundancy | Yes |
| Link redundancy inside access uplinks | Yes |
| DHCP server redundancy | No |
| PAT location | FW1 |
| Inter-VLAN routing location | CORE1/CORE2 |
| DHCP relay location | CORE1/CORE2 SVIs |
| DHCP server location | INFRA1 |
| Management network | VLAN 50 |

---

## 8. STP and HSRP Design Intent

The core pair will use aligned STP root and HSRP active placement.

Design intent:

- For each VLAN, the preferred Layer 2 path and the active default gateway should live on the same core switch.
- This keeps forwarding paths predictable.
- This allows basic per-VLAN load sharing across the core pair.

Example role split:

| VLAN | Preferred Core | STP Role | HSRP Role |
|---:|---|---|---|
| 10 | CORE1 | Root primary | Active |
| 20 | CORE2 | Root primary | Active |
| 30 | CORE1 | Root primary | Active |
| 40 | CORE2 | Root primary | Active |
| 50 | CORE1 | Root primary | Active |

---

## 9. Firewall-to-Core Transit Intent

The firewall-to-core handoff uses a dedicated transit VLAN.

Purpose:

- Give FW1 one stable next-hop IP toward the core pair.
- Allow CORE1 or CORE2 to own the transit HSRP VIP.
- Keep FW1 logically pointed at one next hop even if the active core changes.

Concept:

- FW1 has an inside IP on the firewall transit VLAN.
- CORE1 has an SVI on the firewall transit VLAN.
- CORE2 has an SVI on the firewall transit VLAN.
- CORE1 and CORE2 share an HSRP VIP on the firewall transit VLAN.
- FW1 routes campus-bound traffic toward that HSRP VIP.
- CORE1 and CORE2 route outbound/default traffic toward FW1.

This does not provide firewall appliance redundancy. It only provides core-side path redundancy for a single firewall.

---

## 10. Access Layer Intent

Access switches remain Layer 2.

Access-layer design intent:

- Endpoints connect to explicit access VLANs.
- Access switches uplink redundantly to both cores.
- Each uplink path uses a two-link LACP bundle.
- STP controls which logical uplink forwards for each VLAN.
- Global PortFast and BPDU Guard protect endpoint-facing ports.
- Unused ports are parked and shut down.
- Infrastructure management uses VLAN 50.

---

## 11. Deferred Features

The following are intentionally deferred from v1:

- Firewall HA.
- Dual edge routers.
- BGP to ISP.
- DMZ.
- VPN.
- Dynamic routing between firewall/core/edge.
- VRFs.
- Complex ACL segmentation.
- Port security.
- DHCP snooping.
- Dynamic ARP Inspection.
- IP Source Guard.
- Storm control.
- 802.1X.
- Voice VLANs.
- TACACS/RADIUS.
- SNMP.
- NetFlow.
- Syslog server.
- NTP server.
- Custom Linux DHCP server image.
- Monitoring stack.

---

## 12. Initial Build Checklist

Before addressing or configuration:

- [ ] Add planned CML nodes.
- [ ] Add all required physical links.
- [ ] Confirm each node supports the required number of interfaces.
- [ ] Arrange the topology clearly in CML.
- [ ] Save/export the initial CML topology.
- [ ] Capture a topology screenshot.
- [ ] Validate whether ASAv supports the intended redundant inside interface behavior.
- [ ] Only then proceed to interface numbering, port-channel numbering, VLANs, addressing, and configuration.

---

## 13. V1 Success Criteria

V1 should eventually validate:

- End-to-end client reachability toward ISP simulation.
- DHCP relay from client VLANs to INFRA1.
- DHCP clients receiving HSRP VIPs as default gateways.
- PAT through FW1.
- Inter-VLAN routing through CORE1/CORE2.
- HSRP gateway redundancy.
- STP/HSRP alignment.
- LACP trunk operation.
- Access uplink redundancy.
- Firewall-to-core transit behavior.
- Management VLAN reachability.