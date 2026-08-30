# Topology and Device Roles

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design / Planning  

---

## Purpose

This document defines the planned topology and device roles for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to identify the devices used in the lab, describe the role of each device, document the planned physical link structure, and capture CML-specific topology assumptions before VLANs, addressing, interface numbering, port-channel numbering, or configuration work is finalized.

---

## Scope

### In Scope

- Planned v1 device list
- Suggested Cisco CML node types
- Device roles
- High-level topology path
- Initial physical link map
- Interface count estimate
- CML topology assumptions
- Firewall HA topology direction
- Topology validation items

### Out of Scope

- VLAN ID assignment
- IP addressing
- Interface numbering
- Port-channel numbering
- Device configurations
- Detailed routing configuration
- Firewall policy configuration
- Verification output
- Failure testing results

---

## Summary

Talos Solutions Enterprise Campus v1 is designed as a single-campus enterprise network built in Cisco CML.

The topology uses a simulated ISP, a customer edge router, a shared outside firewall transit segment, an active/standby ASAv firewall pair, a redundant collapsed core/distribution pair, multiple Layer 2 access switches, centralized infrastructure services, and simulated endpoints.

High-level traffic path:

```text
ISP1 -> EDGE1 -> OS1 -> FW1/FW2 -> CORE1/CORE2 -> ASW1/ASW2/ASW3 -> Clients
```

The v1 topology is intended to demonstrate a practical enterprise campus foundation without adding unnecessary complexity such as dual ISPs, dynamic routing, DMZs, or multi-site WAN design.

The firewall design uses an ASAv active/standby HA pair to provide firewall appliance redundancy between the edge network and the campus core.

---

## Planned Devices

| Device | Suggested CML Node | Role |
|---|---|---|
| ISP1 | IOSv | Simulated ISP/provider router |
| EDGE1 | IOSv | Customer edge router |
| OS1 | IOSvL2 | Shared outside firewall transit switch |
| FW1 | ASAv | Primary firewall in HA pair |
| FW2 | ASAv | Secondary firewall in HA pair |
| CORE1 | IOSvL2 | Collapsed core/distribution switch |
| CORE2 | IOSvL2 | Collapsed core/distribution switch |
| ASW1 | IOSvL2 | Layer 2 access switch |
| ASW2 | IOSvL2 | Layer 2 access switch |
| ASW3 | IOSvL2 | Layer 2 access switch |
| INFRA1 | IOSv | Simulated centralized DHCP/infrastructure server |
| C1 | Alpine/Desktop | User endpoint simulation |
| C2 | Alpine/Desktop | User endpoint simulation |
| C3 | Alpine/Desktop | User or admin endpoint simulation |
| C4 | Alpine/Desktop | Endpoint simulation |
| PRN1 | Alpine/Desktop | Simulated printer/IoT endpoint |

---

## High-Level Topology

The planned topology follows a traditional enterprise campus pattern:

```text
ISP / WAN Simulation
        |
      EDGE1
        |
       OS1
        |
    FW1 / FW2
        |
  CORE1 / CORE2
        |
ASW1 / ASW2 / ASW3
        |
 Clients / Printer / Infrastructure
```

Topology description:

```text
ISP-backed, single-edge, active/standby firewall pair, dual-core collapsed campus with dual-homed Layer 2 access switches and centralized DHCP service simulation.
```

---

## Firewall HA Topology Intent

The v1 design uses an active/standby ASAv firewall pair.

The firewall pair provides firewall appliance redundancy. The active firewall forwards traffic, while the standby firewall is available to take over if the active firewall or a monitored interface fails.

Firewall HA design intent:

- FW1 and FW2 operate as an active/standby pair.
- OS1 provides the shared outside Layer 2 segment between EDGE1, FW1, and FW2.
- FW1 connects inside toward CORE1.
- FW2 connects inside toward CORE2.
- CORE1 and CORE2 share the inside firewall transit VLAN across the inter-core trunk.
- FW1 and FW2 use a dedicated failover/state link.
- PAT is performed by the active firewall.

---

## Device Roles

### ISP1

**Role:** Simulated upstream ISP/provider router.

ISP1 provides an external routing target for the lab. It does not represent a full ISP design.

Included in v1:

- Basic routed connectivity to EDGE1
- External reachability target
- Simulated upstream network behavior

Out of scope for v1:

- Full ISP routing design
- BGP
- Multiple providers
- Provider redundancy

---

### EDGE1

**Role:** Customer edge router between ISP1 and the firewall outside segment.

EDGE1 represents the customer-owned edge device that connects the enterprise to the simulated provider network.

Included in v1:

- Routed link to ISP1
- Routed link to OS1
- Static default route toward ISP1
- Basic forwarding between the firewall outside segment and ISP1

Out of scope for v1:

- NAT/PAT
- BGP
- Edge redundancy
- Direct core connectivity
- Firewall/security policy enforcement

Design note:

PAT should live on the firewall pair, not EDGE1. EDGE1 should route traffic between the firewall outside segment and ISP simulation while FW1/FW2 handle the security boundary and translation.

---

### OS1

**Role:** Outside transit switch.

OS1 represents an outside transit switch used to provide a shared Layer 2 handoff between EDGE1 and the FW1/FW2 firewall pair.

Included in v1:

- Layer 2 connectivity between EDGE1 and both firewall outside interfaces
- Shared outside firewall subnet support
- Outside firewall handoff simulation
- Minimal switching function

Out of scope for v1:

- User access switching
- Campus VLAN transport
- Routing
- Security policy enforcement
- Advanced Layer 2 features
- Full outside switch redundancy

Design note:

OS1 allows both ASAv firewalls to participate in the same outside subnet while preserving a clean separation between the customer edge router and the firewall security boundary.

In v1, OS1 is implemented as a single outside transit switch to keep the edge simple. A redundant outside switch pair is a possible future enhancement.

---

### FW1 and FW2

**Role:** Active/standby routed firewall pair between EDGE1/OS1 and the campus core.

FW1 and FW2 provide the north/south security boundary for the campus network.

Included in v1:

- Active/standby ASAv failover
- Outside connectivity through OS1
- Inside connectivity toward CORE1 and CORE2
- Dedicated failover/state link between FW1 and FW2
- PAT for campus outbound traffic
- Basic inside/outside stateful firewall behavior
- Static routing

Out of scope for v1:

- DMZ
- VPN
- Inbound public services
- Complex firewall policy matrix
- IDS/IPS tuning
- Multi-context firewalling
- Dual ISP firewall edge design

Design notes:

- FW1 and FW2 operate as an active/standby firewall pair.
- The active firewall provides the routed security boundary and PAT.
- The standby firewall provides appliance redundancy.
- The firewall pair should be validated before finalizing baseline verification.
- Firewall failover behavior should be included in v1 failure testing.

---

### CORE1 and CORE2

**Role:** Dual collapsed core/distribution layer.

CORE1 and CORE2 provide campus Layer 3 gateway services, inter-VLAN routing, HSRP redundancy, STP control, and the upstream path toward the firewall pair.

Included in v1:

- VLAN SVIs
- Inter-VLAN routing
- HSRP for VLAN default gateways
- STP root and HSRP active alignment
- Per-VLAN gateway/path preference
- Two-link LACP trunk between CORE1 and CORE2
- LACP trunk bundles to access switches
- Dedicated firewall transit VLAN
- Shared inside firewall transit VLAN between CORE1 and CORE2
- DHCP relay toward INFRA1
- Static default route toward the firewall pair
- Management VLAN gateway

Out of scope for v1:

- Dynamic routing
- VRFs
- Policy-based routing
- Complex ACL segmentation
- QoS
- MST
- Private VLANs
- TACACS/RADIUS
- SNMP/NetFlow/syslog integration

Design note:

CORE1 and CORE2 must share the firewall inside/transit VLAN across the inter-core trunk so the firewall HA pair can operate with equivalent inside network connectivity.

---

### ASW1, ASW2, and ASW3

**Role:** Layer 2 access switches.

The access switches provide endpoint connectivity and redundant uplinks to both core switches.

Included in v1:

- Traditional Layer 2 access switching
- Two-link LACP bundle to CORE1
- Two-link LACP bundle to CORE2
- Explicit access port assignments
- Global PortFast default on access switches
- Global BPDU Guard default on PortFast ports
- Management SVI in VLAN 50
- Default gateway pointing to VLAN 50 HSRP VIP
- Dedicated native VLAN on trunks
- Parking VLAN for unused ports
- Shutdown unused ports
- DTP disabled on trunks where supported
- Allowed VLAN lists on trunks
- Interface descriptions

Out of scope for v1:

- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- 802.1X
- Voice VLANs

Design note:

The access switches operate as traditional Layer 2 access switches in v1, with inter-VLAN routing centralized on CORE1 and CORE2.

Each access switch uses one LACP bundle to CORE1 and one separate LACP bundle to CORE2. A single port-channel should not be split across both cores because this topology does not use StackWise Virtual, VSS, vPC, or MLAG.

---

### INFRA1

**Role:** Simulated centralized infrastructure server.

INFRA1 provides centralized DHCP service for the v1 lab.

Included in v1:

- IOSv-based DHCP server
- DHCP pools for campus VLANs
- DHCP default gateways set to HSRP VIPs
- Connected to the server/infrastructure VLAN

Future option:

- Replace INFRA1 with a custom Linux or Windows server image after the base network is stable.

Out of scope for v1:

- DNS server
- Syslog server
- NTP server
- Monitoring stack
- Production-style server redundancy

---

### Endpoint Nodes

**Role:** Simulated clients, printers, and test hosts.

Endpoint nodes are used to validate access-layer switching, DHCP relay, default gateway behavior, firewall/PAT behavior, and end-to-end reachability.

Planned endpoint nodes:

| Device | Planned Use |
|---|---|
| C1 | User endpoint |
| C2 | User endpoint |
| C3 | User or admin endpoint |
| C4 | Endpoint simulation |
| PRN1 | Simulated printer/IoT endpoint |

---

## Initial Link Map

This is the initial CML build target.

No VLANs, IP addresses, interface numbers, or port-channel numbers are assigned in this document.

| Link Group | Required Links |
|---|---|
| ISP edge | ISP1 to EDGE1 |
| Edge outside transit | EDGE1 to OS1 |
| Firewall outside | OS1 to FW1; OS1 to FW2 |
| Firewall HA | Dedicated failover/state link between FW1 and FW2 |
| Firewall inside | FW1 to CORE1; FW2 to CORE2 |
| Core interconnect | Two links between CORE1 and CORE2 |
| ASW1 uplinks | Two links from ASW1 to CORE1; two links from ASW1 to CORE2 |
| ASW2 uplinks | Two links from ASW2 to CORE1; two links from ASW2 to CORE2 |
| ASW3 uplinks | Two links from ASW3 to CORE1; two links from ASW3 to CORE2 |
| Infrastructure server | ASW2 to INFRA1 |
| Endpoints | ASW1 to C1/C2; ASW2 to C3; ASW3 to C4/PRN1 |

---

## Interface Count Estimate

| Device | Estimated Links Needed | Notes |
|---|---:|---|
| ISP1 | 1 | Link to EDGE1 |
| EDGE1 | 2 | Links to ISP1 and OS1 |
| OS1 | 3 | EDGE1, FW1 outside, FW2 outside |
| FW1 | 3 | Outside, inside, failover/state |
| FW2 | 3 | Outside, inside, failover/state |
| CORE1 | 9 | FW1, CORE2 x2, ASW1 x2, ASW2 x2, ASW3 x2 |
| CORE2 | 9 | FW2, CORE1 x2, ASW1 x2, ASW2 x2, ASW3 x2 |
| ASW1 | 6 | CORE1 x2, CORE2 x2, C1, C2 |
| ASW2 | 6 | CORE1 x2, CORE2 x2, INFRA1, C3 |
| ASW3 | 6 | CORE1 x2, CORE2 x2, C4, PRN1 |
| INFRA1 | 1 | Link to ASW2 |

The topology should fit within standard IOSv and IOSvL2 interface limits, but this must be confirmed directly in CML during the initial build.

---

## Physical Link Reference

Final interface numbering may change during the CML build. This table captures the intended physical relationship between devices.

| Local Device | Peer Device | Description |
|---|---|---|
| ISP1 | EDGE1 | Simulated ISP connection to customer edge |
| EDGE1 | OS1 | EDGE1 connection to shared outside firewall segment |
| OS1 | FW1 | FW1 outside connection |
| OS1 | FW2 | FW2 outside connection |
| FW1 | FW2 | Dedicated failover/state link |
| FW1 | CORE1 | FW1 inside connection to CORE1 |
| FW2 | CORE2 | FW2 inside connection to CORE2 |
| CORE1 | CORE2 | Two-link inter-core LACP trunk |
| CORE1 | ASW1 | Two-link LACP uplink bundle |
| CORE2 | ASW1 | Two-link LACP uplink bundle |
| CORE1 | ASW2 | Two-link LACP uplink bundle |
| CORE2 | ASW2 | Two-link LACP uplink bundle |
| CORE1 | ASW3 | Two-link LACP uplink bundle |
| CORE2 | ASW3 | Two-link LACP uplink bundle |
| ASW2 | INFRA1 | Infrastructure server connection |
| ASW1 | C1 | Endpoint connection |
| ASW1 | C2 | Endpoint connection |
| ASW2 | C3 | Endpoint connection |
| ASW3 | C4 | Endpoint connection |
| ASW3 | PRN1 | Printer/IoT endpoint connection |

---

## Topology Assumptions

- The lab is built in Cisco CML.
- ISP1, EDGE1, and INFRA1 use IOSv.
- OS1, CORE1, CORE2, ASW1, ASW2, and ASW3 use IOSvL2.
- FW1 and FW2 use ASAv.
- FW1 and FW2 operate as an active/standby firewall pair.
- OS1 provides the shared outside Layer 2 segment for the firewall pair.
- CORE1 and CORE2 share the inside firewall transit VLAN across their inter-core trunk.
- Access switches operate as traditional Layer 2 access switches in v1.
- CORE1 and CORE2 provide Layer 3 gateway services.
- The firewall pair provides the security and PAT boundary.
- EDGE1 does not bypass the firewall pair to reach the core.
- INFRA1 is connected inside the campus network, not outside the firewall.

---

## Design Notes

- The topology intentionally uses dual cores and a firewall HA pair, but still uses a single ISP and single edge router for v1.
- This keeps the design enterprise-relevant while avoiding unnecessary first-version WAN complexity.
- FW1/FW2 provide firewall appliance redundancy between the edge network and the campus core.
- CORE1 and CORE2 provide campus gateway redundancy using HSRP.
- Access switches are dual-homed to the core pair using separate LACP bundles.
- STP controls which logical access uplink forwards for each VLAN.
- FW1/FW2 provide the routed security boundary between the campus and edge.
- INFRA1 provides centralized DHCP to avoid placing DHCP directly on the core switches.
- Advanced services such as DNS, syslog, monitoring, Suricata, ELK/OpenSearch, and automation are future additions.

---

## Validation or Success Criteria

This topology document is complete when:

- All planned v1 devices are identified.
- Each device has a clear role.
- CML node types are selected.
- Required links are documented.
- Estimated interface counts are documented.
- CML interface support is validated during topology build.
- FW1/FW2 HA topology is physically built in CML.
- Firewall failover can be tested before the final v1 verification phase.
- The final interface map is moved into `docs/07-interface-map.md` once interface numbering is locked.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/00-project-background.md`
- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/08-port-channel-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`

---

## Change Log

| Date       | Change                                                                |
|------------|-----------------------------------------------------------------------|
| 2026-08-30 | Initial draft                                                         |
| 2026-08-30 | Updated topology direction to use FW1/FW2 ASAv active/standby HA pair |