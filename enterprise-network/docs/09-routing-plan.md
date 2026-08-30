# Routing Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the planned routing design for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document the expected routing behavior between the simulated ISP, customer edge router, firewall HA pair, core switches, infrastructure services, and internal VLANs.

---

## Scope

### In Scope

- Static routing design
- ISP-to-edge routing
- Edge-to-firewall routing
- Firewall outside routing
- Firewall inside routing
- Core default routing
- Campus internal routing assumptions
- Firewall HA routing behavior
- Routing-related validation expectations

### Out of Scope

- Dynamic routing
- BGP
- OSPF
- EIGRP
- VRFs
- Policy-based routing
- Route redistribution
- Multi-site WAN routing
- Dual ISP routing
- Production internet routing
- IPv6 routing
- Final device configuration output

---

## Summary

The Talos Solutions Enterprise Campus v1 lab uses static routing.

The outbound traffic path is:

```text
Internal VLANs -> CORE1/CORE2 -> FW1/FW2 -> OS1 outside transit segment -> EDGE1 -> ISP1
```

The firewall pair acts as the routed security and PAT boundary between the campus and the edge network.

CORE1 and CORE2 provide inter-VLAN routing for internal campus VLANs. The core pair uses HSRP for client default gateway redundancy and static default routes toward the active firewall inside address.

FW1 and FW2 operate as an ASAv active/standby pair. The active firewall owns the active inside and outside interface addresses and forwards traffic. The standby firewall owns standby interface addresses and is available for failover.

---

## Routing Design Standards

| Area | Standard |
|---|---|
| Routing method | Static routing |
| Campus internal routing | CORE1/CORE2 connected SVIs |
| Client default gateway | HSRP VIP per VLAN |
| Core default route | Toward firewall active inside IP |
| Firewall default route | Toward EDGE1 outside-transit IP |
| Firewall route to campus | Toward CORE1/CORE2 HSRP VIP on VLAN 900 |
| EDGE1 default route | Toward ISP1 |
| NAT/PAT location | FW1/FW2 active firewall |
| Dynamic routing | Deferred from v1 |

---

## High-Level Routing Path

### Outbound Client Traffic

Expected outbound flow:

```text
Client
  -> VLAN HSRP VIP on CORE1/CORE2
  -> CORE default route to firewall active inside IP
  -> Active firewall PAT
  -> EDGE1 via outside transit segment
  -> ISP1
  -> Simulated external destination
```

### Return Traffic

Expected return flow:

```text
Simulated external destination
  -> ISP1
  -> EDGE1
  -> Active firewall outside IP
  -> Firewall translation/state table
  -> CORE1/CORE2
  -> Client VLAN
  -> Client
```

---

## Routing Domains

| Routing Area | Devices | Purpose |
|---|---|---|
| ISP simulation | ISP1 | Simulated upstream/external routing |
| Customer edge | EDGE1 | Routes between ISP1 and firewall outside segment |
| Firewall boundary | FW1/FW2 | Security boundary, PAT, static routing |
| Campus core | CORE1/CORE2 | Inter-VLAN routing and campus default gateway |
| Access layer | ASW1/ASW2/ASW3 | Layer 2 only; no routing |
| Infrastructure | INFRA1 | Server/infrastructure host using VLAN 30 gateway |

---

## ISP1 Routing

### Role

ISP1 simulates upstream provider connectivity.

### Connected Networks

| Network | Purpose |
|---|---|
| `203.0.113.0/30` | ISP1-to-EDGE1 transit |
| `203.0.113.100/32` | Simulated external reachability target |

### Expected Routes

ISP1 should know how to reach the firewall outside transit network through EDGE1.

| Destination | Next Hop | Purpose |
|---|---|---|
| `198.51.100.0/29` | `203.0.113.2` | Route to firewall outside transit network |

### Design Notes

- ISP1 does not need routes to internal campus networks if PAT is working correctly.
- Internal campus networks should be hidden behind firewall PAT for outbound traffic.
- ISP1 is not intended to simulate a full provider routing environment.

---

## EDGE1 Routing

### Role

EDGE1 is the customer edge router between ISP1 and the firewall outside segment.

### Connected Networks

| Network | Purpose |
|---|---|
| `203.0.113.0/30` | ISP1-to-EDGE1 transit |
| `198.51.100.0/29` | Firewall outside transit segment through OS1 |

### Expected Routes

| Destination | Next Hop | Purpose |
|---|---|---|
| `0.0.0.0/0` | `203.0.113.1` | Default route toward ISP1 |

### Design Notes

- EDGE1 should not perform PAT in v1.
- EDGE1 should not connect directly to CORE1 or CORE2.
- EDGE1 should not bypass the firewall pair.
- EDGE1 routes between the ISP transit and firewall outside transit segment.
- The firewall pair owns security policy and address translation.

---

## OS1 Routing

### Role

OS1 is the outside transit switch.

### Routing Behavior

OS1 is Layer 2 only and does not perform routing.

### Connected Segment

| VLAN | Network | Purpose |
|---:|---|---|
| 901 | `198.51.100.0/29` | Shared outside segment between EDGE1, FW1, and FW2 |

### Design Notes

- OS1 provides Layer 2 adjacency between EDGE1 and both firewall outside interfaces.
- OS1 should not carry internal campus VLANs.
- OS1 should not route traffic.
- OS1 exists as a realistic outside firewall handoff switch.

---

## Firewall Pair Routing

### Role

FW1 and FW2 provide the routed security boundary between EDGE1/OS1 and the campus core.

The firewall pair operates in active/standby mode.

### Connected Networks

| Network | Purpose |
|---|---|
| `198.51.100.0/29` | Firewall outside transit network |
| `10.10.90.0/24` | Firewall inside transit network |
| `10.255.255.0/30` | Firewall failover/state link |

### Firewall Interface Addressing

| Interface Role | Active IP | Standby IP | Purpose |
|---|---|---|---|
| Outside | `198.51.100.2` | `198.51.100.3` | Outside firewall transit |
| Inside | `10.10.90.254` | `10.10.90.253` | Inside firewall transit |
| Failover/state | `10.255.255.1` | `10.255.255.2` | HA communication |

### Expected Routes

| Destination | Next Hop | Purpose |
|---|---|---|
| `0.0.0.0/0` | `198.51.100.1` | Default route toward EDGE1 |
| `10.10.0.0/16` | `10.10.90.1` | Campus summary route toward CORE1/CORE2 HSRP VIP |

### Design Notes

- The active firewall owns the active interface IPs.
- The standby firewall owns the standby interface IPs.
- During failover, the active IPs move to the new active firewall.
- PAT is performed by the active firewall.
- The firewall routes campus-bound traffic toward the core HSRP VIP on VLAN 900.
- The firewall failover/state link should not be used for normal routing.

---

## CORE1 and CORE2 Routing

### Role

CORE1 and CORE2 provide campus Layer 3 gateway services.

They route between internal VLANs and provide the default path from the campus toward the firewall pair.

### Connected Campus Networks

| VLAN | Name | Subnet |
|---:|---|---|
| 10 | USERS | `10.10.10.0/24` |
| 20 | ADMIN | `10.10.20.0/24` |
| 30 | SERVERS_INFRA | `10.10.30.0/24` |
| 40 | PRINTERS_IOT | `10.10.40.0/24` |
| 50 | MGMT | `10.10.50.0/24` |
| 900 | FW_TRANSIT | `10.10.90.0/24` |

### Expected Routes

| Destination | Next Hop | Purpose |
|---|---|---|
| `0.0.0.0/0` | `10.10.90.254` | Default route toward active firewall inside IP |

### Campus Routing Behavior

CORE1 and CORE2 use connected SVIs for internal VLAN routing.

Clients should use HSRP VIPs as their default gateways.

| VLAN | HSRP VIP | Purpose |
|---:|---|---|
| 10 | `10.10.10.1` | User VLAN gateway |
| 20 | `10.10.20.1` | Admin VLAN gateway |
| 30 | `10.10.30.1` | Server/infrastructure VLAN gateway |
| 40 | `10.10.40.1` | Printer/IoT VLAN gateway |
| 50 | `10.10.50.1` | Management VLAN gateway |
| 900 | `10.10.90.1` | Firewall inside transit core-side gateway |

### Design Notes

- CORE1 and CORE2 should have `ip routing` enabled.
- Access switches should not perform inter-VLAN routing.
- CORE1 and CORE2 should both have a default route toward the firewall active inside IP.
- CORE1 and CORE2 should share VLAN 900 across the inter-core trunk.
- HSRP provides stable default gateways for internal VLANs.
- STP and HSRP active placement should be aligned per VLAN.

---

## Access Layer Routing

### Role

ASW1, ASW2, and ASW3 are Layer 2 access switches.

### Routing Behavior

Access switches do not route user traffic in v1.

They should use VLAN 50 for management reachability.

Expected access switch management behavior:

| Device | Management VLAN | Default Gateway |
|---|---:|---|
| ASW1 | 50 | `10.10.50.1` |
| ASW2 | 50 | `10.10.50.1` |
| ASW3 | 50 | `10.10.50.1` |

### Design Notes

- Access switches should use `ip default-gateway 10.10.50.1` if they are operating as Layer 2 switches.
- Access switches should not have routed SVIs for user VLANs.
- Access switches should not have static routes for internal or external networks.

---

## INFRA1 Routing

### Role

INFRA1 provides centralized DHCP service.

### Addressing

| Device | Address | Default Gateway |
|---|---|---|
| INFRA1 | `10.10.30.10/24` | `10.10.30.1` |

### Routing Behavior

INFRA1 should use the VLAN 30 HSRP VIP as its default gateway.

Expected route behavior:

| Destination | Next Hop | Purpose |
|---|---|---|
| `0.0.0.0/0` | `10.10.30.1` | Default route toward CORE1/CORE2 |

### Design Notes

- INFRA1 is inside the campus network.
- INFRA1 should not connect outside the firewall.
- DHCP relay from CORE1/CORE2 SVIs should point to INFRA1.
- INFRA1 should be reachable from routed client VLANs through CORE1/CORE2.

---

## Static Route Summary

### ISP1

| Destination | Next Hop |
|---|---|
| `198.51.100.0/29` | `203.0.113.2` |

### EDGE1

| Destination | Next Hop |
|---|---|
| `0.0.0.0/0` | `203.0.113.1` |

### FW1/FW2

| Destination | Next Hop |
|---|---|
| `0.0.0.0/0` | `198.51.100.1` |
| `10.10.0.0/16` | `10.10.90.1` |

### CORE1/CORE2

| Destination | Next Hop |
|---|---|
| `0.0.0.0/0` | `10.10.90.254` |

### INFRA1

| Destination | Next Hop |
|---|---|
| `0.0.0.0/0` | `10.10.30.1` |

---

## PAT Routing Relationship

PAT occurs on the active firewall.

Expected PAT behavior:

```text
Internal source address -> Translated to firewall outside active IP
```

Example:

```text
10.10.10.100 -> 198.51.100.2
```

Design notes:

- EDGE1 should see traffic sourced from the firewall outside active IP after PAT.
- ISP1 should not need to route directly to internal `10.10.0.0/16` networks.
- Return traffic should come back to the active firewall outside IP.
- Firewall state should allow return traffic to the original internal client.

---

## Failure Behavior Expectations

### Firewall Failover

Expected behavior:

- FW1 or FW2 is active.
- The active firewall owns `198.51.100.2` and `10.10.90.254`.
- If the active firewall fails, the standby firewall becomes active.
- The active IPs move to the new active firewall.
- CORE1/CORE2 continue using `10.10.90.254` as their default next hop.
- EDGE1 continues using the connected outside subnet to reach the firewall active outside IP.

### Core Failover

Expected behavior:

- HSRP provides stable client default gateways.
- If one core fails, the other core takes over affected HSRP roles.
- Internal clients continue using the same default gateway IPs.
- Routing toward the firewall should remain available if the surviving core can reach VLAN 900 and the active firewall.

### Access Uplink Failure

Expected behavior:

- LACP and STP handle access uplink failures.
- Access switches continue forwarding through an available core path.
- Client default gateways remain the HSRP VIPs.

---

## Dynamic Routing Deferred

Dynamic routing is intentionally deferred from v1.

Deferred options include:

- OSPF between core and firewall
- OSPF or EIGRP inside the campus
- BGP between EDGE1 and ISP1
- Dynamic routing for future multi-site WAN design
- Route tracking or floating static routes

Reasoning:

Static routing keeps the first integrated enterprise build controlled and easier to validate. Dynamic routing can be added after the base topology, firewall HA, HSRP, STP, DHCP, PAT, and failure testing are stable.

---

## Routing Validation Expectations

Expected validation commands may include:

```text
show ip route
show ip route static
show ip interface brief
show standby brief
show arp
ping
traceroute
```

Firewall validation commands may include:

```text
show route
show xlate
show conn
show nat
show failover
packet-tracer
```

Detailed command output should be captured in the verification documents, not in this design document.

---

## Design Notes

- Static routing is used in v1 for control and simplicity.
- CORE1 and CORE2 route between internal VLANs.
- Access switches operate as traditional Layer 2 access switches in v1.
- FW1/FW2 provide the routed security boundary and PAT.
- EDGE1 routes between ISP1 and the firewall outside segment.
- OS1 is Layer 2 only.
- ISP1 is a simulated upstream network.
- The firewall failover/state link is not part of the routed data path.
- Client default gateways should always be HSRP VIPs, not physical SVI addresses.

---

## Validation or Success Criteria

The routing plan is successful when:

- Static routes are documented for ISP1, EDGE1, FW1/FW2, CORE1/CORE2, and INFRA1.
- Internal VLANs route through CORE1/CORE2.
- Clients use HSRP VIPs as default gateways.
- CORE1 and CORE2 route outbound traffic to the firewall active inside IP.
- The firewall routes campus-bound traffic to the VLAN 900 HSRP VIP.
- The firewall routes default traffic to EDGE1.
- EDGE1 routes default traffic to ISP1.
- PAT allows internal clients to reach the simulated external destination.
- Firewall failover does not require core default route changes.
- Dynamic routing remains deferred from v1.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/08-port-channel-plan.md`
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `docs/12-management-plan.md`
- `verification/03-l3-verification.md`
- `verification/05-firewall-pat-verification.md`
- `verification/07-failure-testing.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |