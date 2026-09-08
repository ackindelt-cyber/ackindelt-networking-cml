# Known Limitations

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Reference  

---

## Purpose

This document identifies the known limitations of the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to clearly separate intentional v1 design boundaries from future improvements, platform constraints, and features that are not yet implemented.

---

## Scope

### In Scope

- Intentional v1 limitations
- Cisco CML platform limitations
- Topology limitations
- Firewall limitations
- Routing limitations
- Layer 2 and access-layer limitations
- Services limitations
- Management and monitoring limitations
- Security-hardening limitations
- Future improvement areas

### Out of Scope

- Detailed future implementation plans
- Final device configurations
- Verification command output
- Troubleshooting procedures
- Multi-site design details
- Production hardening standards

---

## Summary

Talos Solutions Enterprise Campus v1 is designed to be a complete and validated enterprise campus foundation, not a fully mature production enterprise network.

The v1 lab intentionally focuses on:

- Single-campus design
- Firewall HA
- Dual collapsed core/distribution switches
- Layer 2 access switches
- LACP uplinks
- HSRP gateway redundancy
- STP/HSRP alignment
- Centralized DHCP
- Static routing
- PAT
- Baseline verification
- Failure testing

Several features are intentionally deferred so the base topology can be built, validated, documented, and finished before adding more complexity.

---

## Topology Limitations

### Single Campus Only

V1 models one enterprise campus.

Not included in v1:

- Branch office
- Second campus
- Data center site
- WAN cloud
- Site-to-site connectivity
- Multi-site routing
- Inter-site failover

Design note:

Multi-campus expansion is a strong candidate for v2.

---

### Single ISP

V1 uses one simulated ISP connection.

Limitation:

- No provider redundancy
- No dual-homed internet edge
- No ISP failover testing
- No BGP provider peering

Design note:

Dual ISP and BGP should be deferred to a future edge/WAN module.

---

### Single Edge Router

V1 uses one customer edge router, EDGE1.

Limitation:

- EDGE1 is a single point of failure.
- No edge router HA.
- No dual-edge routing.
- No floating static routes or edge failover.

Design note:

The v1 focus is the campus and firewall/core design, not WAN edge redundancy.

---

### Single Outside Transit Switch

V1 uses one outside transit switch, OS1.

Limitation:

- OS1 is a single point of failure for the outside firewall segment.
- No redundant outside switch pair.
- No outside switch failover testing.

Design note:

OS1 represents a realistic outside firewall handoff switch, but v1 uses a single instance to keep the outside edge simple.

---

## Firewall Limitations

### Firewall HA Included, But Not Full Perimeter Design

V1 includes an active/standby ASAv firewall pair.

However, v1 does not include a complete enterprise perimeter design.

Not included in v1:

- DMZ
- Public services
- Inbound NAT
- Site-to-site VPN
- Remote access VPN
- Dual ISP firewall edge
- Complex security zones
- Full firewall policy matrix
- IDS/IPS tuning

Design note:

Firewall HA is included because it is central to the enterprise edge design. More advanced firewall services are deferred.

---

### ASAv Platform Constraints

The lab uses ASAv virtual firewalls.

Known or expected constraints may include:

- Feature differences from physical ASA appliances
- CML image limitations
- Interface behavior differences compared to hardware
- Resource limits based on the CML/Proxmox environment
- Failover behavior that may differ from production hardware deployments

Design note:

Any confirmed ASAv-specific behavior should be documented in:

```text
cml/cml-notes.md
```

---

### No DMZ

V1 does not include a DMZ.

Limitation:

- No public-facing internal service segment
- No inbound NAT testing
- No outside-to-DMZ policy testing
- No DMZ-to-inside policy testing

Design note:

A DMZ would be a good future firewall expansion after baseline v1 is stable.

---

## Routing Limitations

### Static Routing Only

V1 uses static routing.

Not included in v1:

- OSPF
- EIGRP
- BGP
- Route redistribution
- Dynamic failover routing
- Route summarization beyond static summary routes
- Advanced route filtering

Design note:

Static routing keeps v1 controlled and easier to validate. Dynamic routing can be added later after the base design is proven.

---

### No VRFs

V1 does not include VRFs.

Limitation:

- No separate routing tables
- No management VRF
- No guest/user/server route separation through VRF-lite
- No multi-tenant routing simulation

Design note:

VRFs are valuable but should not be added until the base campus design is stable.

---

### No Policy-Based Routing

V1 does not use policy-based routing.

Limitation:

- Traffic forwarding follows normal routing tables.
- No source-based path selection.
- No application-specific routing.
- No special routing policy for management or security tools.

Design note:

PBR is not needed for the baseline enterprise campus design.

---

## Layer 2 and Access-Layer Limitations

### Traditional STP-Based Access Redundancy

V1 uses traditional Layer 2 redundancy with STP.

Limitation:

- Some redundant paths may be blocked by STP.
- Access uplinks are not all forwarding at the same time for every VLAN.
- No multi-chassis EtherChannel is used.

Not included in v1:

- StackWise Virtual
- VSS
- vPC
- MLAG
- EVPN/VXLAN
- Fabric-based access

Design note:

This is intentional. IOSvL2 supports traditional switching concepts well, and the design is appropriate for v1.

---

### No Access-Layer Security Hardening

V1 does not include full access-layer security hardening.

Deferred features:

- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- 802.1X
- Voice VLANs
- MACsec

Design note:

These features are valuable, but they should be added after VLANs, trunks, HSRP, STP, DHCP, routing, firewall HA, and PAT are stable.

---

### No Wireless

V1 does not include wireless networking.

Not included:

- Wireless LAN controller
- Access points
- SSIDs
- WPA/802.1X wireless authentication
- Guest wireless VLANs
- Wireless client roaming

Design note:

Wireless is outside the current v1 scope.

---

## Services Limitations

### DHCP Is Centralized But Not Redundant

V1 uses INFRA1 as a centralized DHCP server.

Limitation:

- INFRA1 is a single DHCP server.
- No DHCP failover.
- No DHCP redundancy.
- No split scopes.
- No Windows/Linux production DHCP service.

Design note:

The goal is to demonstrate centralized DHCP and relay, not production DHCP resiliency.

---

### DNS Not Included in Baseline v1

V1 does not require a DNS server.

Limitation:

- No internal name resolution
- No internal domain functionality
- No DNS records
- No DNS forwarding or caching
- No dynamic DNS

Design note:

DNS is a good future service module. A lightweight Linux DNS node inside CML would be appropriate later.

---

### Syslog Not Included in Baseline v1

V1 does not include centralized syslog.

Limitation:

- Device logs are not centrally collected.
- No log retention.
- No log search.
- No external logging pipeline.
- No SIEM/logging integration.

Design note:

A lightweight internal syslog server is a good future CML service. ELK/OpenSearch should likely run externally in Proxmox if added later.

---

### NTP Not Included in Baseline v1

V1 does not include NTP.

Limitation:

- Device timestamps may not be synchronized.
- Logs may be harder to correlate.
- Time-based validation is less realistic.

Design note:

NTP is a good candidate for a future services module.

---

## Management Limitations

### Limited Management Hardening

V1 includes a management VLAN plan, but not full management hardening.

Deferred features:

- SSH-only management enforcement
- Management ACLs
- TACACS+
- RADIUS
- SNMPv3
- Centralized logging
- Configuration backup
- Role-based access control
- Jump host / bastion host
- Certificate-based management

Design note:

V1 should create the management addressing foundation without pretending full production management security is complete.

---

### No Out-of-Band Management

V1 uses in-band management concepts only.

Not included:

- Dedicated out-of-band management network
- Terminal server
- Dedicated management switch
- Separate management VRF
- Dedicated management firewall path

Design note:

Out-of-band management is not necessary for the first version of the lab.

---

## Monitoring and Operations Limitations

### No Monitoring Stack

V1 does not include a full monitoring stack.

Deferred tools and features:

- SNMP polling
- NetFlow/IPFIX
- Grafana
- Prometheus
- ELK/OpenSearch
- Splunk
- Network dashboards
- Alerting
- Config backup tooling

Design note:

Monitoring and operations tooling should be added after baseline network behavior is verified.

---

### No Suricata / IDS

V1 does not include Suricata or another IDS.

Limitation:

- No passive traffic inspection
- No IDS alerts
- No security event correlation
- No packet inspection dashboard

Design note:

Suricata is a good future security monitoring module. It should likely run externally in Proxmox and connect to CML through an external connector or intentional traffic path.

---

## Security Policy Limitations

### Basic Firewall Policy Only

V1 uses basic inside/outside firewall behavior.

Limitation:

- No detailed inter-VLAN ACL matrix
- No application-aware policy
- No user-based policy
- No DMZ policy
- No inspection tuning beyond what is required for baseline testing

Design note:

The baseline firewall goal is to prove routed firewall placement, PAT, and stateful behavior.

---

### No Internal Segmentation Policy

V1 does not enforce a detailed internal segmentation policy.

Limitation:

- User, admin, server, printer/IoT, and management VLANs may be routable through the core unless ACLs are added later.
- No strict east-west traffic control in baseline v1.
- No security policy between every VLAN pair.

Design note:

Internal segmentation should be added after baseline routing, DHCP, and firewall behavior are stable.

---

## Automation Limitations

### No Automation in Baseline v1

V1 is built manually.

Not included:

- Ansible configuration deployment
- pyATS validation
- Automated testing
- CI/CD pipeline
- GitHub Actions
- Automated config backup
- Automated documentation generation

Design note:

Automation is a high-value future addition after the manual build is complete and verified.

---

## CML / Virtual Lab Limitations

### Virtual Device Behavior

Because the lab runs in Cisco CML, behavior may differ from physical hardware.

Potential limitations:

- Interface naming differences
- Feature support gaps
- ASAv limitations
- IOSvL2 limitations
- Performance limits
- Boot time and resource constraints
- Differences from Catalyst/Nexus/ASA hardware platforms

Design note:

CML is appropriate for learning, design validation, and portfolio demonstration, but it is not a perfect replacement for production hardware.

---

### Resource Limits

The full topology may require meaningful CPU, RAM, and disk resources.

Potential constraints:

- Multiple IOSvL2 switches
- Two ASAv firewalls
- Multiple endpoint nodes
- CML host performance
- Proxmox resource allocation

Design note:

If performance becomes an issue, endpoint count or optional nodes may be reduced.

---

## Documentation Limitations

### Draft State

Most documents begin as draft planning artifacts.

Limitation:

- Some details may change during implementation.
- Interface numbering will not be final until CML topology build.
- Port-channel member interfaces will not be final until topology build.
- Verification output will not exist until the lab is configured and tested.

Design note:

Documents should be updated as the build becomes real. The documentation should reflect the final validated state before v1 is considered complete.

---

## Intentional Deferred Features

The following are intentionally deferred from v1:

- Dual ISP
- Dual edge routers
- BGP
- DMZ
- VPN
- Dynamic routing
- VRFs
- Policy-based routing
- Complex ACL segmentation
- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- 802.1X
- Voice VLANs
- TACACS/RADIUS
- SNMP
- NetFlow
- Syslog
- NTP
- DNS
- ELK/OpenSearch
- Suricata
- Monitoring dashboards
- Automation
- Multi-site design

---

## Design Notes

Known limitations are not failures.

They define the boundary of v1 and help keep the project controlled. A finished, validated v1 is more valuable than an unfinished lab that tries to include every possible feature.

Future improvements should be tracked in:

```text
docs/15-future-roadmap.md
```

---

## Validation or Success Criteria

This limitations document is successful when:

- Intentional v1 limitations are documented.
- Deferred features are clearly separated from baseline v1 scope.
- Platform limitations are acknowledged.
- The reader can understand what v1 does and does not attempt to prove.
- Future work is captured without expanding v1 uncontrollably.
- The document helps present the project as scoped and deliberate rather than incomplete.

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
- `docs/06-addressing-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/12-management-plan.md`
- `docs/13-build-order.md`
- `docs/15-future-roadmap.md`
- `docs/11-dhcp-plan.md`
- `docs/16-required-runbooks.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |