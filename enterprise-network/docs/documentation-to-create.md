# Enterprise Campus Documentation Planning List

## Purpose

This document is a planning list for the documentation that may be included in the Talos Solutions Enterprise Campus project.

This is not the final documentation structure. It is a working list to help decide what documentation is useful, what belongs in v1, and what can be deferred.

---

## Core Project Documentation

* `README.md`

  * Project overview
  * Talos Solutions scenario
  * High-level topology
  * Technologies demonstrated
  * What is included in v1
  * What is out of scope
  * Links to key docs

* `initial-planning.md`

  * Initial design intent
  * Device list
  * Link map
  * Feature scope
  * Deferred features
  * Initial CML build checklist

* `known-limitations.md`

  * Single edge router
  * Single firewall
  * No firewall HA in v1
  * No dual ISP
  * No dynamic routing in v1
  * No full monitoring/security stack
  * CML platform limitations

* `future-roadmap.md`

  * Future DNS/syslog/NTP services
  * Access-layer hardening
  * Firewall HA
  * DMZ
  * Monitoring/logging integration
  * Multi-site expansion
  * Custom Linux service images

---

## Topology and Design Documentation

* `topology.md`

  * Device roles
  * Physical link map
  * Logical topology summary
  * CML node types
  * Interface count estimate
  * Topology assumptions

* `interface-map.md`

  * Device
  * Interface
  * Connected device
  * Connected interface
  * Link purpose
  * Port-channel membership
  * Notes

* `port-channel-plan.md`

  * Port-channel IDs
  * Member interfaces
  * Local device
  * Remote device
  * LACP mode
  * Trunk/native/allowed VLAN details
  * Purpose

* `vlan-plan.md`

  * VLAN ID
  * VLAN name
  * Purpose
  * Gateway location
  * HSRP VIP
  * Preferred core
  * STP root primary/secondary
  * Notes

* `addressing-plan.md`

  * WAN transit addressing
  * Edge-to-firewall addressing
  * Firewall-to-core transit addressing
  * VLAN subnets
  * SVI IPs
  * HSRP VIPs
  * DHCP ranges
  * Reserved addresses
  * Management IPs

---

## Device-Specific Design Documentation

* `edge-router-plan.md`

  * EDGE1 role
  * ISP-facing link
  * Firewall-facing link
  * Static default route
  * Out-of-scope edge features

* `firewall-plan.md`

  * FW1 role
  * Inside/outside design
  * PAT placement
  * Basic policy intent
  * Firewall-to-core transit behavior
  * Known firewall limitations

* `core-design-plan.md`

  * CORE1/CORE2 role
  * SVIs
  * HSRP
  * STP/HSRP alignment
  * Inter-VLAN routing
  * DHCP relay
  * Default path to firewall

* `access-layer-plan.md`

  * ASW1/ASW2/ASW3 role
  * LACP uplinks
  * Access port behavior
  * PortFast/BPDU Guard
  * Native VLAN
  * Parking VLAN
  * Deferred access security features

* `dhcp-plan.md`

  * INFRA1 role
  * DHCP pools
  * Excluded ranges
  * Default gateways handed to clients
  * DHCP relay placement
  * DHCP validation notes

* `management-plan.md`

  * Management VLAN purpose
  * Management IPs
  * Device management reachability
  * Future SSH/ACL/TACACS/syslog/SNMP plans

---

## Configuration Documentation

* `configs/ISP1.cfg`
* `configs/EDGE1.cfg`
* `configs/FW1.cfg`
* `configs/CORE1.cfg`
* `configs/CORE2.cfg`
* `configs/ASW1.cfg`
* `configs/ASW2.cfg`
* `configs/ASW3.cfg`
* `configs/INFRA1.cfg`

Potential supporting docs:

* `config-notes.md`

  * Important implementation notes
  * Platform-specific caveats
  * Commands that differ between IOSv, IOSvL2, and ASAv

* `config-change-log.md`

  * Major config changes
  * Why changes were made
  * Date/phase of change

---

## Verification Documentation

* `baseline-verification.md`

  * Device status checks
  * Interface checks
  * Trunk checks
  * EtherChannel checks
  * HSRP checks
  * STP checks
  * DHCP checks
  * PAT/firewall checks
  * End-to-end connectivity checks

* `failure-testing.md`

  * LACP member failure
  * Full port-channel failure
  * Access uplink failure
  * CORE1 failure
  * CORE2 failure
  * HSRP failover
  * STP path changes
  * Firewall-to-core path testing

* `dhcp-verification.md`

  * DHCP bindings
  * Client lease validation
  * DHCP relay validation
  * Correct default gateway validation

* `firewall-pat-verification.md`

  * PAT translations
  * Inside-to-outside connectivity
  * Outside-to-inside default deny behavior
  * Firewall route validation

* `management-verification.md`

  * VLAN 50 reachability
  * Management SVI status
  * Device-to-device management connectivity
  * Future SSH validation

* `command-reference.md`

  * Common verification commands
  * Expected command outputs
  * Troubleshooting command list

---

## Diagrams and Visual Artifacts

* `diagrams/topology-cml.png`

  * Screenshot/export of the CML topology

* `diagrams/logical-topology.png`

  * Clean logical topology diagram

* `diagrams/physical-link-map.png`

  * Physical/link-focused diagram

* `diagrams/stp-hsrp-overlay.png`

  * Optional diagram showing preferred STP roots and HSRP active gateways

* `diagrams/failure-paths.png`

  * Optional diagram showing expected failover paths

---

## CML Artifacts

* `cml/topology.yaml`

  * CML topology export

* `cml/notes.md`

  * CML-specific build notes
  * Node image versions
  * Known CML behavior
  * Any interface or image limitations discovered

---

## Optional Future Documentation

* `dns-plan.md`

  * Future DNS server design
  * Internal domain
  * DNS records
  * DHCP DNS integration

* `syslog-plan.md`

  * Future syslog server design
  * Device logging targets
  * Severity levels
  * Log validation

* `monitoring-plan.md`

  * Future ELK/OpenSearch/Splunk/Grafana integration
  * External connector design
  * Log forwarding path

* `security-hardening-plan.md`

  * Port security
  * DHCP snooping
  * Dynamic ARP Inspection
  * IP Source Guard
  * Management ACLs

* `multi-site-expansion-plan.md`

  * Future branch/campus expansion
  * WAN design
  * Site-to-site routing
  * Centralized services
  * Inter-site failure testing

---

## Possible v1 Minimum Documentation Set

For v1, the minimum useful documentation may be:

* `README.md`
* `initial-planning.md`
* `topology.md`
* `interface-map.md`
* `port-channel-plan.md`
* `vlan-plan.md`
* `addressing-plan.md`
* `configs/`
* `baseline-verification.md`
* `failure-testing.md`
* `known-limitations.md`
* `future-roadmap.md`

---

## Documentation Notes

The documentation should be useful first and polished second.

The goal is to show:

* What was designed
* Why it was designed that way
* How it was built
* How it was verified
* What failed or changed
* What is intentionally deferred

