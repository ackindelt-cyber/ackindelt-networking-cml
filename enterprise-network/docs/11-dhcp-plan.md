# DHCP Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Design  

---

## Purpose

This document defines the DHCP design for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to document the role of INFRA1 as the centralized DHCP server, identify which VLANs receive DHCP service, define DHCP relay behavior, document planned DHCP pools, and define validation expectations.

---

## Scope

### In Scope

- INFRA1 DHCP server role
- DHCP relay placement
- DHCP pools for client VLANs
- Default gateway assignment using HSRP VIPs
- Reserved and excluded address ranges
- DHCP behavior for users, admin, printers/IoT, and infrastructure VLANs
- DHCP validation expectations

### Out of Scope

- DNS server implementation
- Production DHCP redundancy
- Windows DHCP services
- Linux DHCP services
- DHCP failover
- Dynamic DNS
- IPv6 DHCP
- DHCP snooping
- Final device configuration output
- Verification command output

---

## Summary

The Talos Solutions Enterprise Campus v1 lab uses INFRA1 as a centralized DHCP server.

INFRA1 is connected to VLAN 30, the `SERVERS_INFRA` VLAN. CORE1 and CORE2 provide Layer 3 gateway services for campus VLANs and relay DHCP requests from client VLANs to INFRA1.

Clients should receive the HSRP VIP for their VLAN as the default gateway.

Example:

```text
VLAN 10 client
  -> DHCP request
  -> CORE1/CORE2 SVI relay
  -> INFRA1 DHCP server
  -> DHCP lease with default gateway 10.10.10.1
```

DHCP is included in v1 because it demonstrates a common enterprise infrastructure pattern: centralized DHCP with relay from routed access VLANs.

---

## DHCP Design Standards

| Item | Standard |
|---|---|
| DHCP server | INFRA1 |
| DHCP server platform | IOSv |
| DHCP server VLAN | VLAN 30 / SERVERS_INFRA |
| DHCP relay location | CORE1/CORE2 SVIs |
| Client default gateway | HSRP VIP for each VLAN |
| DHCP redundancy | Deferred |
| DNS server option | Optional / future |
| Domain name option | Optional / future |
| DHCP snooping | Deferred |

---

## DHCP Server

### Device

| Device | Role | Address |
|---|---|---|
| INFRA1 | Centralized DHCP server | `10.10.30.10/24` |

### Default Gateway

INFRA1 should use the VLAN 30 HSRP VIP as its default gateway.

```text
10.10.30.1
```

### Design Notes

- INFRA1 should be inside the campus network.
- INFRA1 should not connect outside the firewall.
- INFRA1 provides DHCP service only; DNS, syslog, and NTP are future services.
- INFRA1 uses IOSv for v1 to avoid requiring a custom Linux or Windows server image.
- DHCP redundancy is not included in v1.

---

## DHCP Relay Design

DHCP relay should be configured on the routed SVIs for VLANs that require DHCP service.

Relay target:

```text
10.10.30.10
```

Relay should be placed on CORE1 and CORE2 SVIs for client VLANs.

Expected relay VLANs:

| VLAN | Name | DHCP Relay Needed | Relay Target |
|---:|---|---|---|
| 10 | USERS | Yes | `10.10.30.10` |
| 20 | ADMIN | Yes | `10.10.30.10` |
| 30 | SERVERS_INFRA | No | Local DHCP server VLAN |
| 40 | PRINTERS_IOT | Yes | `10.10.30.10` |
| 50 | MGMT | No | Static management addressing |
| 900 | FW_TRANSIT | No | Static transit addressing |
| 901 | OUTSIDE_TRANSIT | No | Static transit addressing |
| 998 | NATIVE | No | No routed gateway |
| 999 | PARKING | No | No routed gateway |

Design note:

VLAN 30 does not require DHCP relay because INFRA1 is directly connected to VLAN 30.

---

## DHCP Gateway Assignment

DHCP clients should receive the HSRP VIP for their VLAN as the default gateway.

| VLAN | Name | DHCP Default Gateway |
|---:|---|---|
| 10 | USERS | `10.10.10.1` |
| 20 | ADMIN | `10.10.20.1` |
| 40 | PRINTERS_IOT | `10.10.40.1` |

The DHCP server should not hand out CORE1 or CORE2 physical SVI addresses as client default gateways.

Correct:

```text
default-router 10.10.10.1
```

Incorrect:

```text
default-router 10.10.10.2
default-router 10.10.10.3
```

---

## DHCP Pool Plan

### VLAN 10 - USERS

| Item | Value |
|---|---|
| Pool name | `VLAN10-USERS` |
| Subnet | `10.10.10.0/24` |
| Default gateway | `10.10.10.1` |
| DHCP range | `10.10.10.100-10.10.10.199` |
| Excluded low range | `10.10.10.1-10.10.10.99` |
| Excluded high / future reserved range | `10.10.10.200-10.10.10.254` |

Expected use:

- C1
- C2
- General user endpoint testing

---

### VLAN 20 - ADMIN

| Item | Value |
|---|---|
| Pool name | `VLAN20-ADMIN` |
| Subnet | `10.10.20.0/24` |
| Default gateway | `10.10.20.1` |
| DHCP range | `10.10.20.100-10.10.20.199` |
| Excluded low range | `10.10.20.1-10.10.20.99` |
| Excluded high / future reserved range | `10.10.20.200-10.10.20.254` |

Expected use:

- C3 if used as an admin endpoint
- Future management-access testing
- Future segmentation testing

---

### VLAN 40 - PRINTERS_IOT

| Item | Value |
|---|---|
| Pool name | `VLAN40-PRINTERS-IOT` |
| Subnet | `10.10.40.0/24` |
| Default gateway | `10.10.40.1` |
| DHCP range | `10.10.40.100-10.10.40.199` |
| Excluded low range | `10.10.40.1-10.10.40.99` |
| Excluded high / future reserved range | `10.10.40.200-10.10.40.254` |

Expected use:

- PRN1 DHCP testing
- Simulated printer/IoT endpoint testing
- VLAN 40 DHCP relay validation

Design note:

Printers are often statically addressed or assigned DHCP reservations in production environments. For v1, PRN1 will use DHCP so VLAN 40 relay behavior can be validated. Static printer addressing and DHCP reservations are deferred.

---

## Static / Non-DHCP VLANs

### VLAN 30 - SERVERS_INFRA

VLAN 30 contains INFRA1 and future service nodes.

Expected addressing:

| Device | Address | Notes |
|---|---|---|
| INFRA1 | `10.10.30.10` | Static DHCP server address |
| DNS1 | `10.10.30.20` | Future |
| SYSLOG1 | `10.10.30.21` | Future |
| NTP1 | `10.10.30.22` | Future |

DHCP is not required for VLAN 30 in v1.

---

### VLAN 50 - MGMT

VLAN 50 should use static addressing.

Expected management addresses:

| Device | Address |
|---|---|
| HSRP VIP | `10.10.50.1` |
| CORE1 | `10.10.50.2` |
| CORE2 | `10.10.50.3` |
| ASW1 | `10.10.50.11` |
| ASW2 | `10.10.50.12` |
| ASW3 | `10.10.50.13` |

DHCP should not be used for network device management in v1.

---

### Transit and Special VLANs

The following VLANs should not use DHCP:

| VLAN | Name | Reason |
|---:|---|---|
| 900 | FW_TRANSIT | Static firewall/core transit addressing |
| 901 | OUTSIDE_TRANSIT | Static edge/firewall outside addressing |
| 998 | NATIVE | No routed gateway |
| 999 | PARKING | No active hosts |

---

## DNS and Domain Options

DNS is not implemented as a required v1 service.

Possible DHCP options:

| Option | V1 Status | Notes |
|---|---|---|
| DNS server | Optional / deferred | Future DNS1 may use `10.10.30.20` |
| Domain name | Optional / deferred | Possible future domain: `talos.local` |

For v1, DHCP validation should focus on:

- Correct IP address
- Correct subnet mask
- Correct default gateway
- Correct relay behavior
- Correct VLAN assignment

DNS functionality should not be required for baseline v1 success.

---

## DHCP Configuration Intent

The final IOS DHCP configuration should generally include:

- DHCP service enabled on INFRA1
- Excluded address ranges
- DHCP pool per client VLAN
- Correct network statement
- Correct default-router value
- Optional DNS/domain values if configured
- Static default route from INFRA1 toward VLAN 30 HSRP VIP

Example intent:

```text
INFRA1 provides VLAN 10 DHCP leases.
CORE1/CORE2 relay VLAN 10 DHCP requests to INFRA1.
Clients receive 10.10.10.1 as the default gateway.
```

Final commands should be stored in:

```text
configs/INFRA1.cfg
configs/CORE1.cfg
configs/CORE2.cfg
```

---

## DHCP Relay Configuration Intent

CORE1 and CORE2 should relay DHCP requests from client VLAN SVIs to INFRA1.

Expected relay target:

```text
10.10.30.10
```

SVIs expected to use relay:

```text
Vlan10
Vlan20
Vlan40
```

SVIs not expected to use relay:

```text
Vlan30
Vlan50
Vlan900
```

Design note:

If DHCP is later added for additional VLANs, the relay plan should be updated before configuration changes are made.

---

## DHCP Traffic Flow

### Client Lease Process

Expected flow:

```text
Client broadcasts DHCPDISCOVER
  -> Access switch forwards within VLAN
  -> CORE1/CORE2 SVI receives broadcast
  -> SVI relays request to INFRA1
  -> INFRA1 selects correct DHCP pool
  -> INFRA1 replies through CORE1/CORE2
  -> Client receives lease
```

### Why Relay Is Required

DHCP clients use broadcast traffic to request an address.

Because INFRA1 is centralized in VLAN 30 and clients are in separate routed VLANs, CORE1 and CORE2 must relay DHCP requests across Layer 3 boundaries.

---

## DHCP Verification Expectations

Expected verification commands may include:

### On INFRA1

```text
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show running-config | section dhcp
show ip route
```

### On CORE1 / CORE2

```text
show running-config interface vlan 10
show running-config interface vlan 20
show running-config interface vlan 40
show ip interface brief
show standby brief
show ip route
```

### On Client Nodes

```text
ip addr
ip route
ping <default-gateway>
ping <INFRA1>
ping <external-test-address>
```

Detailed output should be captured in the verification documents, not in this design document.

---

## DHCP Failure Testing Expectations

DHCP-related failure testing may include:

- Client receives a valid address in VLAN 10.
- Client receives the correct default gateway.
- Client renews or obtains an address after HSRP failover.
- Client obtains an address when the non-preferred core is active.
- Client fails to obtain an address if relay is missing.
- Client fails to obtain an address if INFRA1 is unreachable.

Full failure testing should be documented in:

```text
verification/07-failure-testing.md
```

---

## Deferred DHCP Features

The following DHCP-related features are deferred from v1:

- DHCP redundancy
- DHCP failover
- Windows DHCP server
- Linux DHCP server
- Dynamic DNS updates
- DHCP snooping
- DHCP option tuning
- DHCP reservations for printers
- IPv6 DHCP

---

## Design Notes

- INFRA1 is the centralized DHCP server.
- CORE1 and CORE2 provide DHCP relay.
- Clients should receive HSRP VIPs as default gateways.
- VLAN 30 uses static addressing for infrastructure services.
- VLAN 50 uses static addressing for network management.
- Transit VLANs should not use DHCP.
- DNS is not required for v1 DHCP success.
- DHCP should be validated before moving into advanced services.

---

## Validation or Success Criteria

The DHCP plan is successful when:

- INFRA1 DHCP role is documented.
- DHCP relay placement is documented.
- DHCP pools are documented for required client VLANs.
- Excluded and reserved address ranges are documented.
- Clients receive addresses from the correct VLAN pool.
- Clients receive HSRP VIPs as default gateways.
- INFRA1 is reachable from routed client VLANs.
- DHCP works through CORE1/CORE2 relay.
- Static VLANs and transit VLANs are not incorrectly configured for DHCP.
- DNS remains clearly deferred from required v1 functionality.

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
- `verification/04-dhcp-verification.md`
- `verification/07-failure-testing.md`
- `configs/INFRA1.cfg`
- `configs/CORE1.cfg`
- `configs/CORE2.cfg`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |