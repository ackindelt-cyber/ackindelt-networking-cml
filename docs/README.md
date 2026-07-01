# Documentation

Supporting documentation for the Cisco CML lab portfolio.

This directory contains runbooks, templates, and supplemental documentation used to keep the repository organized, repeatable, and easier to review.

The goal of this area is to support practical network engineering workflow: configure, verify, troubleshoot, document, and reuse.

---

## Purpose

The documentation in this directory supports the lab work by providing:

* Operational runbooks for common networking technologies
* Reusable documentation templates
* Supplemental context for reviewing or maintaining the repository
* Consistent structure for future module and enterprise-network documentation

This documentation is written for practical use, not academic explanation.

---

## Directory Structure

```text
docs/
other/
runbooks/
  configuration/
templates/
```

### `other/`

Supplemental documentation and repository context.

This folder contains supporting material that does not fit directly into a module, runbook, or template. It can include review guidance, portfolio context, or other repository-level notes.

### `runbooks/`

Operational references for configuration, verification, and troubleshooting.

Runbooks are intended to document reusable workflows that apply across modules and future integrated lab designs. A good runbook should help explain how to approach a technology, what to verify, and where to look when expected behavior is not present.

### `templates/`

Reusable documentation formats.

Templates are used to keep lab and runbook documentation consistent as the repository grows.

---

## Current Focus

The current focus of this directory is maintaining standardized documentation that supports existing modules and version 1.0 of the planned `enterprise-network` lab.

Runbooks should capture more than configuration syntax. They should explain the expected behavior, useful verification commands, common failure points, and practical troubleshooting logic.

---

## Documentation Standards

Documentation in this folder should be:

* Clear enough to support review or handoff
* Practical enough to be useful during lab validation
* Consistent with the structure used across the repository
* Honest about lab scope, assumptions, and limitations

The goal is not to over-document every detail. The goal is to capture the information that would matter when building, validating, troubleshooting, or reusing the work later.

A good runbook should help someone quickly answer what the expected state is, what should be checked first, what output proves the behavior, and what failure points are most likely.

---

## Status

This directory is actively maintained as runbooks, templates, and supporting documentation are refined.

Current priority:

* Maintain reusable operational references
* Keep documentation aligned with the current repository structure
* Support existing modules with consistent documentation patterns
* Support the first version of the planned `enterprise-network` lab
