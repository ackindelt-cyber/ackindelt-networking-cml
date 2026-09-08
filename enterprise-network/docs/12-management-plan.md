# Management Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the management network design for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document the role of the management VLAN, planned management addressing, device management behavior, and future management services such as SSH, ACLs, TACACS/RADIUS, syslog, SNMP, NetFlow, and monitoring.

---

## Scope

### In Scope

- Management VLAN purpose
- Management addressing plan
- Device management reachability
- Access switch management behavior
- Core switch management behavior
- Firewall management considerations
- Edge and outside switch management considerations
- Future management service direction

### Out of Scope

- Full production management hardening
- TACACS/RADIUS implementation
- SNMP implementation
- NetFlow implementation
- Syslog implementation
- ELK/OpenSearch integration
- Out-of-band management network
- Bastion/jump host design
- Final device configuration output
- Verification command output

---

## Summary

The Talos Solutions Enterprise Campus v1 lab uses VLAN 50 as the in-band management VLAN for network infrastructure devices.

VLAN 50 provides a dedicated management network for switches and future infrastructure management services. Access switches use management SVIs in VLAN 50 and point their default gateway to the VLAN 50 HSRP VIP on CORE1/CORE2.

Management services are intentionally limited in v1. The primary goal is to create a clean addressing and reachability foundation that can support future management features.

---

## Management Design Standards

| Item | Standard |
|---|---|
| Management VLAN | VLAN 50 / MGMT |
| Management subnet | `10.10.50.0/24` |
| Gateway | CORE1/CORE2 HSRP VIP |
| HSRP VIP | `10.10.50.1` |
| CORE1 SVI | `10.10.50.2` |
| CORE2 SVI | `10.10.50.3` |
| Access switch management | VLAN 50 SVI |
| Access switch default gateway | `10.10.50.1` |
| DHCP for management | No |
| Management addressing | Static |
| Out-of-band management | Deferred |

---

## Management VLAN

### VLAN

| VLAN | Name | Purpose |
|---:|---|---|
| 50 | MGMT | Network device management |

### Subnet

```text
10.10.50.0/24
```

### Gateway

The VLAN 50 gateway should be the HSRP VIP shared by CORE1 and CORE2.

```text
10.10.50.1
```

Design intent:

- CORE1 and CORE2 provide the management VLAN gateway.
- Access switches use VLAN 50 for management SVIs.
- Infrastructure management traffic is separated from user, admin, server, printer/IoT, and transit VLANs.
- VLAN 50 should not be used as a general endpoint VLAN.

---

## Planned Management Addressing

| Device | Planned Management Address | Management Method | Notes |
|---|---|---|---|
| CORE1 | `10.10.50.2` | VLAN 50 SVI | Core management and HSRP member |
| CORE2 | `10.10.50.3` | VLAN 50 SVI | Core management and HSRP member |
| ASW1 | `10.10.50.11` | VLAN 50 SVI | Layer 2 switch management |
| ASW2 | `10.10.50.12` | VLAN 50 SVI | Layer 2 switch management |
| ASW3 | `10.10.50.13` | VLAN 50 SVI | Layer 2 switch management |
| EDGE1 | TBD | Routed or future management design | Management approach not finalized |
| OS1 | TBD | Console/local-only for v1 | Do not extend internal management VLANs to OS1 without a deliberate future design |
| FW1 | TBD | Inside or management interface | Firewall management approach not finalized |
| FW2 | TBD | Inside or management interface | Firewall management approach not finalized |
| INFRA1 | `10.10.30.10` | Server/infrastructure VLAN | Not in VLAN 50 for v1 |

---

## Core Switch Management

CORE1 and CORE2 provide the management VLAN gateway using SVIs and HSRP.

Expected behavior:

- CORE1 has an SVI in VLAN 50.
- CORE2 has an SVI in VLAN 50.
- CORE1 and CORE2 share the VLAN 50 HSRP VIP.
- Network devices use the VLAN 50 HSRP VIP as their default gateway where needed.
- STP root and HSRP active placement should align with the VLAN 50 preferred core.

Planned VLAN 50 role:

| Device | Address | Role |
|---|---|---|
| CORE1 | `10.10.50.2` | VLAN 50 SVI |
| CORE2 | `10.10.50.3` | VLAN 50 SVI |
| HSRP VIP | `10.10.50.1` | Management VLAN gateway |

---

## Access Switch Management

ASW1, ASW2, and ASW3 are Layer 2 access switches.

Expected behavior:

- Each access switch has one management SVI in VLAN 50.
- Each access switch uses `10.10.50.1` as its default gateway.
- Access switches should not route user VLANs.
- Access switches should not have SVIs for normal client VLANs.
- Management reachability depends on VLAN 50 being carried over the core-to-access trunks.

Expected access switch management settings:

| Device | Management VLAN | Management IP | Default Gateway |
|---|---:|---|---|
| ASW1 | 50 | `10.10.50.11` | `10.10.50.1` |
| ASW2 | 50 | `10.10.50.12` | `10.10.50.1` |
| ASW3 | 50 | `10.10.50.13` | `10.10.50.1` |

---

## EDGE1 Management

EDGE1 management is not finalized in v1.

Possible approaches:

| Option | Description | V1 Fit |
|---|---|---|
| In-band through routed interfaces | Manage EDGE1 using one of its routed interface addresses | Simple |
| Add management loopback | Use a loopback address for management reachability | Possible later |
| Dedicated management VLAN/interface | Add EDGE1 to VLAN 50 or separate management path | More complex |

Initial recommendation:

Use existing routed interface reachability during early v1 build and defer a polished EDGE1 management design until the base topology is stable.

---

## OS1 Management

OS1 is the outside transit switch between EDGE1 and the firewall pair.

OS1 management is optional in v1.

Possible approaches:

| Option | Description | V1 Fit |
|---|---|---|
| No management IP | OS1 operates as a simple outside transit switch | Simple |
| Local management only | Configure hostname/basic access but no routed management path | Good for v1 |
| Dedicated future management path | Add management reachability through a deliberate future design | More complex; should not leak internal VLANs into the outside segment |

Initial recommendation:

Keep OS1 simple in v1. OS1 exists to provide the shared outside firewall segment, not to demonstrate management design.

---

## Firewall Management

Firewall management is not finalized in v1.

Possible approaches:

| Option | Description | Notes |
|---|---|---|
| In-band inside management | Manage FW1/FW2 through the inside interface | Simple and common in labs |
| Dedicated management interface | Use ASAv management interface if useful | May require additional topology decisions |
| VLAN 50 path | Allow management from the management VLAN to the firewall inside interface | Likely future direction |
| Defer management access | Configure firewall locally during v1 | Simplest for initial build |

Initial recommendation:

Use local console/configuration during the initial firewall HA build. Add structured firewall management later after routing, HA, and PAT are verified.

---

## INFRA1 Management

INFRA1 is located in VLAN 30, not VLAN 50.

Planned address:

```text
10.10.30.10
```

INFRA1 should use the VLAN 30 HSRP VIP as its default gateway:

```text
10.10.30.1
```

Design note:

INFRA1 is a simulated infrastructure server, not a network device management endpoint. It does not need to live in VLAN 50 for v1.

---

## Management Access Methods

### Console Access

Console access is expected during initial lab build.

Use cases:

- Initial configuration
- Recovery from misconfiguration
- Firewall HA testing
- Troubleshooting routing or VLAN failures

### SSH

SSH is a future management goal but may not be required for baseline v1.

Future SSH requirements may include:

- Hostname and domain configuration
- Local user accounts
- RSA key generation
- VTY line configuration
- Transport input SSH only
- Management ACLs
- Login banners

### Telnet

Telnet should not be used as a planned management method.

If temporary Telnet access is used for lab troubleshooting, it should be removed before final configs are saved.

---

## Management Security Intent

Full management hardening is deferred from v1, but the design should support it.

Future management controls may include:

- SSH-only access
- Local user authentication
- Management ACLs
- TACACS/RADIUS
- Centralized logging
- SNMPv3
- Configuration backup
- NTP
- Login banners
- Role-based access control

---

## Management ACL Direction

Management ACLs are deferred from baseline v1.

Future intent:

- Allow management access from trusted admin sources.
- Restrict management access from normal user VLANs.
- Restrict access to VTY lines.
- Restrict firewall management access.
- Document management sources and allowed protocols.

Possible future management source:

| Source | Purpose |
|---|---|
| VLAN 20 ADMIN | Admin workstation source VLAN |
| VLAN 50 MGMT | Network infrastructure management network |
| Future jump host | Centralized management access point |

---

## Future Management Services

### Syslog

Future syslog design may include:

- Lightweight internal syslog server in CML
- Network devices logging to SYSLOG1
- Possible forwarding to ELK/OpenSearch externally through Proxmox

Reserved future address:

```text
SYSLOG1: 10.10.30.21
```

### SNMP

SNMP is deferred from v1.

Future SNMP design may include:

- SNMPv3
- Monitoring server
- Interface and device health polling
- Restricted management ACLs

### NetFlow

NetFlow is deferred from v1.

Future NetFlow design may include:

- Export from core or edge devices
- External collector
- Traffic visibility dashboard

### NTP

NTP is deferred from v1.

Future NTP design may include:

- Internal NTP server
- Device time synchronization
- Accurate logs and event correlation

Reserved future address:

```text
NTP1: 10.10.30.22
```

### TACACS/RADIUS

Centralized AAA is deferred from v1.

Future AAA design may include:

- TACACS+ for network device administration
- RADIUS for future access control or 802.1X testing
- Local fallback accounts

---

## Management Traffic Expectations

Expected v1 management reachability:

| Source | Destination | Expected Result |
|---|---|---|
| CORE1 | ASW1 management IP | Reachable if VLAN 50/trunks are correct |
| CORE1 | ASW2 management IP | Reachable if VLAN 50/trunks are correct |
| CORE1 | ASW3 management IP | Reachable if VLAN 50/trunks are correct |
| ASW1 | VLAN 50 HSRP VIP | Reachable |
| ASW2 | VLAN 50 HSRP VIP | Reachable |
| ASW3 | VLAN 50 HSRP VIP | Reachable |
| INFRA1 | VLAN 50 devices | Should route through CORE1/CORE2 if allowed |
| User VLANs | Management devices | Not restricted in baseline v1 unless ACLs are added later |

Design note:

Baseline v1 may allow broader management reachability than a hardened production design. That limitation should be documented honestly.

---

## Management Verification Expectations

Expected verification commands may include:

### On CORE1 / CORE2

```text
show ip interface brief
show standby brief
show vlan brief
show interfaces trunk
show spanning-tree vlan 50
ping 10.10.50.11
ping 10.10.50.12
ping 10.10.50.13
```

### On Access Switches

```text
show ip interface brief
show vlan brief
show interfaces trunk
show spanning-tree vlan 50
show running-config | include ip default-gateway
ping 10.10.50.1
ping 10.10.50.2
ping 10.10.50.3
```

### On INFRA1

```text
ping 10.10.50.1
ping 10.10.50.11
ping 10.10.50.12
ping 10.10.50.13
```

Detailed output should be captured in the verification documents, not in this design document.

---

## Deferred Management Features

The following management features are deferred from v1:

- SSH hardening
- Management ACLs
- TACACS/RADIUS
- SNMP
- NetFlow
- Syslog server
- NTP server
- Configuration backup
- Monitoring dashboards
- Out-of-band management
- Jump host / bastion host
- Certificate-based management

---

## Design Notes

- VLAN 50 is the dedicated management VLAN.
- Access switches use VLAN 50 SVIs for management.
- Access switches use the VLAN 50 HSRP VIP as their default gateway.
- INFRA1 remains in VLAN 30, not VLAN 50.
- OS1 management should stay simple in v1.
- Firewall management should be finalized after HA, routing, and PAT are stable.
- Full management hardening is deferred.
- V1 should build a clean foundation for future operational maturity.

---

## Validation or Success Criteria

The management plan is successful when:

- VLAN 50 purpose is documented.
- Management addressing is documented for core and access switches.
- Access switch default gateway behavior is documented.
- Static management addressing is documented.
- Deferred management services are clearly identified.
- Firewall management is acknowledged as an open design item.
- OS1 management is treated as optional.
- Management verification expectations are documented.
- The plan supports future SSH, ACL, AAA, syslog, SNMP, NetFlow, NTP, and monitoring modules.

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
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `verification/06-management-verification.md`
- `configs/CORE1.cfg`
- `configs/CORE2.cfg`
- `configs/ASW1.cfg`
- `configs/ASW2.cfg`
- `configs/ASW3.cfg`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |