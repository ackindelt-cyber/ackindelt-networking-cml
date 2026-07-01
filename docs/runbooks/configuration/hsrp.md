# Runbook — HSRP SVI Gateway Redundancy

## Overview

This runbook provides a structured process for configuring and validating Hot Standby Router Protocol (HSRP) on Layer 3 switch virtual interfaces.

HSRP provides default gateway redundancy by allowing two Layer 3 devices to share a virtual IP address. End hosts use the HSRP virtual IP as their default gateway. If the active gateway fails, the standby device can assume the active role and continue forwarding traffic for the subnet.

This runbook focuses on an SVI-based HSRP design using two distribution switches. The example topology uses VLAN 10, with D1 intended to operate as the active HSRP device and D2 intended to operate as the standby HSRP device.

## Scope and Assumptions

This runbook assumes:

* The environment uses SVIs for Layer 3 gateway services.
* VLANs and SVIs already exist before HSRP is configured.
* Layer 2 connectivity between the distribution switches is already operational.
* Both HSRP devices have unique real IP addresses in the target VLAN.
* End hosts use the HSRP virtual IP address as their default gateway.
* HSRP role placement is controlled with priority and preemption.

HSRP can also be implemented on routed LAN interfaces or router-on-a-stick subinterfaces, but those designs are outside the primary scope of this runbook.


---

## Reference Design

This runbook uses a two-device HSRP design with one active gateway and one standby gateway for VLAN 10.

### Devices

| Device | Role                  | Purpose                       |
| ------ | --------------------- | ----------------------------- |
| D1     | Distribution Switch 1 | Intended HSRP active gateway  |
| D2     | Distribution Switch 2 | Intended HSRP standby gateway |

### VLAN and Network

| VLAN | Name / Purpose | Subnet          | Default Gateway |
| ---- | -------------- | --------------- | --------------- |
| 10   | Users          | 192.168.10.0/24 | 192.168.10.1    |

### HSRP Interface Summary

| Device | Interface | Real IP Address | HSRP Group | Intended Role | Virtual IP   |
| ------ | --------- | --------------- | ---------- | ------------- | ------------ |
| D1     | Vlan10    | 192.168.10.2/24 | 10         | Active        | 192.168.10.1 |
| D2     | Vlan10    | 192.168.10.3/24 | 10         | Standby       | 192.168.10.1 |

### HSRP Role Design

| Device | Priority | Preempt | Expected State |
| ------ | -------- | ------- | -------------- |
| D1     | 110      | Enabled | Active         |
| D2     | 100      | Enabled | Standby        |

### HSRP Virtual MAC

| HSRP Version   | Group | Virtual MAC    |
| -------------- | ----- | -------------- |
| HSRP Version 1 | 10    | 0000.0c07.ac0a |

**Note:** For HSRP version 1, the virtual MAC format is `0000.0c07.acXX`, where `XX` is the HSRP group number in hexadecimal. Group `10` converts to `0a`.

---

## Prerequisites and Pre-Checks

Before configuring HSRP, confirm that the Layer 2 and Layer 3 baseline is already working.

### Prerequisites

* [ ] Target VLAN exists on both distribution switches.
* [ ] Target SVI exists on both distribution switches.
* [ ] Each SVI has a unique real IP address.
* [ ] SVIs are administratively enabled.
* [ ] Layer 2 adjacency exists between the HSRP peers.
* [ ] The target VLAN is allowed across required trunks.
* [ ] End-user access ports are assigned to the correct VLAN.
* [ ] No duplicate IP address exists for the planned HSRP virtual IP.
* [ ] IP routing is enabled if the devices are Layer 3 switches.

### Baseline Verification

Run these commands before applying HSRP configuration.

```bash
show ip interface brief
show vlan brief
show interfaces trunk
show cdp neighbors
show running-config interface vlan 10
show running-config | include ^ip routing
```

### Expected Baseline Results

* [ ] `Vlan10` is present on both devices.
* [ ] `Vlan10` is up/up on both devices.
* [ ] D1 has the expected real SVI address.
* [ ] D2 has the expected real SVI address.
* [ ] Required trunks are operational.
* [ ] VLAN 10 is allowed on required trunks.
* [ ] CDP confirms the expected Layer 2 adjacency.
* [ ] IP routing is enabled on Layer 3 switches.
* [ ] The planned virtual IP is not already assigned as a real interface IP.


---

## Configuration Procedure

Use this procedure to configure HSRP for VLAN 10 on D1 and D2.

### Configuration Notes

* D1 is configured with a higher priority so it becomes the active HSRP gateway.
* D2 uses the default priority value of `100` and should become the standby HSRP gateway.
* Preemption is enabled so the intended active device can reclaim the active role after recovery.
* The HSRP virtual IP must be in the same subnet as the real SVI addresses.
* The HSRP virtual IP must not be assigned as a real interface IP address.

---

### Step 1 — Configure D1 as the Intended Active Gateway

```bash
enable
configure terminal

interface vlan 10
description VLAN10 Users Gateway
ip address 192.168.10.2 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 110
standby 10 preempt
standby 10 name HSRP_VLAN10_Gateway
no shutdown

end
write memory
```

### Step 2 — Configure D2 as the Intended Standby Gateway

```bash
enable
configure terminal

interface vlan 10
description VLAN10 Users Gateway
ip address 192.168.10.3 255.255.255.0
standby 10 ip 192.168.10.1
standby 10 priority 100
standby 10 preempt
standby 10 name HSRP_VLAN10_Gateway
no shutdown

end
write memory
```

### Step 3 — Confirm Configuration Was Applied

```bash
show running-config interface vlan 10
show standby brief
```

Expected result:

* [ ] D1 shows HSRP group `10` configured on `Vlan10`.
* [ ] D2 shows HSRP group `10` configured on `Vlan10`.
* [ ] Both devices use virtual IP `192.168.10.1`.
* [ ] D1 priority is `110`.
* [ ] D2 priority is `100`.
* [ ] Preemption is enabled on both devices.
* [ ] HSRP group name matches on both devices.

---

## Post-Configuration Validation

Use this section to confirm that HSRP is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify SVI Interface State

Run on both D1 and D2.

```bash
show ip interface brief
```

Expected results:

* [ ] `Vlan10` is up/up on D1.
* [ ] `Vlan10` is up/up on D2.
* [ ] D1 `Vlan10` has real IP address `192.168.10.2`.
* [ ] D2 `Vlan10` has real IP address `192.168.10.3`.
* [ ] No unexpected interface state issues are present.

---

### Step 2 — Verify HSRP Summary State

Run on both D1 and D2.

```bash
show standby brief
```

Expected results:

* [ ] D1 shows `Active` for VLAN 10 / group 10.
* [ ] D2 shows `Standby` for VLAN 10 / group 10.
* [ ] Virtual IP is `192.168.10.1`.
* [ ] D1 priority is `110`.
* [ ] D2 priority is `100`.
* [ ] Standby peer information is present.
* [ ] No device is stuck in `Speak`, `Listen`, or `Init`.

---

### Step 3 — Verify Detailed HSRP State

Run on both D1 and D2.

```bash
show standby vlan 10
```

Expected results:

* [ ] HSRP group is `10`.
* [ ] Virtual IP is `192.168.10.1`.
* [ ] D1 is the active router.
* [ ] D2 is the standby router.
* [ ] Preemption is enabled.
* [ ] HSRP timers are visible.
* [ ] Virtual MAC address is `0000.0c07.ac0a`.
* [ ] Active and standby peer information matches the reference design.

---

### Step 4 — Verify Layer 2 Adjacency

Run on both D1 and D2.

```bash
show cdp neighbors
```

Expected results:

* [ ] Expected distribution/access switch adjacencies are present.
* [ ] Devices appear on the expected local interfaces.
* [ ] No required peer is missing.
* [ ] No unexpected topology connection is present.

---

### Step 5 — Verify HSRP Virtual MAC Learning

Run on the relevant switch in the Layer 2 path.

```bash
show mac address-table | include 0000.0c07.ac0a
```

Expected results:

* [ ] HSRP virtual MAC address is present.
* [ ] Virtual MAC is learned through the expected path toward the active HSRP device.
* [ ] MAC table output aligns with the expected active gateway placement.

---

### Step 6 — Verify Client Gateway Reachability

Run from a client in VLAN 10.

```bash
ping -w 5 192.168.10.1
```

Expected results:

* [ ] Client successfully reaches the HSRP virtual IP.
* [ ] Packet loss is 0%.
* [ ] Latency is normal for the lab or local network.
* [ ] Client default gateway is set to `192.168.10.1`.

---

## Failover Validation

Use this section to confirm that the standby device can take over forwarding for the HSRP virtual IP after a failure of the active path.

**Note:** Perform failover testing during a maintenance window or lab validation window. In production, confirm the approved failure method before shutting down any interface.

### Step 1 — Confirm Initial HSRP State

Run on both D1 and D2 before introducing a failure.

```bash
show standby brief
```

Expected results:

* [ ] D1 is `Active` for VLAN 10 / group 10.
* [ ] D2 is `Standby` for VLAN 10 / group 10.
* [ ] Virtual IP is `192.168.10.1`.
* [ ] Client traffic reaches the virtual IP before failover testing begins.

---

### Step 2 — Start Continuous Client Ping

Run from a client in VLAN 10.

```bash
ping 192.168.10.1
```

Expected results:

* [ ] Client receives replies from the HSRP virtual IP.
* [ ] Ping is stable before the failure is introduced.

---

### Step 3 — Simulate Active Gateway Failure

On D1, shut down the VLAN 10 SVI.

```bash
enable
configure terminal

interface vlan 10
shutdown

end
```

Alternative failure methods may include shutting down the uplink or removing the active device from the Layer 2 path, depending on the lab or maintenance plan.

Expected results:

* [ ] D1 no longer forwards for the VLAN 10 HSRP group.
* [ ] D2 transitions from `Standby` to `Active`.
* [ ] Client ping may drop briefly during convergence.
* [ ] Client ping resumes through the new active gateway.

---

### Step 4 — Verify Standby Becomes Active

Run on D2.

```bash
show standby brief
show standby vlan 10
```

Expected results:

* [ ] D2 is now `Active` for VLAN 10 / group 10.
* [ ] Virtual IP remains `192.168.10.1`.
* [ ] Virtual MAC remains `0000.0c07.ac0a`.
* [ ] D2 is forwarding for the HSRP group.

---

### Step 5 — Restore the Original Active Device

On D1, restore the VLAN 10 SVI.

```bash
enable
configure terminal

interface vlan 10
no shutdown

end
```

Expected results:

* [ ] D1 SVI returns to up/up.
* [ ] D1 participates in HSRP again.
* [ ] Because preemption is enabled and D1 has higher priority, D1 should reclaim the active role.
* [ ] D2 should return to standby.

---

### Step 6 — Verify Recovery State

Run on both D1 and D2.

```bash
show standby brief
show standby vlan 10
```

Expected results:

* [ ] D1 is `Active` for VLAN 10 / group 10.
* [ ] D2 is `Standby` for VLAN 10 / group 10.
* [ ] Virtual IP remains `192.168.10.1`.
* [ ] Virtual MAC remains `0000.0c07.ac0a`.
* [ ] Client traffic continues to reach the virtual IP after recovery.

---

## If Validation Fails

If post-configuration or failover validation does not match the expected results, stop and verify the baseline before making additional changes.

Run these checks first:

```bash
show standby brief
show standby vlan 10
show ip interface brief
show running-config interface vlan 10
show cdp neighbors
show interfaces trunk
```

Confirm the following:

* [ ] The target SVI is up/up on both devices.
* [ ] Both devices use the same HSRP group number.
* [ ] Both devices use the same HSRP virtual IP.
* [ ] The intended active device has the higher priority.
* [ ] Preemption is configured if deterministic recovery is required.
* [ ] The target VLAN exists and is allowed across required trunks.
* [ ] Layer 2 adjacency exists between the HSRP peers.

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

Removing or changing HSRP on a live VLAN can interrupt client default-gateway reachability if endpoints are using the HSRP virtual IP as their default gateway.

Before removing or changing HSRP configuration, confirm:

* An approved replacement gateway exists.
* Client default-gateway behavior is understood.
* The original SVI IP addressing plan is documented.
* The expected active and standby roles are documented.
* A maintenance or lab validation window is available if traffic interruption is possible.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                          |
| ---------------- | ------------------------------ |
| Author           | Aaron Kindelt                  |
| Category         | First-Hop Redundancy           |
| Technology       | HSRP                           |
| Applies To       | Routers, Layer 3 Switches      |
| Primary Use Case | SVI default gateway redundancy |
| Version          | 1.0                            |
| Last Updated     | 2026-06-28                     |