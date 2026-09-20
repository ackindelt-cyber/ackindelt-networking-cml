# Runbook — VLANs, Trunks, and SVI Inter-VLAN Routing

## Overview

This runbook provides a structured process for configuring and validating VLANs, 802.1Q trunks, switch virtual interfaces, and inter-VLAN routing on a Layer 3 switch.

VLANs separate Layer 2 broadcast domains. Trunks carry multiple VLANs between switches. SVIs provide Layer 3 gateway interfaces for VLANs, allowing clients in different VLANs to communicate through the Layer 3 switch.

This runbook focuses on a basic distribution/access switching design. The example topology uses D1 as the Layer 3 distribution switch, A1 and A2 as access switches, VLANs 10 and 20 as client VLANs, and VLAN 99 as the native VLAN.

## Scope and Assumptions

This runbook assumes:

* D1 is a Layer 3 switch.
* A1 and A2 are Layer 2 access switches.
* VLANs are configured locally on each switch.
* VTP is not used.
* D1 provides the default gateway for each client VLAN using SVIs.
* Trunk links carry VLANs 10, 20, and 99.
* VLAN 99 is used as the native VLAN.
* Clients use static IP addresses for validation unless DHCP is added separately.
* STP edge-port protection is handled by a separate runbook.

This runbook does not cover DHCP, HSRP, STP root bridge placement, PortFast, BPDU Guard, Root Guard, VTP, EtherChannel, or router-on-a-stick.

---

## Reference Design

This runbook uses one Layer 3 distribution switch and two Layer 2 access switches.

### Devices

| Device | Role                        | Purpose                          |
| ------ | --------------------------- | -------------------------------- |
| D1     | Layer 3 Distribution Switch | VLAN SVIs and inter-VLAN routing |
| A1     | Access Switch               | VLAN 10 client access            |
| A2     | Access Switch               | VLAN 20 client access            |
| C1     | Client                      | VLAN 10 endpoint                 |
| C2     | Client                      | VLAN 20 endpoint                 |

### VLAN Summary

| VLAN | Name            | Purpose                         |
| ---- | --------------- | ------------------------------- |
| 10   | VLAN10_Users    | User client VLAN                |
| 20   | VLAN20_Printers | Printer / secondary client VLAN |
| 99   | VLAN99_Native   | Native VLAN for trunks          |

### Network Summary

| VLAN | Subnet          | Default Gateway | Client Example     |
| ---- | --------------- | --------------- | ------------------ |
| 10   | 192.168.10.0/24 | 192.168.10.1    | C1 — 192.168.10.10 |
| 20   | 192.168.20.0/24 | 192.168.20.1    | C2 — 192.168.20.10 |

### Physical Link Summary

| Link | Device A | Interface A | Device B | Interface B | Purpose             |
| ---- | -------- | ----------- | -------- | ----------- | ------------------- |
| 1    | D1       | Gi0/0       | A1       | Gi0/0       | 802.1Q trunk        |
| 2    | D1       | Gi0/1       | A2       | Gi0/0       | 802.1Q trunk        |
| 3    | A1       | Gi0/1       | C1       | Ethernet    | VLAN 10 access port |
| 4    | A2       | Gi0/1       | C2       | Ethernet    | VLAN 20 access port |

### Interface / Feature Summary

| Device | Interface | Mode   | VLAN / Address  | Purpose                 |
| ------ | --------- | ------ | --------------- | ----------------------- |
| D1     | Vlan10    | SVI    | 192.168.10.1/24 | VLAN 10 default gateway |
| D1     | Vlan20    | SVI    | 192.168.20.1/24 | VLAN 20 default gateway |
| D1     | Gi0/0     | Trunk  | 10,20,99        | Trunk to A1             |
| D1     | Gi0/1     | Trunk  | 10,20,99        | Trunk to A2             |
| A1     | Gi0/0     | Trunk  | 10,20,99        | Trunk to D1             |
| A1     | Gi0/1     | Access | VLAN 10         | C1 access port          |
| A2     | Gi0/0     | Trunk  | 10,20,99        | Trunk to D1             |
| A2     | Gi0/1     | Access | VLAN 20         | C2 access port          |

### Design Values

| Item                      | Value               |
| ------------------------- | ------------------- |
| Routing Device            | D1                  |
| Native VLAN               | 99                  |
| Allowed Trunk VLANs       | 10,20,99            |
| VLAN 10 Gateway           | 192.168.10.1        |
| VLAN 20 Gateway           | 192.168.20.1        |
| Inter-VLAN Routing Method | Layer 3 switch SVIs |

**Note:** Some Cisco platforms require `switchport trunk encapsulation dot1q` before setting an interface to trunk mode. If the command is not supported, configure the trunk without it.

---

## Prerequisites and Pre-Checks

Before configuring VLANs, trunks, and SVIs, confirm that the physical and baseline switch configuration is already working.

### Prerequisites

* [ ] D1, A1, and A2 are connected according to the reference design.
* [ ] Required switch interfaces are enabled.
* [ ] D1 supports Layer 3 switching and `ip routing`.
* [ ] C1 and C2 are connected to the expected access ports.
* [ ] No existing VLAN or trunk configuration needs to be preserved.
* [ ] Interface names have been verified for the platform.

### Baseline Verification

Run these commands before applying the VLAN, trunk, and SVI configuration.

```bash
show ip interface brief
show interfaces status
show cdp neighbors
show running-config | section interface
show running-config | include ^ip routing
```

### Expected Baseline Results

* [ ] Required switch links are physically up.
* [ ] CDP confirms the expected switch adjacencies.
* [ ] D1 supports Layer 3 routing.
* [ ] Interfaces used in this runbook are identified.
* [ ] No conflicting trunk, access VLAN, or SVI configuration is present.

---

## Configuration Procedure

Use this procedure to configure VLANs, trunks, SVIs, and inter-VLAN routing.

### Configuration Notes

* VLANs must exist on each switch that carries or uses the VLAN.
* Trunk links must allow VLANs 10, 20, and 99.
* Native VLANs must match on both sides of each trunk.
* D1 requires `ip routing` to route between SVIs.
* Client default gateways should point to the SVI IP address for their VLAN.
* Access ports should be assigned to a single VLAN.

---

### Step 1 — Configure D1 VLANs, SVIs, Routing, and Trunks

Run on D1.

```bash
enable
configure terminal

vlan 10
name VLAN10_Users

vlan 20
name VLAN20_Printers

vlan 99
name VLAN99_Native

ip routing

interface vlan 10
description VLAN10 Users Gateway
ip address 192.168.10.1 255.255.255.0
no shutdown

interface vlan 20
description VLAN20 Printers Gateway
ip address 192.168.20.1 255.255.255.0
no shutdown

interface gigabitEthernet 0/0
description Trunk to A1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown

interface gigabitEthernet 0/1
description Trunk to A2
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown

end
write memory
```

### Step 2 — Configure A1 VLANs, Trunk, and VLAN 10 Access Port

Run on A1.

```bash
enable
configure terminal

vlan 10
name VLAN10_Users

vlan 20
name VLAN20_Printers

vlan 99
name VLAN99_Native

interface gigabitEthernet 0/0
description Trunk to D1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown

interface gigabitEthernet 0/1
description Access Port to C1 - VLAN 10
switchport mode access
switchport access vlan 10
no shutdown

end
write memory
```

### Step 3 — Configure A2 VLANs, Trunk, and VLAN 20 Access Port

Run on A2.

```bash
enable
configure terminal

vlan 10
name VLAN10_Users

vlan 20
name VLAN20_Printers

vlan 99
name VLAN99_Native

interface gigabitEthernet 0/0
description Trunk to D1
switchport trunk encapsulation dot1q
switchport mode trunk
switchport trunk native vlan 99
switchport trunk allowed vlan 10,20,99
no shutdown

interface gigabitEthernet 0/1
description Access Port to C2 - VLAN 20
switchport mode access
switchport access vlan 20
no shutdown

end
write memory
```

### Step 4 — Confirm Configuration Was Applied

Run on D1, A1, and A2 as appropriate.

```bash
show vlan brief
show interfaces trunk
show running-config interface gigabitEthernet 0/0
show running-config interface gigabitEthernet 0/1
show running-config | include ^ip routing
show running-config interface vlan 10
show running-config interface vlan 20
```

Expected result:

* [ ] VLANs 10, 20, and 99 exist.
* [ ] Trunk interfaces are configured as trunks.
* [ ] Trunk native VLAN is 99.
* [ ] Trunk allowed VLAN list is 10,20,99.
* [ ] A1 access port is assigned to VLAN 10.
* [ ] A2 access port is assigned to VLAN 20.
* [ ] D1 has SVIs for VLANs 10 and 20.
* [ ] D1 has IP routing enabled.

---

## Post-Configuration Validation

Use this section to confirm that VLANs, trunks, SVIs, and inter-VLAN routing are configured correctly after implementation.

### Step 1 — Verify VLAN Database

Run on D1, A1, and A2.

```bash
show vlan brief
```

Expected results:

* [ ] VLAN 10 exists.
* [ ] VLAN 20 exists.
* [ ] VLAN 99 exists.
* [ ] A1 client-facing port is assigned to VLAN 10.
* [ ] A2 client-facing port is assigned to VLAN 20.

---

### Step 2 — Verify Trunk State

Run on D1, A1, and A2.

```bash
show interfaces trunk
```

Expected results:

* [ ] D1 trunk to A1 is operational.
* [ ] D1 trunk to A2 is operational.
* [ ] A1 trunk to D1 is operational.
* [ ] A2 trunk to D1 is operational.
* [ ] Native VLAN is 99 on each trunk.
* [ ] VLANs 10, 20, and 99 are allowed on each trunk.

---

### Step 3 — Verify SVI State on D1

Run on D1.

```bash
show ip interface brief
show running-config interface vlan 10
show running-config interface vlan 20
```

Expected results:

* [ ] `Vlan10` exists.
* [ ] `Vlan10` is up/up.
* [ ] `Vlan10` has IP address `192.168.10.1`.
* [ ] `Vlan20` exists.
* [ ] `Vlan20` is up/up.
* [ ] `Vlan20` has IP address `192.168.20.1`.

---

### Step 4 — Verify Layer 3 Routing on D1

Run on D1.

```bash
show running-config | include ^ip routing
show ip route connected
```

Expected results:

* [ ] `ip routing` is enabled.
* [ ] Connected route for `192.168.10.0/24` is present.
* [ ] Connected route for `192.168.20.0/24` is present.
* [ ] No unexpected dynamic routing protocol routes are present.

---

### Step 5 — Verify Access Port Configuration

Run on A1 and A2.

```bash
show running-config interface gigabitEthernet 0/1
show interfaces status
```

Expected results:

* [ ] A1 `Gi0/1` is configured as an access port in VLAN 10.
* [ ] A2 `Gi0/1` is configured as an access port in VLAN 20.
* [ ] Client-facing interfaces are up.
* [ ] No client-facing interface is err-disabled.

---

## Functional Validation

Use this section to confirm that clients can reach their default gateway and communicate across VLANs.

### Step 1 — Configure Client Addressing

Configure clients with the following static IP settings.

| Client | VLAN | IP Address    | Subnet Mask   | Default Gateway |
| ------ | ---- | ------------- | ------------- | --------------- |
| C1     | 10   | 192.168.10.10 | 255.255.255.0 | 192.168.10.1    |
| C2     | 20   | 192.168.20.10 | 255.255.255.0 | 192.168.20.1    |

Expected results:

* [ ] C1 is addressed in VLAN 10 subnet.
* [ ] C2 is addressed in VLAN 20 subnet.
* [ ] Each client uses the correct SVI as its default gateway.

---

### Step 2 — Verify Client-to-Gateway Reachability

Run from C1.

```bash
ping -w 5 192.168.10.1
```

Run from C2.

```bash
ping -w 5 192.168.20.1
```

Expected results:

* [ ] C1 can reach the VLAN 10 gateway.
* [ ] C2 can reach the VLAN 20 gateway.
* [ ] Packet loss is 0%.
* [ ] Latency is normal for the lab or local network.

---

### Step 3 — Verify Inter-VLAN Connectivity

Run from C1.

```bash
ping -w 5 192.168.20.10
tracepath 192.168.20.10
```

Run from C2.

```bash
ping -w 5 192.168.10.10
tracepath 192.168.10.10
```

Expected results:

* [ ] C1 can reach C2.
* [ ] C2 can reach C1.
* [ ] Inter-VLAN traffic routes through D1.
* [ ] The first Layer 3 hop is the local SVI gateway.
* [ ] Packet loss is 0%.

---

## If Validation Fails

If post-configuration or functional validation does not match the expected results, stop and verify the baseline before making additional changes.

Run these checks first:

```bash
show vlan brief
show interfaces trunk
show ip interface brief
show ip route connected
show running-config | include ^ip routing
show cdp neighbors
show interfaces status
```

Confirm the following:

* [ ] VLANs 10, 20, and 99 exist on all required switches.
* [ ] Trunks are operational.
* [ ] Native VLAN matches on both sides of each trunk.
* [ ] VLANs 10, 20, and 99 are allowed across each trunk.
* [ ] Access ports are assigned to the correct VLANs.
* [ ] D1 SVIs are up/up.
* [ ] D1 has `ip routing` enabled.
* [ ] Clients are addressed in the correct subnet.
* [ ] Client default gateways match the correct SVI IP addresses.

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

Removing or changing VLANs, trunks, or SVIs can interrupt Layer 2 forwarding, client gateway reachability, and inter-VLAN routing.

Before removing or changing the configuration, confirm:

* The affected VLANs are identified.
* The affected trunk links are identified.
* Client access port assignments are documented.
* SVI gateway addresses are documented.
* The expected client addressing plan is understood.
* A maintenance or lab validation window is available if traffic interruption is possible.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                               |
| ---------------- | --------------------------------------------------- |
| Author           | Aaron Kindelt                                       |
| Category         | Layer 2 / Layer 3 Switching                         |
| Technology       | VLANs, 802.1Q Trunks, SVIs                          |
| Applies To       | Layer 3 Switches, Layer 2 Switches                  |
| Primary Use Case | VLAN segmentation and inter-VLAN routing using SVIs |
| Version          | 1.0                                                 |
| Last Updated     | 2026-06-28                                          |
