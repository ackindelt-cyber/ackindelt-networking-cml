# Runbook — <Feature / Use Case>

## Overview

This runbook provides a structured process for configuring and validating <feature name>.

<Briefly explain what the feature does and why it is used. Keep this section focused on the operational purpose, not a full theory explanation.>

This runbook focuses on <primary implementation type or design>. The example topology uses <devices, VLANs, subnets, routing process, or other key design values>.

## Scope and Assumptions

This runbook assumes:

* <Assumption 1>
* <Assumption 2>
* <Assumption 3>
* <Assumption 4>
* <Assumption 5>

<Optional note about related designs or use cases that are outside the primary scope of this runbook.>

---

## Reference Design

This runbook uses <short description of the reference design>.

### Devices

| Device     | Role   | Purpose   |
| ---------- | ------ | --------- |
| <Device 1> | <Role> | <Purpose> |
| <Device 2> | <Role> | <Purpose> |

### Network Summary

| Network / VLAN / Segment | Purpose   | Subnet   | Gateway / Next Hop    |
| ------------------------ | --------- | -------- | --------------------- |
| <Value>                  | <Purpose> | <Subnet> | <Gateway or next hop> |

### Interface / Feature Summary

| Device     | Interface   | IP Address / Value | Feature Role | Notes   |
| ---------- | ----------- | ------------------ | ------------ | ------- |
| <Device 1> | <Interface> | <Value>            | <Role>       | <Notes> |
| <Device 2> | <Interface> | <Value>            | <Role>       | <Notes> |

### Feature Design Values

| Item             | Value   |
| ---------------- | ------- |
| <Design value 1> | <Value> |
| <Design value 2> | <Value> |
| <Design value 3> | <Value> |

**Note:** Add any feature-specific explanation here, such as router ID selection, HSRP virtual MAC calculation, OSPF area placement, NAT inside/outside role, DHCP relay path, or ACL placement.

---

## Prerequisites and Pre-Checks

Before configuring <feature name>, confirm that the required baseline is already working.

### Prerequisites

* [ ] <Required baseline item 1>
* [ ] <Required baseline item 2>
* [ ] <Required baseline item 3>
* [ ] <Required baseline item 4>
* [ ] <Required baseline item 5>

### Baseline Verification

Run these commands before applying the feature configuration.

```bash
<show command 1>
<show command 2>
<show command 3>
<show command 4>
```

### Expected Baseline Results

* [ ] <Expected result 1>
* [ ] <Expected result 2>
* [ ] <Expected result 3>
* [ ] <Expected result 4>
* [ ] <Expected result 5>

---

## Configuration Procedure

Use this procedure to configure <feature name>.

### Configuration Notes

* <Important configuration note 1>
* <Important configuration note 2>
* <Important configuration note 3>
* <Important configuration note 4>

---

### Step 1 — Configure <Device / Role / Component>

```bash
enable
configure terminal

<configuration commands>

end
write memory
```

### Step 2 — Configure <Device / Role / Component>

```bash
enable
configure terminal

<configuration commands>

end
write memory
```

### Step 3 — Confirm Configuration Was Applied

```bash
<show running-config command>
<feature summary command>
```

Expected result:

* [ ] <Expected configuration result 1>
* [ ] <Expected configuration result 2>
* [ ] <Expected configuration result 3>
* [ ] <Expected configuration result 4>

---

## Post-Configuration Validation

Use this section to confirm that <feature name> is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Interface or Service State

Run on <device or devices>.

```bash
<show command>
```

Expected results:

* [ ] <Expected result 1>
* [ ] <Expected result 2>
* [ ] <Expected result 3>

---

### Step 2 — Verify Feature Summary State

Run on <device or devices>.

```bash
<show command>
```

Expected results:

* [ ] <Expected result 1>
* [ ] <Expected result 2>
* [ ] <Expected result 3>

---

### Step 3 — Verify Detailed Feature State

Run on <device or devices>.

```bash
<show command>
```

Expected results:

* [ ] <Expected result 1>
* [ ] <Expected result 2>
* [ ] <Expected result 3>
* [ ] <Expected result 4>

---

### Step 4 — Verify Dependencies

Run on <device or devices>.

```bash
<show command>
<show command>
```

Expected results:

* [ ] <Expected dependency result 1>
* [ ] <Expected dependency result 2>
* [ ] <Expected dependency result 3>

---

### Step 5 — Verify Client or Data-Plane Behavior

Run from <client, router, switch, or test endpoint>.

```bash
<test command>
```

Expected results:

* [ ] <Expected test result 1>
* [ ] <Expected test result 2>
* [ ] <Expected test result 3>

---

## Functional Validation

Use this section to confirm that the feature works under the intended operating condition.

For redundancy features, this section may be renamed to **Failover Validation**.
For non-redundancy features, use this section for client testing, routing behavior, NAT translation testing, DHCP lease testing, ACL match testing, or other functional checks.

### Step 1 — Confirm Initial State

```bash
<show command>
```

Expected results:

* [ ] <Expected initial state 1>
* [ ] <Expected initial state 2>
* [ ] <Expected initial state 3>

---

### Step 2 — Run Functional Test

```bash
<test command>
```

Expected results:

* [ ] <Expected functional result 1>
* [ ] <Expected functional result 2>
* [ ] <Expected functional result 3>

---

### Step 3 — Verify Final State

```bash
<show command>
<show command>
```

Expected results:

* [ ] <Expected final state 1>
* [ ] <Expected final state 2>
* [ ] <Expected final state 3>

---

## If Validation Fails

If post-configuration or functional validation does not match the expected results, stop and verify the baseline before making additional changes.

Run these checks first:

```bash
<show command 1>
<show command 2>
<show command 3>
<show command 4>
```

Confirm the following:

* [ ] <Baseline check 1>
* [ ] <Baseline check 2>
* [ ] <Baseline check 3>
* [ ] <Baseline check 4>
* [ ] <Baseline check 5>

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

Removing or changing <feature name> can interrupt <traffic type, service, routing behavior, gateway reachability, or other dependency>.

Before removing or changing the configuration, confirm:

* An approved replacement or restored design exists.
* The affected traffic or service behavior is understood.
* The original configuration is documented.
* Expected device roles, paths, or dependencies are documented.
* A maintenance or lab validation window is available if traffic interruption is possible.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value              |
| ---------------- | ------------------ |
| Author           | Aaron Kindelt      |
| Category         | <Category>         |
| Technology       | <Technology>       |
| Applies To       | <Device types>     |
| Primary Use Case | <Primary use case> |
| Version          | 1.0                |
| Last Updated     | <YYYY-MM-DD>       |
