# VLAN Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the planned VLAN structure for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document VLAN IDs, VLAN names, VLAN purposes, expected Layer 3 gateway behavior, STP/HSRP alignment, trunking expectations, and special-purpose VLANs before final addressing and configuration work begins.

---

## Scope

### In Scope

- Campus VLAN list
- VLAN purpose and placement
- Firewall outside transit VLAN
- Firewall inside transit VLAN
- Native VLAN
- Parking VLAN
- HSRP gateway intent
- STP root alignment intent
- Trunk VLAN expectations
- VLANs deferred from v1

### Out of Scope

- Final IP addressing
- SVI IP assignments
- HSRP VIP addresses
- DHCP scope ranges
- Interface numbering
- Port-channel numbering
- Final switch configuration
- Access-layer security configuration
- Verification output

---

## Summary

The v1 VLAN design separates user, admin, infrastructure, printer/IoT, management, firewall transit, native, and unused-port traffic.

CORE1 and CORE2 provide Layer 3 gateway services for campus VLANs using SVIs and HSRP. Access switches operate as traditional Layer 2 access switches in v1.

The design uses STP and HSRP alignment so that the preferred Layer 2 forwarding path and the active Layer 3 gateway for each VLAN normally reside on the same core switch.

A dedicated firewall inside transit VLAN connects the firewall HA pair to the core pair. A separate outside transit VLAN exists on OS1 to provide the shared outside segment between EDGE1 and FW1/FW2.

---

## Planned VLANs

| VLAN | Name | Purpose | Layer 3 Gateway Location | Notes |
|---:|---|---|---|---|
| 10 | USERS | Standard user endpoints | CORE1/CORE2 HSRP | General client access |
| 20 | ADMIN | Admin / IT endpoints | CORE1/CORE2 HSRP | Administrative user systems |
| 30 | SERVERS_INFRA | Servers and infrastructure services | CORE1/CORE2 HSRP | INFRA1 and future internal services |
| 40 | PRINTERS_IOT | Printers, IoT, and operational devices | CORE1/CORE2 HSRP | Printer and simulated IoT endpoint traffic |
| 50 | MGMT | Network device management | CORE1/CORE2 HSRP | Switch/router/firewall management reachability |
| 900 | FW_TRANSIT | Firewall inside transit VLAN | CORE1/CORE2 HSRP | Shared inside segment between firewall HA pair and core pair |
| 901 | OUTSIDE_TRANSIT | Firewall outside transit VLAN | EDGE1 / Firewall outside subnet | Local to OS1; not carried into campus trunks |
| 998 | NATIVE | Dedicated native VLAN for trunks | None | No endpoint access |
| 999 | PARKING | Unused / disabled access ports | None | No active hosts |

---

## VLAN Role Details

### VLAN 10 - USERS

VLAN 10 is used for standard user endpoint simulation.

Expected use:

- General client nodes
- DHCP client testing
- Default gateway testing through HSRP
- Internal and external reachability testing

Expected gateway behavior:

- Gateway provided by CORE1/CORE2 HSRP
- DHCP default-router should be the VLAN 10 HSRP VIP

---

### VLAN 20 - ADMIN

VLAN 20 is used for admin or IT endpoint simulation.

Expected use:

- Administrative client node
- Future management-access testing
- Future segmentation testing

Expected gateway behavior:

- Gateway provided by CORE1/CORE2 HSRP
- DHCP default-router should be the VLAN 20 HSRP VIP

---

### VLAN 30 - SERVERS_INFRA

VLAN 30 is used for centralized infrastructure services.

Expected use:

- INFRA1 DHCP server
- Future DNS server
- Future syslog server
- Future NTP server
- Other internal service simulation

Expected gateway behavior:

- Gateway provided by CORE1/CORE2 HSRP
- INFRA1 should use the VLAN 30 HSRP VIP as its default gateway

---

### VLAN 40 - PRINTERS_IOT

VLAN 40 is used for printers, IoT, and operational device simulation.

Expected use:

- PRN1 simulated printer endpoint
- Future IoT/OT-like device simulation
- Future access-control or segmentation testing

Expected gateway behavior:

- Gateway provided by CORE1/CORE2 HSRP
- DHCP default-router should be the VLAN 40 HSRP VIP if DHCP is used

---

### VLAN 50 - MGMT

VLAN 50 is used for network device management.

Expected use:

- Management SVIs on access switches
- Management IPs for core switches
- Future SSH management access
- Future management ACLs
- Future syslog/SNMP/NetFlow source reachability

Expected gateway behavior:

- Gateway provided by CORE1/CORE2 HSRP
- Access switches should use the VLAN 50 HSRP VIP as their default gateway

Design note:

VLAN 50 is for network infrastructure management. It should not be used as a normal endpoint/user VLAN.

---

### VLAN 900 - FW_TRANSIT

VLAN 900 is used as the firewall inside transit VLAN between the ASAv HA pair and the core pair.

Expected use:

- FW1 inside interface
- FW2 inside interface
- CORE1 SVI
- CORE2 SVI
- HSRP VIP used as the firewall pair's next hop toward internal networks

Design intent:

- CORE1 and CORE2 share VLAN 900 across the inter-core trunk.
- FW1 connects inside toward CORE1.
- FW2 connects inside toward CORE2.
- The active firewall routes campus-bound traffic toward the CORE1/CORE2 HSRP VIP on VLAN 900.
- CORE1 and CORE2 route outbound/default traffic toward the active firewall inside address.

Design note:

VLAN 900 is not a normal client VLAN. It exists only for routed firewall-to-core transit behavior.

---

### VLAN 901 - OUTSIDE_TRANSIT

VLAN 901 is used as the outside firewall transit VLAN on OS1.

Expected use:

- EDGE1 outside-firewall-facing interface
- FW1 outside interface
- FW2 outside interface

Design intent:

- OS1 provides a shared Layer 2 outside segment.
- EDGE1, FW1, and FW2 participate in the same outside firewall subnet.
- VLAN 901 is local to OS1.
- VLAN 901 should not be carried into the campus core or access layer.

Design note:

VLAN 901 represents the outside firewall handoff segment. It is separate from internal campus VLANs.

---

### VLAN 998 - NATIVE

VLAN 998 is used as the dedicated native VLAN on 802.1Q trunks.

Expected use:

- Native VLAN on trunk links
- No endpoint access
- No routed gateway

Design intent:

- Avoid using VLAN 1 as the native VLAN.
- Keep the native VLAN unused by endpoints.
- Make trunk behavior explicit and predictable.

---

### VLAN 999 - PARKING

VLAN 999 is used for unused and disabled access ports.

Expected use:

- Unused switchports
- Disabled access ports
- Ports intentionally removed from production VLANs

Design intent:

- Unused ports should be assigned to VLAN 999 and shut down.
- VLAN 999 should not provide active network access.
- VLAN 999 should not have an SVI gateway.

---

## STP and HSRP Alignment Intent

CORE1 and CORE2 should align STP root placement with HSRP active gateway placement.

Design intent:

- For each VLAN, the preferred Layer 2 path and active default gateway should normally live on the same core switch.
- This keeps forwarding paths predictable.
- This supports basic per-VLAN load sharing across the core pair.
- This reduces unnecessary traffic crossing the inter-core trunk just to reach the active default gateway.

Planned role split:

| VLAN | Preferred Core | STP Role | HSRP Role | Notes |
|---:|---|---|---|---|
| 10 | CORE1 | Root primary | Active | User VLAN |
| 20 | CORE2 | Root primary | Active | Admin VLAN |
| 30 | CORE1 | Root primary | Active | Server/infrastructure VLAN |
| 40 | CORE2 | Root primary | Active | Printer/IoT VLAN |
| 50 | CORE1 | Root primary | Active | Management VLAN |
| 900 | CORE1 | Root primary | Active | Firewall inside transit VLAN |

CORE1 and CORE2 should be configured as STP primary/secondary opposites per VLAN based on the preferred core.

---

## VLAN Trunking Expectations

### Core-to-Core Trunk

The CORE1-to-CORE2 inter-core port-channel should carry all shared campus VLANs and transit VLANs.

Expected VLANs:

```text
10,20,30,40,50,900,998,999
```

The outside transit VLAN should not be carried here:

```text
901
```

VLAN 901 is local to OS1 and should not enter the campus core.

---

### Core-to-Access Trunks

Core-to-access port-channels should carry the VLANs required by each access switch.

Expected common allowed VLANs:

```text
10,20,30,40,50,998,999
```

VLAN 900 should generally not be carried to access switches unless there is a specific design reason.

VLAN 901 should never be carried to access switches.

---

### OS1 Outside Segment

OS1 should carry only the outside firewall transit segment needed between EDGE1, FW1, and FW2.

Expected local VLAN:

```text
901
```

OS1 should not carry internal campus VLANs.

---

## VLAN 1 Usage

VLAN 1 should not be used for endpoint access, management, native VLAN, or intentional lab services.

Design intent:

- Do not place clients in VLAN 1.
- Do not use VLAN 1 for management.
- Do not use VLAN 1 as the planned native VLAN.
- Keep VLAN 1 unused except for default platform behavior that cannot be removed.

---

## Deferred VLANs

The following VLAN types are intentionally deferred from v1:

- Voice VLAN
- Guest wireless VLAN
- Dedicated security tools VLAN
- DMZ VLAN
- Dedicated backup VLAN
- Dedicated storage VLAN
- Dedicated out-of-band management VLAN
- Multi-site WAN transit VLANs

These may be added in later versions if the lab expands.

---

## Design Notes

- VLANs should have clear purposes and should not be created without a reason.
- VLAN 900 and VLAN 901 are transit VLANs, not user access VLANs.
- VLAN 900 is internal to the firewall-to-core handoff.
- VLAN 901 is external/outside-facing and local to OS1.
- VLAN 998 provides a dedicated unused native VLAN.
- VLAN 999 provides a controlled location for unused access ports.
- Final VLAN-to-access-switch placement should be documented after the topology and endpoint placement are confirmed.

---

## Validation or Success Criteria

The VLAN plan is successful when:

- All v1 VLANs are documented.
- Each VLAN has a clear purpose.
- VLAN 1 is not used for intentional access or management.
- CORE1/CORE2 provide HSRP gateways for routed campus VLANs.
- STP root placement aligns with HSRP active placement.
- VLAN 900 is available between the firewall pair and the core pair.
- VLAN 901 remains local to OS1.
- Trunks carry only the VLANs required for their role.
- Unused access ports are assigned to VLAN 999 and shut down.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
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