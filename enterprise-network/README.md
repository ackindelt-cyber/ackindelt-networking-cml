# Enterprise Network

One planned enterprise-style Cisco CML network.

This directory will contain the integrated lab environment built from validated module patterns and supporting runbooks.

The integrated topology is planned and will be built after the supporting modules and runbooks have been standardized.

---

## Purpose

The purpose of this lab is to combine individual networking technologies into one cumulative environment.

The modules in this repository validate technologies in focused lab topologies. The enterprise network will use those validated patterns to build a larger topology that better reflects how routing, switching, services, redundancy, segmentation, and security controls interact together.

---

## Planned Version 1.0 Focus

Version 1.0 is intended to establish the first integrated network baseline.

The initial build will likely focus on:

* Layer 2 access and distribution behavior
* VLANs and trunks
* Inter-VLAN routing
* Routing between network segments
* Basic network services
* Redundancy where appropriate
* Clear verification and documentation

The exact scope may change as supporting modules and runbooks are standardized.

---

## Expected Structure

As the lab is built, this directory will likely include:

* `configs/` — device configurations for the integrated topology
* `topology/` — CML topology export and diagrams
* `verification/` — validation commands, expected behavior, and observed results
* `docs/` — design notes, assumptions, and supporting documentation specific to this lab
* `implementation/` — enterprise-specific implementation notes for technologies added to the integrated topology

Folder structure may change as the lab develops.

---

## Current Status

Planned.

The integrated topology has not been built yet. Current work is focused on standardizing the supporting modules, runbooks, and planning documentation that will support the first version of this environment.
