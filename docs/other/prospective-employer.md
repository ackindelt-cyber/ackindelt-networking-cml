# Repository Review Guide

This document provides guidance for evaluating the technical work in this repository.

For a broad overview of the repository purpose, structure, and current focus, start with the root [`README.md`](../../README.md). This document is focused on how to review the artifacts efficiently.

---

## Recommended Review Path

For the quickest review, start with the root `README.md`, then use this guide to evaluate the technical artifacts.

A practical review path is:

1. Read the root `README.md` for repository purpose, structure, and current status.
2. Open one complete module under `modules/`.
3. Review the module README, configurations, topology files, verification notes, and lab reflections.
4. Review `docs/runbooks/` to see reusable configuration and validation workflows.
5. Review `enterprise-network/` for planned integration of validated patterns.

A strong module should make it clear:

* What technology or behavior was being validated
* Why the topology was built that way
* What configurations were applied
* How the result was verified
* What broke or required troubleshooting
* What was learned from the lab

Recommended artifacts to review inside a module:

```text
README.md
configs/
topology/
verification/
lab_reflections.md
```

A polished module should show more than a final working configuration. It should make clear what was built, how the behavior was verified, and how issues were investigated when the result did not match the expected behavior.

---

## Evaluating Modules

Modules are focused technology validation labs. Some module topologies are intentionally simplified or purpose-built so a specific technology can be isolated and observed clearly.

When reviewing a module, look for:

* Complete and consistently named device configs
* A readable topology diagram or CML topology export
* Verification commands that prove the intended behavior
* Expected or observed output where useful
* Reflections that explain what mattered, what failed, or what would change in a larger design

A module should not be judged only by how realistic the topology looks. Many are deliberately constrained so the technology behavior is easier to test and understand.

---

## Evaluating Runbooks

Runbooks are located under `docs/runbooks/`.

They are intended to capture reusable configuration, verification, and troubleshooting workflows.

When reviewing a runbook, look for:

* Clear configuration sequence
* Useful verification commands
* Expected behavior
* Common failure points
* Troubleshooting decision logic
* Notes that would help someone operate or validate the technology later

A good runbook should be useful beyond a single lab. It should explain how to approach the technology, not just list commands.

---

## Enterprise Network Status

The `enterprise-network/` lab is planned. The integrated topology has not been built yet, but initial planning documentation is being developed.

Version 1.0 will be built after the supporting modules and runbooks have been standardized. The intent is to combine validated module patterns into one cumulative enterprise-style CML environment.

Until that work begins, design capability should be evaluated through:

* The quality of individual modules
* The consistency of the documentation
* The verification evidence
* The usefulness of the runbooks
* The ability to explain assumptions, constraints, and tradeoffs

---

## What This Repository Demonstrates

This repository is intended to demonstrate practical network engineering habits:

* Build focused lab environments
* Validate behavior with observable evidence
* Troubleshoot from device output instead of assumption
* Document configurations and verification steps clearly
* Create reusable operational references
* Prepare validated patterns for larger integrated designs

The emphasis is on workflow: build, verify, troubleshoot, document, and improve.

---

## Scope Boundaries

This repository is intentionally scoped. It is designed to demonstrate lab-based network engineering workflow, not to replace production design review.

It is not intended to be:

* Vendor production documentation
* A complete enterprise reference architecture
* A certification cram guide
* A claim that every module topology is deployable as-is
* A replacement for real design review, change control, monitoring, security policy, or operational ownership

The labs are production-oriented in workflow, but they remain lab environments.

---

## Useful Discussion Areas

Useful review or interview discussion areas include:

* Why a topology was designed a certain way
* Whether the verification evidence proves the intended behavior
* What assumptions were made
* What would change in a larger network
* Which failure points are most likely in production
* How individual modules should be combined into the future enterprise network

The repository is meant to support technical discussion, not just display finished configs.
