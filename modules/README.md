# Modules

Focused Cisco CML labs used to validate individual networking technologies and operational patterns.

The goal is to isolate specific behaviors, verify them clearly, and document the results before reusing those patterns in a larger integrated design.

Some module topologies are simplified or purpose-built for validation. They are not presented as production network designs. The value is in showing what was built, how it was verified, and what was learned from the lab.

---

## Purpose

The modules in this directory support the larger portfolio by providing:

* Focused technology validation labs
* Device configurations used in each lab
* Topology diagrams and CML exports where available
* Verification commands and observed behavior
* Lab reflections based on what was built and tested

A polished module should show more than a final working configuration. It should make clear what was built, how the behavior was verified, and what issues were investigated when the result did not match the expected behavior.

---

## Directory Structure

```text
modules/
layer-2/
layer-3/
security/
services/
```

### `layer-2/`

Layer 2 switching technologies and behaviors.

This area is used for modules related to switching, VLANs, trunks, spanning tree, EtherChannel, and other Layer 2 behaviors.

### `layer-3/`

Layer 3 routing and gateway behavior.

This area is used for modules related to routing, route selection, dynamic routing protocols, and first-hop redundancy.

### `security/`

Network security and traffic-control labs.

This area is used for modules related to access control, traffic filtering, VPN concepts, and other security-adjacent network controls.

### `services/`

Supporting network services.

This area is used for modules related to network services such as DHCP, NAT, and other infrastructure functions that support routing and connectivity.

---

## Standard Module Format

A polished module generally includes:

```text
README.md
configs/
topology/
verification/
lab_reflections.md
```

Expected contents:

* `README.md` explains the purpose, topology, design intent, validation goals, and key takeaways.
* `configs/` contains device configurations used in the lab.
* `topology/` contains diagrams and CML topology exports where available.
* `verification/` contains useful show commands, test commands, and expected or observed behavior.
* `lab_reflections.md` captures what mattered, what broke, what was corrected, and what was learned.

Packet captures are included only when they add useful validation value.

Troubleshooting sections inside module README files are intended as quick-reference notes for that specific lab. They may include issues observed during validation or checks that are directly relevant to the module, but they are not intended to be exhaustive troubleshooting guides for the technology.

---

## Module Status

Module status is tracked inside each module README rather than through folder names.

Common status values:

* `Validated` — topology, configurations, and verification are complete
* `In Progress` — lab exists but documentation or validation is still being refined
* `Planned` — lab has not been built or is not ready for review
* `Archived` — retained for reference but no longer part of the active portfolio path

---

## Current Focus

The current focus of this directory is standardizing existing modules so they can support version 1.0 of the planned `enterprise-network` lab.

Priority work includes:

* Maintaining consistent folder and file names
* Keeping module READMEs clear and reviewable
* Confirming configs are complete and consistently named
* Adding useful verification notes where needed
* Keeping lab reflections honest and specific to what was observed
* Preparing validated module patterns for future enterprise-network integration

---

## How These Modules Should Be Used

These modules are building blocks.

Each one validates a specific technology or behavior in isolation. Once the behavior is understood and documented, the useful patterns can be used as reference material for runbooks and future enterprise-network implementation.

A module does not need to look like a full production topology to be useful. It needs to clearly show the technology, the expected behavior, the verification method, and the practical lessons from the lab.

## CML VLAN Export Limitation
 
Cisco CML topology exports and exported device configuration files may not preserve VLAN database state.
 
For VLAN-based labs, VLAN IDs, VLAN names, and intended VLAN design are documented in the module README. The "topology.yaml" file should be treated as the source of truth for CML node/link layout only. Exported device configuration files may include interface-level VLAN assignments, trunk settings, SVIs, and routing configuration, but they should not be relied on as the only source for VLAN database information.
 
When rebuilding or reviewing VLAN-based labs, use the module README VLAN tables and configuration notes as the authoritative reference for VLAN design.
