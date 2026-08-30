# Documentation Plan

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Planning / Reference  

---

## Purpose

This document defines the planned documentation structure for the Talos Solutions Enterprise Campus v1 lab.

The goal is to keep project documentation organized, consistent, and useful as the lab moves from planning to implementation, verification, troubleshooting, and future expansion.

---

## Scope

### In Scope

- Documentation folder structure
- Purpose of each major document group
- Planned document list
- Documentation standards
- Create-now vs create-later guidance
- Relationship between design docs, configs, diagrams, CML exports, verification, and troubleshooting notes

### Out of Scope

- Final technical design decisions
- VLAN and addressing details
- Device configurations
- Verification command output
- Troubleshooting procedures
- CML topology exports
- Network diagrams

---

## Summary

The Talos Solutions Enterprise Campus project should use a consistent documentation structure so the lab can be understood, built, validated, and expanded over time.

Documentation should support the project instead of becoming the project. Each document should have a clear purpose and should help explain, build, verify, troubleshoot, or improve the network.

The documentation should be organized into the following major areas:

- `docs/` - Planning, design, and decision documentation
- `topology/` - Diagrams and visual topology artifacts
- `cml/` - Cisco CML topology exports and CML-specific notes
- `configs/` - Final device configurations
- `verification/` - Validation plans, results, and command references
- `troubleshooting/` - Issue-specific troubleshooting notes

---

## Planned Folder Structure

```text
enterprise-network/
├── README.md
├── docs/
├── topology/
├── cml/
├── configs/
├── verification/
└── troubleshooting/
```

---

## Folder Purpose

| Folder | Purpose |
|---|---|
| `docs/` | Design, planning, assumptions, decisions, and roadmap documentation |
| `topology/` | CML screenshots, logical diagrams, physical diagrams, and overlay diagrams |
| `cml/` | CML topology export files and CML-specific notes |
| `configs/` | Final complete configurations for each network device |
| `verification/` | Baseline validation, feature validation, failure testing, and command references |
| `troubleshooting/` | Notes and procedures for diagnosing common lab issues |

---

## Planned Documents

### Root

| File | Purpose |
|---|---|
| `README.md` | Main project entry point and overview |

### `docs/`

| File | Purpose |
|---|---|
| `_document-template.md` | Standard template for project documentation |
| `00-project-background.md` | Explains the Talos Solutions name and project context |
| `01-initial-planning.md` | Captures the current v1 planning snapshot |
| `02-documentation-plan.md` | Defines the documentation structure and standards |
| `03-topology-and-device-roles.md` | Defines devices, CML node types, roles, and physical/logical topology |
| `04-design-decisions.md` | Captures major design choices and reasoning |
| `05-vlan-plan.md` | Defines VLAN IDs, names, purposes, HSRP roles, and STP alignment |
| `06-addressing-plan.md` | Defines IP addressing, subnets, SVIs, HSRP VIPs, and reserved ranges |
| `07-interface-map.md` | Maps physical interfaces between devices |
| `08-port-channel-plan.md` | Defines LACP bundles, member links, trunks, native VLANs, and allowed VLANs |
| `09-routing-plan.md` | Defines static routing behavior between ISP, edge, firewall, and core |
| `10-firewall-plan.md` | Defines FW1/FW2 HA, firewall interfaces, PAT, policy intent, and limitations |
| `11-dhcp-plan.md` | Defines INFRA1 DHCP service and DHCP relay behavior |
| `12-management-plan.md` | Defines the management VLAN and future management services |
| `13-build-order.md` | Defines the planned implementation sequence |
| `14-known-limitations.md` | Documents intentional limitations and platform constraints |
| `15-future-roadmap.md` | Tracks future expansion ideas |
| `16-required-runbooks.md` | Tracks module runbooks required to build the enterprise lab |

### `topology/`

| File | Purpose |
|---|---|
| `cml-topology-screenshot.png` | Screenshot of the topology as built in Cisco CML |
| `logical-topology.png` | Clean logical topology diagram |
| `physical-link-map.png` | Diagram focused on physical links and port relationships |
| `stp-hsrp-overlay.png` | Optional diagram showing STP root and HSRP active placement |

### `cml/`

| File | Purpose |
|---|---|
| `talos-enterprise-campus-v1.yaml` | Cisco CML topology export |
| `cml-notes.md` | Notes about CML version, node images, interface limits, and platform behavior |

### `configs/`

| File | Purpose |
|---|---|
| `ISP1.cfg` | Final ISP1 configuration |
| `EDGE1.cfg` | Final EDGE1 configuration |
| `FW1.cfg` | Final FW1 configuration |
| `CORE1.cfg` | Final CORE1 configuration |
| `CORE2.cfg` | Final CORE2 configuration |
| `ASW1.cfg` | Final ASW1 configuration |
| `ASW2.cfg` | Final ASW2 configuration |
| `ASW3.cfg` | Final ASW3 configuration |
| `INFRA1.cfg` | Final INFRA1 configuration |

### `verification/`

| File | Purpose |
|---|---|
| `00-verification-plan.md` | Defines the overall verification approach |
| `01-baseline-verification.md` | Captures baseline operational checks |
| `02-l2-verification.md` | Validates VLANs, trunks, STP, and EtherChannel |
| `03-l3-verification.md` | Validates SVIs, HSRP, routing, and reachability |
| `04-dhcp-verification.md` | Validates DHCP relay and client leases |
| `05-firewall-pat-verification.md` | Validates firewall routing and PAT behavior |
| `06-management-verification.md` | Validates management VLAN reachability |
| `07-failure-testing.md` | Validates redundancy and failover behavior |
| `command-reference.md` | Lists useful verification and troubleshooting commands |

### `troubleshooting/`

| File | Purpose |
|---|---|
| `common-issues.md` | General troubleshooting notes |
| `lacp-troubleshooting.md` | EtherChannel and LACP troubleshooting |
| `stp-hsrp-troubleshooting.md` | STP and HSRP troubleshooting |
| `dhcp-troubleshooting.md` | DHCP and relay troubleshooting |
| `firewall-troubleshooting.md` | Firewall, routing, and PAT troubleshooting |

---

## Documentation Standards

Project documents should generally follow this structure:

```text
Status
Purpose
Scope
Summary
Details
Design Notes
Validation or Success Criteria
Open Questions
Related Documents
Change Log
```

Not every document needs to be long. Some sections may be brief or marked as not applicable.

The goal is consistency, not unnecessary paperwork.

---

## Document Template

The standard project document template is stored at:

```text
docs/_document-template.md
```

Planning and design documents should generally follow this template unless a different format makes more sense for the document.

---

## Current Planning Documentation Set

The following documents are part of the current planning and design documentation pass:

- `README.md`
- `docs/_document-template.md`
- `docs/00-project-background.md`
- `docs/01-initial-planning.md`
- `docs/02-documentation-plan.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
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
- `docs/16-required-runbooks.md`

---

## Create Later

The following artifacts should be created after the CML topology is built, configured, or verified:

### `topology/`

- `cml-topology-screenshot.png`
- `logical-topology.png`
- `physical-link-map.png`
- `stp-hsrp-overlay.png`
- `firewall-ha-topology.png`

### `cml/`

- `talos-enterprise-campus-v1.yaml`
- `cml-notes.md`

### `configs/`

- `ISP1.cfg`
- `EDGE1.cfg`
- `OS1.cfg`
- `FW1.cfg`
- `FW2.cfg`
- `CORE1.cfg`
- `CORE2.cfg`
- `ASW1.cfg`
- `ASW2.cfg`
- `ASW3.cfg`
- `INFRA1.cfg`

### `verification/`

- `00-verification-plan.md`
- `01-baseline-verification.md`
- `02-l2-verification.md`
- `03-l3-verification.md`
- `04-dhcp-verification.md`
- `05-firewall-pat-verification.md`
- `06-management-verification.md`
- `07-failure-testing.md`
- `command-reference.md`

### `troubleshooting/`

- `common-issues.md`
- `lacp-troubleshooting.md`
- `stp-hsrp-troubleshooting.md`
- `dhcp-troubleshooting.md`
- `firewall-troubleshooting.md`

---

## Design Notes

The documentation should avoid becoming a collection of random notes.

Each document should support one or more of the following goals:

- Explain the design
- Capture a decision
- Support implementation
- Support verification
- Support troubleshooting
- Document limitations
- Plan future improvements

If a document does not help explain, build, verify, or operate the lab, it should be deferred or removed.

---

## Validation or Success Criteria

The documentation structure is successful when:

- A reader can understand the purpose of the project from the root `README.md`
- Design intent is captured in `docs/`
- Diagrams are separated from written design docs
- CML exports are separated from diagrams
- Final configurations are easy to find
- Verification evidence is organized and repeatable
- Troubleshooting notes are separate from design documents
- Future work is tracked without expanding v1 scope unnecessarily

---

## Open Questions

- None currently identified.

---

## Related Documents

- `README.md`
- `docs/_document-template.md`
- `docs/00-project-background.md`
- `docs/01-initial-planning.md`
- `docs/03-topology-and-device-roles.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`

---

## Change Log

| Date       | Change        |
|------------|---------------|
| 2026-08-30 | Initial draft |