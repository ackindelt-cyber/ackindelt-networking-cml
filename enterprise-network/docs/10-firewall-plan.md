# Firewall Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the firewall design for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document the role of the ASAv firewall pair, the inside and outside firewall segments, firewall HA behavior, PAT placement, routing expectations, basic policy intent, and firewall-related validation requirements.

---

## Scope

### In Scope

- FW1/FW2 active/standby firewall pair
- Firewall outside transit design
- Firewall inside transit design
- Firewall failover/state link
- Firewall routing expectations
- PAT placement
- Basic inside/outside security policy intent
- Firewall HA validation expectations
- Firewall-related limitations

### Out of Scope

- DMZ design
- VPN
- Remote access VPN
- Site-to-site VPN
- Inbound public services
- Complex firewall policy matrix
- IDS/IPS tuning
- Multi-context firewalling
- Dual ISP firewall edge design
- Production firewall hardening
- Final firewall configuration output

---

## Summary

The Talos Solutions Enterprise Campus v1 lab uses an active/standby ASAv firewall pair as the routed security boundary between the enterprise campus and the simulated edge/ISP network.

High-level firewall placement:

```text
ISP1 -> EDGE1 -> OS1 -> FW1/FW2 -> CORE1/CORE2 -> Internal VLANs
```

FW1 and FW2 operate as a firewall HA pair. The active firewall forwards traffic, performs PAT, maintains connection state, and routes traffic between the outside and inside firewall networks. The standby firewall is available to take over if the active firewall or a monitored interface fails.

---

## Firewall Design Standards

| Area | Standard |
|---|---|
| Firewall platform | ASAv |
| Firewall count | Two |
| Firewall mode | Routed |
| HA mode | Active/standby |
| NAT/PAT location | Firewall pair |
| Outside segment | VLAN 901 / OUTSIDE_TRANSIT |
| Inside segment | VLAN 900 / FW_TRANSIT |
| Failover/state link | Dedicated FW1-to-FW2 link |
| Routing method | Static routing |
| DMZ | Deferred |
| VPN | Deferred |
| Inbound public services | Deferred |

---

## Firewall Devices

| Device | Role |
|---|---|
| FW1 | ASAv firewall, HA pair member |
| FW2 | ASAv firewall, HA pair member |

The firewalls should be configured as an active/standby pair.

Expected behavior:

- One firewall is active.
- One firewall is standby.
- The active firewall owns the active inside and outside IP addresses.
- The standby firewall owns the standby inside and outside IP addresses.
- If the active firewall fails, the standby firewall becomes active.
- The active firewall performs routing, stateful inspection, and PAT.

---

## Firewall Topology

Conceptual layout:

```text
EDGE1
  |
 OS1
 / \
FW1 FW2
 |   |
CORE1 CORE2
```

Traffic path:

```text
Internal client
  -> CORE1/CORE2
  -> Active firewall inside interface
  -> Active firewall outside interface
  -> EDGE1
  -> ISP1
```

Return traffic path:

```text
ISP1
  -> EDGE1
  -> Active firewall outside interface
  -> Active firewall state/translation table
  -> CORE1/CORE2
  -> Internal client
```

---

## Outside Firewall Segment

### Purpose

The outside firewall segment provides shared Layer 2 connectivity between EDGE1 and the outside interfaces of FW1/FW2.

### VLAN

| VLAN | Name | Purpose |
|---:|---|---|
| 901 | OUTSIDE_TRANSIT | Shared outside firewall transit segment |

### Subnet

```text
198.51.100.0/29
```

### Devices on the Segment

| Device | Role | Address |
|---|---|---|
| EDGE1 | Firewall outside next hop | `198.51.100.1/29` |
| FW1/FW2 active outside IP | Active firewall outside address | `198.51.100.2/29` |
| FW1/FW2 standby outside IP | Standby firewall outside address | `198.51.100.3/29` |

### Design Notes

- OS1 provides the outside Layer 2 segment.
- OS1 should not route traffic.
- OS1 should not carry internal campus VLANs.
- VLAN 901 should remain local to OS1.
- EDGE1 should route traffic toward the active firewall outside address as needed.
- The firewall pair should use EDGE1 as its default route next hop.

---

## Inside Firewall Segment

### Purpose

The inside firewall segment provides connectivity between the firewall pair and the collapsed core/distribution pair.

### VLAN

| VLAN | Name | Purpose |
|---:|---|---|
| 900 | FW_TRANSIT | Firewall inside transit VLAN |

### Subnet

```text
10.10.90.0/24
```

### Devices on the Segment

| Device | Role | Address |
|---|---|---|
| CORE1/CORE2 HSRP VIP | Core-side firewall next hop | `10.10.90.1/24` |
| CORE1 SVI | CORE1 firewall transit SVI | `10.10.90.2/24` |
| CORE2 SVI | CORE2 firewall transit SVI | `10.10.90.3/24` |
| FW1/FW2 active inside IP | Active firewall inside address | `10.10.90.254/24` |
| FW1/FW2 standby inside IP | Standby firewall inside address | `10.10.90.253/24` |

### Design Notes

- FW1 connects inside toward CORE1.
- FW2 connects inside toward CORE2.
- CORE1 and CORE2 share VLAN 900 across the inter-core trunk.
- CORE1 and CORE2 use the active firewall inside IP as their default route next hop.
- The firewall pair routes internal campus traffic toward the VLAN 900 HSRP VIP.
- VLAN 900 is a transit VLAN only and should not be used for endpoints.

---

## Firewall Failover / State Link

### Purpose

The firewall failover/state link is used for ASAv HA communication between FW1 and FW2.

### Planned Subnet

```text
10.255.255.0/30
```

### Planned Addressing

| Device Role | Address | Purpose |
|---|---|---|
| FW1/FW2 failover active IP | `10.255.255.1/30` | Active unit failover/state address |
| FW1/FW2 failover standby IP | `10.255.255.2/30` | Standby unit failover/state address |

### Design Notes

- The failover/state link is dedicated to firewall HA.
- It should not be used for normal data-plane routing.
- It should not carry user, management, or transit VLAN traffic.
- The design may be adjusted later if separate failover and state links are preferred.

---

## Firewall Interface Summary

| Interface Role | Network | Active IP | Standby IP | Connected To |
|---|---|---|---|---|
| Outside | `198.51.100.0/29` | `198.51.100.2` | `198.51.100.3` | OS1 |
| Inside | `10.10.90.0/24` | `10.10.90.254` | `10.10.90.253` | CORE1/CORE2 |
| Failover/state | `10.255.255.0/30` | `10.255.255.1` | `10.255.255.2` | FW1/FW2 direct link |

---

## Firewall Routing

### Firewall Default Route

The firewall pair should route default traffic toward EDGE1.

| Destination | Next Hop | Purpose |
|---|---|---|
| `0.0.0.0/0` | `198.51.100.1` | Default route toward EDGE1 |

### Firewall Route to Campus

The firewall pair should route internal campus traffic toward the CORE1/CORE2 HSRP VIP on VLAN 900.

| Destination | Next Hop | Purpose |
|---|---|---|
| `10.10.0.0/16` | `10.10.90.1` | Campus summary route toward core pair |

### Core Default Route

CORE1 and CORE2 should route outbound/default traffic toward the active firewall inside IP.

| Device | Destination | Next Hop |
|---|---|---|
| CORE1 | `0.0.0.0/0` | `10.10.90.254` |
| CORE2 | `0.0.0.0/0` | `10.10.90.254` |

### Design Notes

- Static routing is used in v1.
- Dynamic routing between the firewall and core is deferred.
- The firewall should not need individual routes for every VLAN if the `10.10.0.0/16` summary is used.
- More specific routes may be used later if the design requires tighter control.

---

## PAT Design

### Decision

PAT should be performed on the active firewall.

### Expected Behavior

Internal source addresses should be translated to the firewall outside active IP when reaching the simulated external network.

Example:

```text
10.10.10.100 -> 198.51.100.2
```

### Source Networks

PAT should apply to internal campus VLANs that require outbound access.

Planned PAT source networks:

```text
10.10.10.0/24
10.10.20.0/24
10.10.30.0/24
10.10.40.0/24
10.10.50.0/24
```

VLAN 900 should not be treated as a normal PAT source network because it is the firewall inside transit VLAN.

### Design Notes

- EDGE1 should not perform PAT.
- PAT belongs on the firewall because the firewall is the security and translation boundary.
- ISP1 should not need routes back to internal `10.10.0.0/16` networks if PAT is working correctly.
- PAT behavior should be validated using client traffic to the simulated external destination.

---

## Basic Security Policy Intent

The v1 firewall policy should remain simple.

### Intended Policy Behavior

| Direction | Intent |
|---|---|
| Inside to outside | Allow internal clients to reach simulated external destinations |
| Outside to inside | Deny unsolicited inbound traffic by default |
| Firewall management | To be finalized in the management plan |
| DMZ traffic | Not applicable in v1 |
| VPN traffic | Not applicable in v1 |

### Design Notes

- The firewall should demonstrate basic stateful behavior.
- Inside-originated traffic should be allowed outbound.
- Return traffic should be allowed because of firewall state.
- Unsolicited outside-to-inside traffic should not be allowed.
- Any temporary test permissions should be documented if used.

---

## Security Levels

Planned security-level intent:

| Interface | Security Level | Purpose |
|---|---:|---|
| Inside | Higher | Campus/internal side |
| Outside | Lower | Edge/ISP-facing side |
| Failover/state | N/A | HA communication only |

Final security-level values should be confirmed during firewall configuration.

---

## Firewall HA Behavior

### Normal Operation

Expected normal behavior:

- One firewall is active.
- One firewall is standby.
- The active firewall owns the active outside IP.
- The active firewall owns the active inside IP.
- The active firewall forwards traffic.
- The active firewall performs PAT.
- The standby firewall monitors HA status and is ready to take over.

### Failover Operation

Expected failover behavior:

- The standby firewall becomes active if the active firewall or monitored interface fails.
- Active interface IPs move to the new active firewall.
- CORE1 and CORE2 continue using `10.10.90.254` as their default next hop.
- EDGE1 continues reaching the active firewall outside IP through the connected outside segment.
- Existing connections may or may not survive depending on final stateful failover behavior and test conditions.

### Validation Notes

Firewall HA should be validated in the failure testing phase.

Tests should include:

- Verify active/standby status.
- Generate client traffic through the active firewall.
- Confirm PAT translations.
- Trigger firewall failover.
- Confirm the standby firewall becomes active.
- Confirm client traffic works after failover.
- Confirm active interface IPs move correctly.
- Confirm routing does not require manual changes.

---

## Firewall Management

Firewall management design is not finalized in this document.

Possible options:

- In-band management through the inside interface
- Dedicated management interface if supported and useful
- Management access from VLAN 50
- Management access deferred until a later management phase

Firewall management should be documented in:

```text
docs/12-management-plan.md
```

---

## Firewall Verification Expectations

Expected firewall verification commands may include:

```text
show failover
show interface ip brief
show route
show nat
show xlate
show conn
show arp
ping
packet-tracer
show running-config failover
show running-config nat
show running-config route
```

Detailed command output should be captured in the verification documents, not in this design document.

---

## Firewall Failure Testing Expectations

Firewall failure testing should eventually validate:

- FW1 active / FW2 standby baseline
- FW2 active / FW1 standby after failover
- Outside interface failover behavior
- Inside interface failover behavior
- Full firewall node failure behavior
- PAT behavior before and after failover
- Client reachability before and after failover
- Core default route stability during firewall failover
- EDGE1 reachability to the active firewall outside IP after failover

Detailed failure testing should be documented in:

```text
verification/07-failure-testing.md
```

---

## Deferred Firewall Features

The following firewall features are intentionally deferred from v1:

- DMZ
- Inbound NAT
- Public services
- Remote access VPN
- Site-to-site VPN
- IDS/IPS tuning
- Advanced application inspection
- Complex ACL matrix
- Dual ISP firewall edge
- Dynamic routing with the core
- Dynamic routing with the edge
- Multi-context firewalling

---

## Design Notes

- FW1 and FW2 provide the routed security boundary.
- OS1 provides the shared outside firewall handoff segment.
- VLAN 900 provides the inside firewall transit segment.
- VLAN 901 provides the outside firewall transit segment.
- PAT should occur on the active firewall.
- EDGE1 should remain a routed edge device and should not perform PAT.
- CORE1 and CORE2 should route default traffic toward the active firewall inside IP.
- The firewall should route campus traffic toward the VLAN 900 HSRP VIP.
- Firewall HA adds complexity, but it provides a stronger and more realistic enterprise design.

---

## Validation or Success Criteria

The firewall plan is successful when:

- FW1 and FW2 are documented as an active/standby pair.
- Outside, inside, and failover/state networks are documented.
- Active and standby firewall IP addresses are documented.
- PAT placement is documented.
- Firewall static routes are documented.
- Core default route behavior is documented.
- Basic policy intent is documented.
- Firewall management is identified as a separate planning item.
- Firewall HA validation expectations are documented.
- Deferred firewall features are clearly separated from v1 scope.

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
- `docs/09-routing-plan.md`
- `docs/12-management-plan.md`
- `verification/05-firewall-pat-verification.md`
- `verification/07-failure-testing.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |