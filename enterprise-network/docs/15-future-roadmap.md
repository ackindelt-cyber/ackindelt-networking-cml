# Future Roadmap

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Roadmap  

---

## Purpose

This document tracks future expansion ideas for the Talos Solutions Enterprise Campus lab.

The purpose of this document is to preserve useful future ideas without expanding the scope of v1 before the baseline enterprise campus network is built, verified, documented, and failure-tested.

---

## Scope

### In Scope

- Future service additions
- Future security hardening
- Future firewall and edge improvements
- Future monitoring and operations tooling
- Future automation
- Future multi-site expansion
- Future documentation improvements
- Possible version sequencing

### Out of Scope

- Final implementation details
- Final device configurations
- Immediate v1 build requirements
- Production deployment standards
- Detailed multi-site topology design
- Detailed automation code
- Detailed monitoring stack implementation

---

## Summary

Talos Solutions Enterprise Campus v1 focuses on building a stable, validated single-campus enterprise foundation.

Future versions should expand the lab in controlled phases rather than adding every possible feature to the initial build.

The preferred roadmap is:

```text
v1   - Single-campus enterprise foundation
v1.1 - Internal infrastructure services
v1.2 - Access-layer security hardening
v1.3 - Monitoring and operations
v1.4 - Firewall and edge expansion
v2   - Multi-site / branch expansion
v3   - Automation and validation pipeline
```

The roadmap should remain flexible. Version numbers may change as the lab develops.

---

## Roadmap Principles

- Finish and validate v1 before expanding.
- Add features in logical modules.
- Do not add future features just because they are interesting.
- Prefer additions that strengthen the enterprise network story.
- Keep each expansion testable and documented.
- Avoid turning the lab into an unfinished monster topology.
- Document design decisions before implementation.
- Capture verification and failure testing for each major expansion.

---

## Current v1 Foundation

The current v1 design focuses on:

- Single campus topology
- Simulated ISP
- Single customer edge router
- Outside transit switch
- Active/standby ASAv firewall pair
- Dual collapsed core/distribution switches
- Layer 2 access switches
- LACP uplinks
- HSRP gateway redundancy
- STP and HSRP alignment
- Centralized DHCP using INFRA1
- DHCP relay
- Static routing
- Firewall PAT
- Management VLAN foundation
- Baseline verification
- Failure testing

V1 should be completed before future modules are added.

---

## Proposed Version Roadmap

| Version | Theme | Primary Goal |
|---|---|---|
| v1 | Enterprise campus foundation | Build and validate the base single-campus network |
| v1.1 | Infrastructure services | Add DNS, syslog, NTP, and possible lightweight service nodes |
| v1.2 | Access-layer hardening | Add switchport and Layer 2 security controls |
| v1.3 | Monitoring and operations | Add logging, SNMP/NetFlow, dashboards, and operational visibility |
| v1.4 | Firewall and edge expansion | Add DMZ, inbound NAT, VPN, or edge improvements |
| v2 | Multi-site expansion | Add branch/campus WAN design |
| v3 | Automation | Add Ansible, pyATS, automated validation, and repeatable deployment |

---

## v1.1 - Infrastructure Services

### Goal

Add lightweight internal services to make the enterprise lab feel more complete and operationally realistic.

### Candidate Additions

- DNS server
- Syslog server
- NTP server
- Internal web/application test server
- Possible Linux-based DHCP replacement
- Basic internal domain naming

### Preferred Placement

Lightweight infrastructure services should generally live inside CML.

Expected location:

```text
VLAN 30 - SERVERS_INFRA
```

Reserved addressing examples:

| Service | Planned Address | Notes |
|---|---|---|
| DNS1 | `10.10.30.20` | Future internal DNS server |
| SYSLOG1 | `10.10.30.21` | Future syslog server |
| NTP1 | `10.10.30.22` | Future NTP server |
| WEB1 | `10.10.30.30` | Future internal test service |

### Design Notes

- DNS and syslog are good candidates for lightweight Linux nodes inside CML.
- NTP is useful for log accuracy and event correlation.
- A Linux DHCP server can replace IOSv DHCP later if desired.
- These services should not block v1 completion.

---

## v1.2 - Access-Layer Security Hardening

### Goal

Add common enterprise access-layer protections after the baseline switching and routing design is stable.

### Candidate Additions

- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- BPDU Guard validation
- Root Guard
- Unused port hardening
- Management-plane restrictions
- Possible voice VLAN later

### Design Notes

These features are valuable, but they should be added after v1 is stable because they can introduce additional troubleshooting complexity.

The correct sequence is:

```text
First prove VLANs, trunks, STP, HSRP, DHCP, routing, firewall HA, and PAT.
Then harden the access layer.
```

### Expected Artifacts

Possible future documents:

```text
docs/security-hardening-plan.md
verification/access-security-verification.md
troubleshooting/access-security-troubleshooting.md
```

---

## v1.3 - Monitoring and Operations

### Goal

Add operational visibility and logging to the enterprise lab.

### Candidate Additions

- Use or expand centralized syslog from the infrastructure services module
- SNMPv3
- NetFlow/IPFIX
- Use or expand NTP from the infrastructure services module
- Device logging standards

### Preferred Placement

Lightweight services may live inside CML:

```text
SYSLOG1
NTP1
```

Heavier platforms should likely run externally in Proxmox:

```text
ELK/OpenSearch
Splunk
Grafana/Prometheus
Large log storage
Long-term monitoring stack
```

### External Connector Direction

If heavier monitoring is added, use a CML external connector or intentional routed path to connect CML to Proxmox-hosted services.

Conceptual model:

```text
CML Enterprise Network
        |
External Connector / Routed Path
        |
Proxmox Operations Tooling
```

### Design Notes

Monitoring should be added after v1 is stable so there is a working network to monitor.

---

## v1.4 - Firewall and Edge Expansion

### Goal

Expand perimeter capabilities after the firewall HA pair and baseline PAT behavior are proven.

### Candidate Additions

- DMZ
- Inbound NAT
- Public-facing test service
- More detailed firewall policy
- Site-to-site VPN
- Remote access VPN if supported
- Firewall management access design
- More advanced inspection policy
- Dual outside transit switch design
- Edge route tracking
- Floating static routes

### Possible DMZ Direction

A future DMZ could contain:

```text
WEB-DMZ1
PUBLIC-APP1
TEST-SERVICE1
```

Possible future DMZ VLAN:

```text
VLAN 100 - DMZ
```

### Design Notes

A DMZ is a strong future addition because it would let the lab demonstrate inbound NAT, firewall zone policy, and more realistic perimeter behavior.

However, it should not be added until baseline inside-to-outside behavior and firewall failover are validated.

---

## v2 - Multi-Site / Branch Expansion

### Goal

Expand Talos Solutions from a single campus into a multi-site enterprise network.

### Candidate Topology

```text
Talos Solutions HQ Campus
        |
Simulated WAN / ISP Cloud
        |
Talos Solutions Branch Office
```

Possible future sites:

- HQ Campus
- Branch Office
- Distribution Center
- Remote office
- Data center simulation

### Candidate Technologies

- Site-to-site VPN
- WAN routing
- OSPF
- EIGRP
- BGP
- Route summarization
- Floating static routes
- IP SLA tracking
- Centralized services
- Branch DHCP relay
- Local branch DHCP fallback
- Multi-site failure testing

### Design Notes

This is probably the strongest next major portfolio expansion after v1.

It would demonstrate broader network engineering skills than simply adding more services to the single-campus lab.

---

## v3 - Automation and Validation

### Goal

Add automation after the manual design is complete and validated.

### Candidate Additions

- Ansible configuration deployment
- Ansible configuration templates
- pyATS validation
- Automated show-command collection
- Automated pre-checks and post-checks
- GitHub Actions or local validation runner
- Change-control simulation
- Config backup automation
- Automated documentation support

### Suggested Automation Sequence

```text
1. Finish manual v1
2. Freeze final configs
3. Build Ansible inventory
4. Convert selected configs to templates
5. Automate simple repeatable tasks
6. Add pyATS validation
7. Compare manual and automated builds
```

### Design Notes

Automation will be more valuable after the manual build is finished because there will be a known-good design to automate against.

The goal should not be automation for its own sake. The goal should be repeatable deployment and validation.

---

## Security Monitoring Expansion

### Goal

Add security visibility after the core network and logging foundation are stable.

### Candidate Additions

- Suricata IDS
- External security monitoring VM
- Log forwarding to ELK/OpenSearch
- Firewall log collection
- Simulated attack or suspicious traffic generation
- Packet capture workflows
- Alert review documentation

### Preferred Placement

Suricata should likely run outside CML in Proxmox.

Reasoning:

- Better resource control
- Easier package updates
- Better log retention
- Easier integration with ELK/OpenSearch
- More realistic operations/security tooling placement

### Design Notes

Suricata needs an intentional traffic visibility path. Because CML IOSvL2 may not support traditional SPAN/port mirroring, the traffic path must be designed carefully.

This should be its own module, not a casual add-on.

---

## Services Expansion

### DNS

Preferred future placement:

```text
DNS1 inside CML on VLAN 30
```

Possible implementation:

- Lightweight Linux image
- dnsmasq
- Unbound
- BIND if needed later

Possible domain:

```text
talos.local
```

Example records:

```text
core1.talos.local
core2.talos.local
fw1.talos.local
fw2.talos.local
infra1.talos.local
```

### Syslog

Preferred future placement:

```text
SYSLOG1 inside CML on VLAN 30
```

Possible implementation:

- Lightweight Linux image
- rsyslog
- syslog-ng

### NTP

Preferred future placement:

```text
NTP1 inside CML on VLAN 30
```

Possible implementation:

- Linux NTP/chrony
- IOS-based NTP if simpler

### Design Notes

DNS, syslog, and NTP are good v1.1 candidates because they improve realism without requiring a full multi-site redesign.

---

## Management Expansion

### Candidate Additions

- SSH management
- Local user accounts
- Management ACLs
- Login banners
- SNMPv3
- TACACS+
- RADIUS
- Jump host
- Configuration backup
- Out-of-band management simulation
- Management VRF

### Design Notes

Management hardening should build on the VLAN 50 foundation.

A reasonable sequence:

```text
1. Enable SSH
2. Add local users
3. Restrict VTY access
4. Add management ACLs
5. Add syslog/NTP
6. Add SNMPv3
7. Add centralized AAA later
```

---

## Documentation Expansion

### Candidate Additions

- Final logical topology diagram
- Physical link diagram
- STP/HSRP overlay diagram
- Firewall HA diagram
- Failure path diagrams
- Verification evidence documents
- Troubleshooting notes
- Change-control simulation
- Interview-ready project summary
- Architecture decision records if needed later

### Design Notes

Documentation should grow as the lab becomes real.

Avoid creating empty documents that do not yet help explain, build, verify, or troubleshoot the network.

---

## Possible Future Folders

If future modules grow large enough, the project may eventually use module folders.

Possible structure:

```text
enterprise-network/
├── docs/
├── topology/
├── cml/
├── configs/
├── verification/
├── troubleshooting/
└── future-modules/
    ├── services/
    ├── security-hardening/
    ├── monitoring/
    ├── firewall-expansion/
    ├── multisite/
    └── automation/
```

This should not be added until needed.

---

## Portfolio Value Ranking

The following ranking reflects long-term portfolio value, not necessarily the exact implementation order:

| Priority | Expansion | Reason |
|---:|---|---|
| 1 | Multi-site / branch expansion | Strongest network engineering growth path |
| 2 | Monitoring and operations | Shows operational maturity |
| 3 | Access-layer security hardening | Strong network/security crossover |
| 4 | DNS/syslog/NTP services | Improves realism with manageable complexity |
| 5 | Firewall/DMZ expansion | Valuable but can become firewall-heavy |
| 6 | Automation | High-value after manual design is stable |

This ranking may change after v1 is built and reviewed.

---

## Deferred Feature List

The following features are intentionally deferred from v1 and may be considered later:

- DNS
- Syslog
- NTP
- SNMP
- NetFlow
- ELK/OpenSearch
- Splunk
- Grafana/Prometheus
- Suricata
- Port security
- DHCP snooping
- Dynamic ARP Inspection
- IP Source Guard
- Storm control
- 802.1X
- Voice VLANs
- Management ACLs
- TACACS/RADIUS
- DMZ
- Inbound NAT
- VPN
- Dual ISP
- Dual edge routers
- BGP
- OSPF/EIGRP in the integrated topology
- VRFs
- Policy-based routing
- Multi-site WAN
- Automation with Ansible
- Validation with pyATS

---

## Design Notes

Future work should support the enterprise story.

Good future additions should answer one of these questions:

- Does this make the network more realistic?
- Does this demonstrate useful network engineering skill?
- Does this improve verification or operations?
- Does this create a clearer interview story?
- Does this build naturally on v1?

If the answer is no, the idea should stay deferred.

---

## Validation or Success Criteria

The roadmap is successful when:

- Future ideas are captured without expanding v1 scope.
- Roadmap items are grouped logically.
- Future additions have a likely sequence.
- Deferred features are not forgotten.
- The roadmap supports career portfolio value.
- The project remains finishable.
- v1 remains focused on the baseline enterprise campus foundation.

---

## Open Questions

- None at this time

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/12-management-plan.md`
- `docs/13-build-order.md`
- `docs/14-known-limitations.md`
- `docs/11-dhcp-plan.md`
- `docs/16-required-runbooks.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |