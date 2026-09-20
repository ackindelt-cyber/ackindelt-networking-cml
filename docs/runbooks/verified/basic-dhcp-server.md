# Runbook — Basic DHCP Server on a Routed LAN

## Overview

This runbook provides a structured process for configuring and validating a basic DHCP server on a routed LAN.

DHCP allows network clients to automatically receive IPv4 addressing information, including an IP address, subnet mask, default gateway, and DNS server. In this design, R1 provides DHCP services for a directly connected LAN subnet.

This runbook focuses on a single DHCP scope for one routed LAN. The example topology uses network `192.168.10.0/24`, with R1 acting as both the DHCP server and default gateway for client devices.

## Scope and Assumptions

This runbook assumes:

* The DHCP server is a Cisco router or Layer 3 device.
* The DHCP client is located on the same subnet as the DHCP server interface.
* The DHCP server interface is already connected to the target LAN.
* The target LAN uses a single IPv4 subnet.
* The default gateway address is excluded from the DHCP pool.
* Clients are configured to obtain an IPv4 address automatically.

This runbook does not cover DHCP relay, multi-VLAN DHCP designs, DHCP redundancy, or advanced DHCP options.

---

## Reference Design

This runbook uses a single-router DHCP design for one routed LAN.

### Devices

| Device | Role          | Purpose                         |
| ------ | ------------- | ------------------------------- |
| R1     | Router        | DHCP server and default gateway |
| A1     | Access Switch | Layer 2 client access           |
| C1     | Client        | DHCP client                     |

### Network Summary

| Network / VLAN / Segment | Purpose             | Subnet          | Gateway / Next Hop |
| ------------------------ | ------------------- | --------------- | ------------------ |
| LAN                      | Client DHCP network | 192.168.10.0/24 | 192.168.10.1       |

### Interface / Feature Summary

| Device | Interface          | IP Address / Value | Feature Role          | Notes                        |
| ------ | ------------------ | ------------------ | --------------------- | ---------------------------- |
| R1     | Gi0/0              | 192.168.10.1/24    | DHCP server interface | Default gateway for clients  |
| A1     | Access ports       | Layer 2 only       | DHCP pass-through     | Forwards client DHCP traffic |
| C1     | Ethernet interface | DHCP assigned      | DHCP client           | Receives address from R1     |

### DHCP Design Values

| Item                   | Value                          |
| ---------------------- | ------------------------------ |
| DHCP Scope             | LAN_USERS                      |
| DHCP Network           | 192.168.10.0/24                |
| Default Gateway        | 192.168.10.1                   |
| Excluded Address Range | 192.168.10.1 - 192.168.10.20   |
| Assignable DHCP Range  | 192.168.10.21 - 192.168.10.254 |
| DNS Server             | 8.8.8.8                        |

**Note:** DHCP pool selection is based on the subnet of the Layer 3 interface that receives the DHCP request. In this design, client DHCP requests are received on R1 `Gi0/0`, so R1 assigns addresses from the `192.168.10.0/24` DHCP pool.

---

## Prerequisites and Pre-Checks

Before configuring DHCP, confirm that the required Layer 2 and Layer 3 baseline is already working.

### Prerequisites

* [ ] R1 is connected to the client LAN.
* [ ] R1 LAN-facing interface is configured with the correct IP address.
* [ ] R1 LAN-facing interface is administratively enabled.
* [ ] A1 provides Layer 2 connectivity between R1 and C1.
* [ ] C1 is connected to the correct access switch port.
* [ ] C1 is configured to obtain an IPv4 address automatically.
* [ ] Reserved infrastructure addresses are identified before creating the DHCP pool.

### Baseline Verification

Run these commands before applying the DHCP configuration.

```bash
show ip interface brief
show cdp neighbors
show running-config interface gigabitEthernet 0/0
```

### Expected Baseline Results

* [ ] R1 `Gi0/0` is up/up.
* [ ] R1 `Gi0/0` has IP address `192.168.10.1/24`.
* [ ] R1 has the expected Layer 2 adjacency toward A1.
* [ ] The client is connected to the expected LAN segment.
* [ ] No existing DHCP pool conflicts with the planned DHCP scope.

---

## Configuration Procedure

Use this procedure to configure R1 as the DHCP server for the LAN.

### Configuration Notes

* The default gateway address must be excluded from DHCP assignment.
* Additional reserved infrastructure addresses should also be excluded.
* The DHCP pool network must match the subnet configured on the LAN-facing interface.
* The `default-router` value should match the default gateway used by clients.
* The DNS server option provides clients with a resolver address but does not guarantee external connectivity by itself.

---

### Step 1 — Configure the LAN Interface on R1

```bash
enable
configure terminal

interface gigabitEthernet 0/0
description LAN Gateway - DHCP Scope 192.168.10.0/24
ip address 192.168.10.1 255.255.255.0
no shutdown

end
write memory
```

### Step 2 — Configure the DHCP Excluded Address Range

```bash
enable
configure terminal

ip dhcp excluded-address 192.168.10.1 192.168.10.20

end
write memory
```

### Step 3 — Configure the DHCP Pool

```bash
enable
configure terminal

ip dhcp pool LAN_USERS
network 192.168.10.0 255.255.255.0
default-router 192.168.10.1
dns-server 8.8.8.8

end
write memory
```

### Step 4 — Confirm Configuration Was Applied

```bash
show running-config | section ip dhcp
show running-config interface gigabitEthernet 0/0
show ip dhcp pool
```

Expected result:

* [ ] DHCP excluded address range is configured.
* [ ] DHCP pool `LAN_USERS` exists.
* [ ] DHCP pool network is `192.168.10.0/24`.
* [ ] Default router is `192.168.10.1`.
* [ ] DNS server is configured.
* [ ] R1 `Gi0/0` is configured as the LAN default gateway.

---

## Post-Configuration Validation

Use this section to confirm that DHCP is configured correctly and operating in the expected state after implementation.

### Step 1 — Verify Interface State

Run on R1.

```bash
show ip interface brief
```

Expected results:

* [ ] `GigabitEthernet0/0` is up/up.
* [ ] `GigabitEthernet0/0` has IP address `192.168.10.1`.
* [ ] No unexpected interface state issues are present.

---

### Step 2 — Verify DHCP Pool Summary

Run on R1.

```bash
show ip dhcp pool
```

Expected results:

* [ ] DHCP pool `LAN_USERS` exists.
* [ ] DHCP pool network is `192.168.10.0/24`.
* [ ] Available address count is greater than zero.
* [ ] Excluded addresses are not available for assignment.
* [ ] Pool is not exhausted.

---

### Step 3 — Verify DHCP Bindings

Run on R1 after a client requests an address.

```bash
show ip dhcp binding
```

Expected results:

* [ ] Client DHCP binding is present.
* [ ] Assigned client address is within `192.168.10.21 - 192.168.10.254`.
* [ ] Client MAC address is associated with the lease.
* [ ] Lease state is active.

---

### Step 4 — Verify DHCP Server Statistics

Run on R1.

```bash
show ip dhcp server statistics
```

Expected results:

* [ ] DHCP message counters increment after client requests.
* [ ] DHCP server receives client requests.
* [ ] DHCP server sends offers or acknowledgments.
* [ ] No excessive drops or failures are present.

---

### Step 5 — Verify Client Addressing

Run from C1.

```bash
ip addr
ip route
```

Expected results:

* [ ] C1 receives an IPv4 address from the DHCP pool.
* [ ] C1 subnet mask matches `/24`.
* [ ] C1 default gateway is `192.168.10.1`.
* [ ] C1 has a connected route for `192.168.10.0/24`.

---

## Functional Validation

Use this section to confirm that DHCP works under the intended operating condition.

### Step 1 — Request a DHCP Lease

Run from C1.

```bash
udhcpc -i eth0
```

Expected results:

* [ ] C1 sends a DHCP request.
* [ ] C1 receives a DHCP lease.
* [ ] C1 is assigned an address from the correct DHCP scope.
* [ ] R1 records the lease in the DHCP binding table.

---

### Step 2 — Verify Client Gateway Reachability

Run from C1.

```bash
ping -w 5 192.168.10.1
```

Expected results:

* [ ] C1 can reach the default gateway.
* [ ] Packet loss is 0%.
* [ ] Latency is normal for the lab or local network.

---

### Step 3 — Verify DHCP Binding on R1

Run on R1.

```bash
show ip dhcp binding
show ip dhcp server statistics
```

Expected results:

* [ ] C1 appears in the DHCP binding table.
* [ ] DHCP statistics reflect successful client lease activity.
* [ ] Assigned lease matches the client addressing observed on C1.

---

### Step 4 — Optional Upstream Reachability Test

Run from C1 only if upstream routing or external connectivity is part of the lab design.

```bash
ping -w 5 <upstream-ip>
```

Expected results:

* [ ] C1 can reach the upstream destination if routing exists.
* [ ] If upstream connectivity fails, confirm whether routing, NAT, or external connectivity is outside the scope of this DHCP runbook.

---

## If Validation Fails

If post-configuration or functional validation does not match the expected results, stop and verify the baseline before making additional changes.

Run these checks first:

```bash
show ip interface brief
show running-config | section ip dhcp
show ip dhcp pool
show ip dhcp binding
show ip dhcp server statistics
show cdp neighbors
```

Confirm the following:

* [ ] R1 LAN-facing interface is up/up.
* [ ] R1 LAN-facing interface is in the same subnet as the DHCP pool.
* [ ] DHCP excluded address range does not exclude the entire pool.
* [ ] DHCP pool network and subnet mask are correct.
* [ ] DHCP `default-router` value is correct.
* [ ] C1 is connected to the correct LAN segment.
* [ ] C1 is configured as a DHCP client.
* [ ] No duplicate DHCP server is serving the same LAN unexpectedly.

If the issue is not resolved after these checks, stop and document the failed validation results before continuing with deeper troubleshooting.

---

## Backout Considerations

Removing or changing DHCP configuration can interrupt client address assignment and renewal for the affected LAN.

Before removing or changing the configuration, confirm:

* An approved replacement DHCP service exists if clients still require dynamic addressing.
* Client addressing behavior is understood.
* Static infrastructure addresses are documented.
* The original DHCP scope and excluded address range are documented.
* A maintenance or lab validation window is available if client address assignment may be interrupted.

For lab use, restore the last known working configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                               |
| ---------------- | ----------------------------------- |
| Author           | Aaron Kindelt                       |
| Category         | Network Services                    |
| Technology       | DHCP                                |
| Applies To       | Routers, Layer 3 Switches           |
| Primary Use Case | Basic DHCP service for a routed LAN |
| Version          | 1.0                                 |
| Last Updated     | 2026-06-28                          |
