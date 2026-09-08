# Addressing Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the planned IP addressing structure for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document internal VLAN subnets, transit networks, HSRP gateway addresses, firewall active/standby addressing, DHCP ranges, reserved address ranges, and external simulation addressing before device configuration begins.

---

## Scope

### In Scope

- Internal campus addressing
- VLAN subnet assignments
- HSRP VIP addressing
- CORE1 and CORE2 SVI addressing
- Firewall inside and outside addressing
- Firewall failover/state link addressing
- ISP and edge transit addressing
- DHCP range planning
- Reserved address ranges
- External reachability test addressing

### Out of Scope

- Final interface numbering
- Final port-channel numbering
- Final device configurations
- DNS records
- Final syslog, NTP, SNMP, and monitoring service implementation details
- Public internet connectivity
- IPv6 addressing
- Multi-site addressing
- Production IP address management

---

## Summary

Talos Solutions Enterprise Campus v1 uses private RFC1918 addressing for internal campus networks and documentation-safe public test ranges for simulated ISP and outside firewall transit networks.

Internal campus addressing uses the `10.10.0.0/16` range.

Campus VLANs generally follow this pattern:

```text
10.10.<VLAN-ID>.0/24
```

Exceptions exist for high-numbered transit VLANs, such as VLAN 900, because the VLAN ID cannot be directly mapped into the third octet.

The firewall outside and ISP simulation use documentation ranges rather than real public IP space.

---

## Addressing Standards

### General Standards

| Item | Standard |
|---|---|
| Internal campus summary | `10.10.0.0/16` |
| Standard VLAN mask | `/24` |
| HSRP VIP | `.1` where possible |
| CORE1 SVI | `.2` where possible |
| CORE2 SVI | `.3` where possible |
| Infrastructure/static devices | `.10-.49` |
| Reserved/static range | `.50-.99` |
| DHCP client range | `.100-.199` |
| Reserved future range | `.200-.254` |

### Gateway Standard

For routed campus VLANs, clients should use the HSRP VIP as their default gateway.

Example:

```text
VLAN 10 USERS
HSRP VIP: 10.10.10.1
CORE1 SVI: 10.10.10.2
CORE2 SVI: 10.10.10.3
Client default gateway: 10.10.10.1
```

---

## Internal Campus VLAN Addressing

| VLAN | Name | Subnet | HSRP VIP | CORE1 SVI | CORE2 SVI | Notes |
|---:|---|---|---|---|---|---|
| 10 | USERS | `10.10.10.0/24` | `10.10.10.1` | `10.10.10.2` | `10.10.10.3` | Standard user endpoints |
| 20 | ADMIN | `10.10.20.0/24` | `10.10.20.1` | `10.10.20.2` | `10.10.20.3` | Admin / IT endpoints |
| 30 | SERVERS_INFRA | `10.10.30.0/24` | `10.10.30.1` | `10.10.30.2` | `10.10.30.3` | INFRA1 and future services |
| 40 | PRINTERS_IOT | `10.10.40.0/24` | `10.10.40.1` | `10.10.40.2` | `10.10.40.3` | Printer and IoT simulation |
| 50 | MGMT | `10.10.50.0/24` | `10.10.50.1` | `10.10.50.2` | `10.10.50.3` | Network device management |
| 900 | FW_TRANSIT | `10.10.90.0/24` | `10.10.90.1` | `10.10.90.2` | `10.10.90.3` | Firewall inside transit |

---

## Special-Purpose VLAN Addressing

| VLAN | Name | Subnet | Gateway | Notes |
|---:|---|---|---|---|
| 901 | OUTSIDE_TRANSIT | `198.51.100.0/29` | EDGE1 / firewall outside subnet | Shared outside segment between EDGE1 and FW1/FW2 |
| 998 | NATIVE | None | None | Dedicated unused native VLAN |
| 999 | PARKING | None | None | Unused / disabled ports |

VLAN 998 and VLAN 999 should not have SVIs or routed gateways in v1.

---

## ISP and Edge Addressing

### ISP1-to-EDGE1 Transit

| Device | Interface Role | IP Address | Notes |
|---|---|---|---|
| ISP1 | ISP-facing transit | `203.0.113.1/30` | Simulated ISP side |
| EDGE1 | ISP-facing transit | `203.0.113.2/30` | Customer edge side |

Transit subnet:

```text
203.0.113.0/30
```

### Simulated External Reachability

| Device | Address | Purpose |
|---|---|---|
| ISP1 loopback/test network | `203.0.113.100/32` | External reachability target |

This address is used only as a simulated external destination for routing and PAT validation.

---

## Firewall Outside Transit Addressing

VLAN 901 provides the shared outside segment between EDGE1, FW1, and FW2 through OS1.

Subnet:

```text
198.51.100.0/29
```

| Device | Role | IP Address | Notes |
|---|---|---|---|
| EDGE1 | Firewall outside next hop | `198.51.100.1/29` | Default gateway for firewall outside |
| FW1/FW2 active outside IP | ASA failover active address | `198.51.100.2/29` | Used by whichever firewall is active |
| FW1/FW2 standby outside IP | ASA failover standby address | `198.51.100.3/29` | Used by standby firewall |
| Available | Reserved | `198.51.100.4-198.51.100.6` | Future use |
| Broadcast | N/A | `198.51.100.7` | Broadcast address |

Design note:

OS1 is Layer 2 only. It provides the outside transit segment but does not route traffic.

---

## Firewall Inside Transit Addressing

VLAN 900 provides the inside firewall transit network between the firewall HA pair and CORE1/CORE2.

Subnet:

```text
10.10.90.0/24
```

| Device | Role | IP Address | Notes |
|---|---|---|---|
| CORE1/CORE2 HSRP VIP | Core-side firewall next hop | `10.10.90.1/24` | Firewall route to campus summary points here |
| CORE1 SVI | VLAN 900 SVI | `10.10.90.2/24` | CORE1 firewall transit SVI |
| CORE2 SVI | VLAN 900 SVI | `10.10.90.3/24` | CORE2 firewall transit SVI |
| FW1/FW2 active inside IP | ASA failover active address | `10.10.90.254/24` | Core default routes point here |
| FW1/FW2 standby inside IP | ASA failover standby address | `10.10.90.253/24` | Standby firewall inside address |

Design intent:

- CORE1 and CORE2 route outbound traffic toward `10.10.90.254`.
- FW1/FW2 route internal campus traffic toward `10.10.90.1`.
- CORE1 and CORE2 share VLAN 900 across the inter-core trunk.
- FW1 connects inside toward CORE1.
- FW2 connects inside toward CORE2.

---

## Firewall Failover / State Link Addressing

The firewall failover/state link is a dedicated connection between FW1 and FW2.

Planned combined failover/state subnet:

```text
10.255.255.0/30
```

| Device | Role | IP Address | Notes |
|---|---|---|---|
| FW1/FW2 failover active IP | ASA failover/state link | `10.255.255.1/30` | Active unit address |
| FW1/FW2 failover standby IP | ASA failover/state link | `10.255.255.2/30` | Standby unit address |

Design notes:

- This subnet is not a campus user network.
- This subnet should not be advertised or used for normal routing.
- This link exists only for firewall failover/state communication.
- The design may be adjusted later if failover and state links are separated.

---

## Management Addressing

VLAN 50 is used for network device management.

Subnet:

```text
10.10.50.0/24
```

| Device | Management Address | Notes |
|---|---|---|
| VLAN 50 HSRP VIP | `10.10.50.1` | Default gateway for management VLAN |
| CORE1 | `10.10.50.2` | CORE1 management/SVI address |
| CORE2 | `10.10.50.3` | CORE2 management/SVI address |
| ASW1 | `10.10.50.11` | Access switch management SVI |
| ASW2 | `10.10.50.12` | Access switch management SVI |
| ASW3 | `10.10.50.13` | Access switch management SVI |
| EDGE1 | TBD | May use routed interface or future management design |
| OS1 | TBD | Management optional |
| FW1/FW2 | TBD | Firewall management approach to be finalized |
| INFRA1 | `10.10.30.10` | Managed through server/infrastructure VLAN in v1 |

Design note:

Management access behavior should be finalized in `docs/12-management-plan.md`.

---

## Infrastructure Addressing

VLAN 30 is used for centralized infrastructure services.

Subnet:

```text
10.10.30.0/24
```

| Device | Address | Role |
|---|---|---|
| VLAN 30 HSRP VIP | `10.10.30.1` | Default gateway |
| CORE1 | `10.10.30.2` | CORE1 SVI |
| CORE2 | `10.10.30.3` | CORE2 SVI |
| INFRA1 | `10.10.30.10` | DHCP server |
| DNS1 | `10.10.30.20` | Future DNS server |
| SYSLOG1 | `10.10.30.21` | Future syslog server |
| NTP1 | `10.10.30.22` | Future NTP server |

Only INFRA1 is planned for v1. DNS1, SYSLOG1, and NTP1 are reserved for possible future service modules.

---

## DHCP Range Plan

### VLAN 10 - USERS

| Item | Value |
|---|---|
| Subnet | `10.10.10.0/24` |
| Default gateway | `10.10.10.1` |
| DHCP range | `10.10.10.100-10.10.10.199` |
| Reserved range | `10.10.10.2-10.10.10.99` |
| Future reserved range | `10.10.10.200-10.10.10.254` |

### VLAN 20 - ADMIN

| Item | Value |
|---|---|
| Subnet | `10.10.20.0/24` |
| Default gateway | `10.10.20.1` |
| DHCP range | `10.10.20.100-10.10.20.199` |
| Reserved range | `10.10.20.2-10.10.20.99` |
| Future reserved range | `10.10.20.200-10.10.20.254` |

### VLAN 30 - SERVERS_INFRA

| Item | Value |
|---|---|
| Subnet | `10.10.30.0/24` |
| Default gateway | `10.10.30.1` |
| DHCP range | Optional / deferred |
| Reserved range | `10.10.30.2-10.10.30.99` |
| Future reserved range | `10.10.30.100-10.10.30.254` |

### VLAN 40 - PRINTERS_IOT

| Item | Value |
|---|---|
| Subnet | `10.10.40.0/24` |
| Default gateway | `10.10.40.1` |
| DHCP range | `10.10.40.100-10.10.40.199` |
| Reserved range | `10.10.40.2-10.10.40.99` |
| Future reserved range | `10.10.40.200-10.10.40.254` |

### VLAN 50 - MGMT

| Item | Value |
|---|---|
| Subnet | `10.10.50.0/24` |
| Default gateway | `10.10.50.1` |
| DHCP range | None |
| Reserved range | `10.10.50.2-10.10.50.254` |

Management devices should use static addressing in v1.

---

## Routing-Relevant Addressing Summary

| Path | Network | Notes |
|---|---|---|
| ISP1 to EDGE1 | `203.0.113.0/30` | Simulated provider handoff |
| EDGE1 to FW outside | `198.51.100.0/29` | Outside firewall transit through OS1 |
| FW inside to CORE1/CORE2 | `10.10.90.0/24` | Inside firewall transit VLAN |
| Campus internal summary | `10.10.0.0/16` | Internal campus summary route |
| Firewall failover/state | `10.255.255.0/30` | Dedicated firewall HA link |

Expected routing direction:

| Device | Expected Route Behavior |
|---|---|
| ISP1 | Route outside firewall transit/public simulation back toward EDGE1 |
| EDGE1 | Default route toward ISP1; connected route to outside firewall transit |
| FW1/FW2 | Default route toward EDGE1; route campus summary toward CORE HSRP VIP |
| CORE1/CORE2 | Default route toward active firewall inside IP |
| INFRA1 | Default route toward VLAN 30 HSRP VIP |

---

## Reserved Addressing

### Future Internal Services

| Service | Reserved Address | VLAN |
|---|---|---|
| DNS1 | `10.10.30.20` | VLAN 30 |
| SYSLOG1 | `10.10.30.21` | VLAN 30 |
| NTP1 | `10.10.30.22` | VLAN 30 |
| Monitoring collector | `10.10.30.30` | VLAN 30 |

### Future External / Security Tooling

External tools such as ELK/OpenSearch, Suricata, and heavier monitoring platforms are expected to run outside CML in Proxmox if added later.

Their addressing should be documented when an external connector design is created.

---

## IPv6

IPv6 is out of scope for v1.

Future IPv6 design should be documented separately if added.

---

## Design Notes

- Internal campus addressing uses `10.10.0.0/16`.
- Documentation-safe public test ranges are used for ISP and outside firewall simulation.
- Campus VLANs use predictable `/24` networks.
- HSRP VIPs use `.1` where possible.
- CORE1 uses `.2` and CORE2 uses `.3` where possible.
- INFRA1 uses `10.10.30.10`.
- ASA active/standby addressing uses active and standby IPs per firewall interface.
- VLAN 900 uses `10.10.90.0/24` instead of `10.10.900.0/24`.
- VLAN 901 uses `198.51.100.0/29` and remains local to OS1.
- VLAN 998 and VLAN 999 should not have routed gateways.

---

## Validation or Success Criteria

The addressing plan is successful when:

- Every routed VLAN has a documented subnet.
- HSRP VIPs are documented for routed campus VLANs.
- CORE1 and CORE2 SVI addresses are documented.
- Firewall active and standby interface addresses are documented.
- DHCP ranges are documented for client VLANs.
- Static infrastructure addresses are reserved.
- Transit networks are clearly separated from client VLANs.
- VLAN 998 and VLAN 999 have no routed gateway.
- The addressing plan supports the routing, firewall, DHCP, and verification plans.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
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