# ackindelt-networking-cml

A Cisco Modeling Labs portfolio focused on practical network engineering, technology validation, verification, troubleshooting, and documentation.

This repository is built around a simple idea: individual technologies should be understood and validated before they are combined into larger network designs.

The labs use focused Cisco CML topologies to isolate and validate specific network behaviors. Some module topologies are intentionally simplified so the technology can be observed clearly.

The production-oriented emphasis is on the workflow: design intent, verification, troubleshooting, documentation, and reuse of validated patterns.

---

## Purpose

This repository demonstrates how I approach network engineering work in a lab environment:

* Build focused technology labs
* Validate expected behavior with show commands and traffic tests
* Capture configurations and topology files
* Document what worked, what broke, and what mattered
* Reuse validated patterns in a larger enterprise-style network

The goal is to show practical engineering workflow, not just configuration syntax.

---

## Quick Review Path

This README is the best starting point for reviewing the repository. It explains the purpose, structure, current status, and how the lab artifacts are intended to be evaluated.

For a quick technical review:

* Start with this `README.md`.
* Read [`docs/other/prospective-employer.md`](docs/other/prospective-employer.md) for guidance on how to evaluate the repository.
* Open one complete module under `modules/` to review the lab structure, configurations, topology files, verification notes, and reflections.
* Review `docs/runbooks/` to see reusable configuration and validation workflows.

The repository is intended to support technical discussion. It is not just a collection of finished configs.

---

## Repository Structure

```text
README.md
docs/
enterprise-network/
modules/
```

### `docs/`

Supporting documentation used across the repository.

Current subfolders:

* `other/` — supplemental documentation and repository context
* `runbooks/` — operational references, verification workflows, and troubleshooting procedures
* `templates/` — reusable documentation formats

This area contains operational runbooks and reusable templates that support both individual modules and the future enterprise network lab.

### `enterprise-network/`

One planned enterprise-style Cisco CML network.

This area contains planning documentation for the integrated enterprise-style network. The integrated topology has not been built yet. Version 1.0 will be built after the supporting modules and runbooks have been standardized.

The goal is to combine validated module patterns into one cumulative network environment that grows over time.

### `modules/`

Focused technology validation labs.

Modules are intentionally scoped around individual technologies or operational patterns. Some module topologies are simplified or purpose-built for learning and validation rather than realism.

A polished module generally includes a README, device configs, topology files, verification notes, and lab reflections when applicable. These modules provide the technical foundation for the larger enterprise network build.

---

## Current Focus

The current focus of this repository is standardizing existing modules and operational runbooks that support version 1.0 of the planned `enterprise-network` lab.

The modules validate individual technologies in focused CML labs. The runbooks capture reusable configuration, verification, and troubleshooting workflows. Together, they provide the foundation for the first integrated enterprise-style network build.

---

## Tools and Environment

Primary tools used in this repository:

* Cisco Modeling Labs
* Cisco IOSv / IOSvL2 images
* Wireshark
* Git and GitHub
* Markdown documentation
* YAML topology exports

The environment is lab-based, but the workflow is intended to reflect real operational habits: build, verify, troubleshoot, document, and improve.

---

## Scope

This repository intentionally does not try to model every enterprise edge case.

It is focused on:

* Practical networking fundamentals
* Clear validation steps
* Reusable configuration patterns
* Troubleshooting logic
* Documentation that supports review and handoff

The labs are production-oriented in workflow, but they are still lab environments. Some module topologies are intentionally simplified to make specific technologies easier to observe and validate.

Any real deployment would require additional review for business requirements, security policy, monitoring, change control, hardware standards, addressing plans, and operational ownership.

---

## Status

This repository is actively maintained and expanded as labs are created, runbooks are refined, and validated patterns are incorporated into the larger enterprise-style network.

---

## License

This project is licensed under the Creative Commons Attribution 4.0 International (CC BY 4.0) license.

You are free to share and adapt the material, provided appropriate credit is given.

See the [LICENSE](LICENSE) file for full details.

---

## Author

**Aaron Kindelt**
Network / Infrastructure Engineering
Focused on practical network design, operational validation, troubleshooting, and clear documentation.
