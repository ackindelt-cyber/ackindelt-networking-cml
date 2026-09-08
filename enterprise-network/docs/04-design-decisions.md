# Design Decisions

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document captures the major design decisions for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to explain why the v1 topology is designed the way it is, what tradeoffs were accepted, and which features were intentionally deferred.

---

## Scope

### In Scope

- Major v1 architecture decisions
- Firewall placement and HA approach
- Edge and ISP design choices
- Core/distribution design choices
- Access layer design choices
- Routing design choices
- DHCP and infrastructure service placement
- Redundancy decisions
- Deferred feature rationale

### Out of Scope

- Final IP addressing
- Final VLAN assignments
- Interface numbering
- Port-channel numbering
- Device configurations
- Verification command output
- Troubleshooting procedures
- Future multi-site design details

---

## Summary

Talos Solutions Enterprise Campus v1 is designed to be a realistic but controlled enterprise campus network.

The design focuses on core network engineering skills:

- Routing and switching
- Firewall placement
- Firewall HA
- Core redundancy
- Access uplink redundancy
- LACP EtherChannels
- HSRP gateway redundancy
- STP and HSRP alignment
- Centralized DHCP
- Static routing
- Baseline validation
- Failure testing

The v1 design intentionally avoids trying to include every possible enterprise feature. The goal is to build a stable and well-documented foundation before expanding into advanced services, security hardening, monitoring, automation, or multi-site design.

---

## Decision: Single-Campus v1 Design

### Decision

Build v1 as a single enterprise campus network.

### Reasoning

A single-campus design is large enough to demonstrate enterprise network concepts without becoming too broad for the first integrated version.

This allows the lab to focus on:

- Core/distribution redundancy
- Layer 2 access design
- Firewall placement
- Infrastructure services
- End-to-end validation
- Failure testing

### Tradeoff

A single-campus design does not demonstrate WAN routing, branch connectivity, dual-provider edge design, or multi-site failover.

Those items are better suited for a future v2 multi-site expansion.

---

## Decision: ISP-to-Edge-to-Firewall-to-Core-to-Access Topology

### Decision

Use the following high-level topology path:

```text
ISP1 -> EDGE1 -> OS1 -> FW1/FW2 -> CORE1/CORE2 -> ASW1/ASW2/ASW3 -> Clients
```

### Reasoning

This creates a clear enterprise traffic flow:

- ISP1 simulates upstream/provider connectivity
- EDGE1 represents the customer edge router
- OS1 provides the shared outside firewall segment
- FW1/FW2 provide the routed firewall boundary
- CORE1/CORE2 provide campus Layer 3 services
- ASW1/ASW2/ASW3 provide Layer 2 endpoint access

This structure keeps routing, security, distribution, and access responsibilities separated.

### Tradeoff

The design uses a single edge router and single ISP connection in v1. This keeps the first version focused, but it means the WAN edge is not redundant.

---

## Decision: Single Edge Router

### Decision

Use one customer edge router, EDGE1, between ISP1 and the firewall outside segment.

### Reasoning

The v1 design is focused on the enterprise campus, not WAN edge redundancy.

A single edge router is enough to demonstrate:

- ISP-facing routed connectivity
- Customer edge routing
- Static default routing
- Firewall outside connectivity

### Tradeoff

EDGE1 is a single point of failure in v1.

Edge redundancy, dual edge routers, dual ISP, and BGP should be deferred to a future edge/WAN expansion.

---

## Decision: PAT Lives on the Firewall Pair

### Decision

Perform PAT on FW1/FW2, not on EDGE1.

### Reasoning

The firewall pair is the enterprise security boundary between the internal campus and external network.

Placing PAT on the firewall keeps security policy and address translation together. This is cleaner than placing NAT on EDGE1 while also using the firewall as the traffic control point.

### Tradeoff

EDGE1 remains a simple routed edge device. This reduces edge complexity but means the firewall pair must be correctly configured for outbound translation and routing.

---

## Decision: Active/Standby ASAv Firewall Pair

### Decision

Use FW1 and FW2 as an active/standby ASAv firewall pair.

### Reasoning

The firewall is a critical boundary between the enterprise campus and the edge network. Using an HA pair gives the lab a stronger enterprise design and allows firewall failover testing.

The selected ASAv image does not support the redundant interface behavior required for a single-firewall redundant-link design, so v1 uses firewall appliance HA instead.

### Design Intent

- FW1 and FW2 operate as an active/standby pair.
- The active firewall forwards traffic.
- The standby firewall is available to take over if the active firewall or a monitored interface fails.
- FW1 and FW2 use a dedicated failover/state link.
- PAT is performed by the active firewall.
- Firewall failover behavior should be validated during failure testing.

### Tradeoff

Firewall HA adds complexity compared to a single firewall. However, the added value is worth it because firewall redundancy is a realistic enterprise design feature.

---

## Decision: Shared Outside Firewall Segment

### Decision

Use OS1 as a shared outside Layer 2 segment between EDGE1, FW1, and FW2.

### Reasoning

Both firewalls in the HA pair need to participate in the same outside subnet so the active firewall can own the active outside address and the standby firewall can take over during failover.

OS1 provides a simple shared segment:

```text
EDGE1
  |
 OS1
 / \
FW1 FW2
```

This allows EDGE1, FW1, and FW2 to participate in the same outside firewall subnet.

### Tradeoff

OS1 is a single outside transit switch in v1, so the outside firewall segment is not fully redundant.

This is acceptable for v1 because the design focus is firewall appliance redundancy, campus core redundancy, access-layer uplink redundancy, and end-to-end validation.

---

## Decision: Dual Collapsed Core/Distribution Switches

### Decision

Use CORE1 and CORE2 as a dual collapsed core/distribution layer.

### Reasoning

A dual-core collapsed design is realistic for a small-to-medium enterprise campus.

CORE1 and CORE2 provide:

- VLAN SVIs
- Inter-VLAN routing
- HSRP default gateway redundancy
- STP root control
- DHCP relay
- Static default path toward the firewall pair
- Redundant access switch uplinks

### Tradeoff

A collapsed core/distribution design is simpler than a full three-tier campus design. For v1, that is acceptable because the lab is focused on foundational enterprise design, not maximum campus scale.

---

## Decision: HSRP for Gateway Redundancy

### Decision

Use HSRP on CORE1 and CORE2 for VLAN default gateway redundancy.

### Reasoning

HSRP gives client VLANs a stable default gateway even if one core switch fails.

Clients should use the HSRP VIP as their default gateway, not the physical SVI address of either core switch.

### Tradeoff

HSRP adds configuration and verification requirements, but it is a core enterprise redundancy feature and is appropriate for v1.

---

## Decision: Align STP Root and HSRP Active Roles

### Decision

Align STP root placement with HSRP active gateway placement per VLAN.

### Reasoning

For each VLAN, the preferred Layer 2 forwarding path should match the active Layer 3 gateway.

This keeps traffic paths predictable and avoids inefficient forwarding where traffic crosses the inter-core link unnecessarily just to reach the active default gateway.

Example intent:

| VLAN | Preferred Core | STP Role | HSRP Role |
|---:|---|---|---|
| 10 | CORE1 | Root primary | Active |
| 20 | CORE2 | Root primary | Active |
| 30 | CORE1 | Root primary | Active |
| 40 | CORE2 | Root primary | Active |
| 50 | CORE1 | Root primary | Active |

### Tradeoff

This requires more intentional configuration than letting STP and HSRP defaults decide behavior. The benefit is a more predictable and professional design.

---

## Decision: Layer 2 Access Switches

### Decision

Use ASW1, ASW2, and ASW3 as traditional Layer 2 access switches in v1.

### Reasoning

The access layer should provide endpoint connectivity, VLAN assignment, trunking, and redundant uplinks to the core pair. Routing between VLANs should be centralized on CORE1 and CORE2 using SVIs, so ASW1, ASW2, and ASW3 remain traditional Layer 2 access switches in v1.

This keeps the access layer simple and consistent.

### Tradeoff

Access switches depend on the core pair for Layer 3 gateway services. That is acceptable for this campus design.

---

## Decision: Separate LACP Bundles to Each Core

### Decision

Each access switch uses one LACP bundle to CORE1 and one separate LACP bundle to CORE2.

### Reasoning

The topology does not use StackWise Virtual, VSS, vPC, MLAG, or another multi-chassis EtherChannel technology.

Because of that, a single port-channel should not be split across CORE1 and CORE2.

Correct design intent:

```text
ASW1 -> Port-channel to CORE1
ASW1 -> Separate port-channel to CORE2
```

STP decides which logical uplink forwards for each VLAN.

### Tradeoff

This design requires STP to block one logical path per VLAN. That is expected in this traditional campus design.

---

## Decision: Two-Link LACP Inter-Core Trunk

### Decision

Use a two-link LACP trunk between CORE1 and CORE2.

### Reasoning

CORE1 and CORE2 need to share VLANs, carry the firewall inside transit VLAN, support HSRP operation, and maintain access-layer redundancy behavior.

A two-link LACP trunk provides additional link redundancy and a cleaner inter-core design than a single trunk link.

### Tradeoff

The inter-core trunk becomes a critical part of the topology and must be carefully verified.

---

## Decision: Centralized DHCP on INFRA1

### Decision

Use INFRA1 as the centralized DHCP server for v1.

### Reasoning

This keeps DHCP service separate from the core switches while still allowing the lab to demonstrate enterprise DHCP relay behavior.

CORE1 and CORE2 SVIs should use DHCP relay toward INFRA1.

Clients should receive:

- An IP address from the correct VLAN scope
- The correct subnet mask
- The HSRP VIP as the default gateway
- DNS/domain values if configured later

### Tradeoff

INFRA1 is a single DHCP server in v1. DHCP redundancy is deferred.

---

## Decision: Static Routing in v1

### Decision

Use static routing between ISP1, EDGE1, FW1/FW2, CORE1/CORE2, and INFRA1.

### Reasoning

Static routing keeps v1 understandable and stable while the topology, VLANs, firewall behavior, HSRP, STP, DHCP, and PAT are being validated.

Dynamic routing can be added later after the base design is proven.

### Tradeoff

Static routing does not demonstrate OSPF, EIGRP, or BGP in the integrated enterprise topology. That is acceptable for v1 because dynamic routing would add complexity before the foundation is validated.

---

## Decision: Dedicated Management VLAN

### Decision

Use a dedicated management VLAN for network device management.

### Reasoning

A dedicated management VLAN separates infrastructure management addressing from normal user, admin, server, printer, and transit networks.

This creates a clean foundation for future management features such as:

- SSH
- Management ACLs
- TACACS/RADIUS
- Syslog
- SNMP
- NetFlow
- Config backup
- Monitoring

### Tradeoff

The management VLAN adds another VLAN and addressing scope to manage. The benefit is worth it because it creates a more realistic enterprise design.

---

## Decision: Defer Advanced Access-Layer Security

### Decision

Defer access-layer hardening features from v1.

Deferred items include:

- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- 802.1X
- Voice VLANs

### Reasoning

These are valuable features, but adding them before the base topology is validated would increase troubleshooting complexity.

The v1 access layer should first prove:

- VLAN access
- Trunking
- LACP
- STP behavior
- HSRP gateway behavior
- DHCP relay
- End-to-end reachability

### Tradeoff

The v1 access layer is not fully hardened. That limitation should be documented and addressed in a future security-hardening phase.

---

## Decision: Defer Monitoring and Security Tooling

### Decision

Defer syslog, SNMP, NetFlow, ELK/OpenSearch, Suricata, and monitoring stack integration from v1.

### Reasoning

These features are valuable, but they belong after the base network is stable.

Future direction:

- Lightweight DNS/syslog/NTP services may be modeled inside CML.
- ELK/OpenSearch, Suricata, and heavier monitoring/security platforms should likely run externally in Proxmox and connect through a CML external connector.

### Tradeoff

V1 will not initially demonstrate full network operations tooling. That is acceptable because v1 is focused on building and validating the enterprise campus foundation.

---

## Decision: Keep v1 Finishable

### Decision

Do not add every good idea to v1.

### Reasoning

The purpose of v1 is to create a complete, validated, and documented enterprise campus foundation.

A smaller finished project is more valuable than a larger unfinished one.

### Tradeoff

Some valuable features are intentionally deferred. Those items should be tracked in the future roadmap instead of being added to v1 without control.

---

## Design Notes

The v1 design should remain focused on:

- Clear topology
- Clean device roles
- Controlled redundancy
- Practical firewall placement
- Predictable Layer 2 and Layer 3 behavior
- Centralized DHCP
- Repeatable verification
- Documented limitations

The project should avoid design sprawl. Future ideas should be captured, but they should not distract from completing the base campus build.

---

## Validation or Success Criteria

This design decisions document is successful when:

- Major design choices are documented.
- Each major decision includes reasoning.
- Tradeoffs are acknowledged.
- Deferred features are clearly separated from v1 scope.
- The design supports implementation and verification.
- The document helps explain the lab to a technical reviewer.

---

## Open Questions

- Should OS1 remain a dedicated IOSvL2 outside segment switch, or should another CML node type be used?
- Should FW1/FW2 use a combined failover/state link or separate failover and state links?
- Should management access be enabled in v1 or reserved for a later management phase?
- Should DNS/syslog/NTP be grouped into a v1.1 services module after baseline validation?
- Should dynamic routing be added before or after firewall HA validation is fully documented?

---

## Related Documents

- `docs/00-project-background.md`
- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/08-port-channel-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `docs/12-management-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |