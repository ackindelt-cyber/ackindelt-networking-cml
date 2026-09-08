# Initial Planning

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Planning  

---

## Purpose

This document captures the initial planning direction for the Talos Solutions Enterprise Campus v1 lab.

The goal of this lab is to build a simulated enterprise campus network in Cisco CML that demonstrates practical network design, implementation, redundancy, firewall placement, infrastructure services, and operational verification.

This document is a planning snapshot. Detailed design information is documented in the supporting design documents.

---

## Scope

### In Scope

- Single simulated enterprise campus
- ISP-to-edge-to-firewall-to-core-to-access topology
- Simulated ISP router
- Single customer edge router
- Outside transit switch for firewall handoff
- Active/standby ASAv firewall pair
- Dual collapsed core/distribution switches
- Layer 2 access switches
- HSRP gateway redundancy
- STP and HSRP alignment
- LACP trunks
- Centralized DHCP using INFRA1
- DHCP relay
- Firewall PAT for outbound access
- Static routing
- Management VLAN foundation
- Baseline verification
- Failure testing

### Out of Scope

- Dual ISP
- Edge router redundancy
- Redundant outside switch pair
- DMZ
- Dynamic routing
- Full access-layer security hardening
- DNS
- Syslog
- ELK/OpenSearch
- Suricata
- Multi-campus design
- Automation

---

## Summary

Talos Solutions Enterprise Campus v1 is a Cisco CML lab that simulates a small-to-medium enterprise campus network.

The planned topology uses a simulated ISP, a single customer edge router, an outside transit switch, an active/standby ASAv firewall pair, dual collapsed core/distribution switches, multiple Layer 2 access switches, and a centralized infrastructure node.

The v1 build focuses on creating a stable enterprise network foundation before adding advanced services, deeper security controls, monitoring, automation, or multi-site expansion.

---

## Planned Topology

High-level path:

```text
ISP1 <-> EDGE1 <-> FW1/FW2 <-> CORE1/CORE2 <-> ASW1/ASW2/ASW3 <-> Clients
```

Planned devices:

| Device | Role                                   |
|--------|----------------------------------------|
| ISP1   | Simulated ISP                          |
| EDGE1  | Customer edge router                   |
| FW1    | Active firewall                        |
| FW2    | Standby firewall                       |
| CORE1  | Collapsed core/distribution switch     |
| CORE2  | Collapsed core/distribution switch     |
| ASW1   | Access switch                          |
| ASW2   | Access switch                          |
| ASW3   | Access switch                          |
| INFRA1 | Centralized infrastructure/DHCP server |
| C1-C4  | Client test nodes                      |
| PRN1   | Simulated printer/IoT endpoint         |

---

## Design Notes

- PAT should live on FW1/FW2
- CORE1 and CORE2 provide redundant default gateways using HSRP.
- STP root placement should align with HSRP active gateway placement.
- Access switches should remain Layer 2 only in v1.
- INFRA1 should provide centralized DHCP service.
- Static routing is preferred in v1 to keep the base design understandable and stable.
- Advanced services should be added only after the base campus network is verified.

---

## Validation or Success Criteria

V1 is successful when:

- The CML topology is built and documented.
- VLANs and addressing are clearly defined.
- Port-channels form correctly.
- Trunks carry the expected VLANs.
- HSRP operates as expected.
- STP root placement matches the design.
- Clients receive DHCP addresses.
- Clients can reach expected internal and external destinations.
- Active Firewall performs outbound PAT.
- Failure testing confirms expected redundancy behavior.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-documentation-plan.md`
- `docs/02-topology-and-device-roles.md`
- `docs/03-design-decisions.md`
- `docs/12-build-order.md`
- `docs/13-known-limitations.md`
- `docs/14-future-roadmap.md`

---

## Change Log

| Date      | Change                                  |
|-----------|-----------------------------------------|
| 2026-08-30| Rewritten as clean v1 planning snapshot |