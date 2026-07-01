# Lab Guide — Basic HSRP

## Overview

This lab demonstrates how to configure and verify Hot Standby Router Protocol.

HSRP provides first-hop gateway redundancy by allowing two routers to share a virtual IP address and virtual MAC address. Hosts use the HSRP virtual IP as their default gateway. One router actively forwards traffic for the virtual gateway, while the standby router takes over if the active router fails.

This lab validates HSRP group configuration, active and standby router roles, virtual IP and virtual MAC behavior, failover after a simulated active-router failure, and recovery after the preferred active router returns.

**Lab Status:** Validated
**End-to-End Verification:** Successful

---

## Objectives

* [x] Configure LAN interface IP addressing on R1 and R2.
* [x] Configure HSRP group 10.
* [x] Configure an HSRP virtual IP address.
* [x] Configure R1 as the preferred active router using priority.
* [x] Enable preemption so R1 resumes the active role after recovery.
* [x] Verify active and standby HSRP states.
* [x] Verify the HSRP virtual MAC address on the access switch.
* [x] Test client connectivity to the HSRP virtual gateway.
* [x] Simulate active-router failure.
* [x] Verify standby router takeover.
* [x] Verify recovery and preemption behavior.
* [x] Capture selected HSRP and ICMP traffic for packet-level validation.

---

## Topology

The topology uses two routers, one Layer 2 switch, and one client. R1 and R2 connect to the same LAN segment and share the HSRP virtual gateway address `192.168.10.1`.

Topology artifacts are available in the `topology/` folder.
 
- [Open topology diagram](topology/diagram.svg)
- [Open CML topology export](topology/topology.yaml)

---

## Addressing Table

| Device   | Interface | IP Address       | Description                      |
| -------- | --------- | ---------------- | -------------------------------- |
| R1       | Gi0/0     | 192.168.10.2/24  | Preferred HSRP active router     |
| R2       | Gi0/0     | 192.168.10.3/24  | HSRP standby router              |
| HSRP VIP | Group 10  | 192.168.10.1/24  | Virtual default gateway          |
| C1       | eth0      | 192.168.10.50/24 | Client using HSRP VIP as gateway |

---

## HSRP Parameters

| Parameter   | Value          |
| ----------- | -------------- |
| HSRP Group  | 10             |
| Virtual IP  | 192.168.10.1   |
| Virtual MAC | 0000.0c07.ac0a |
| R1 Priority | 110            |
| R2 Priority | 100            |
| Hello Timer | 3 seconds      |
| Hold Timer  | 10 seconds     |
| Preemption  | Enabled        |
| Group Name  | HSRP-GROUP10   |

---

## Configuration Steps

### Step 1 — R1 HSRP Configuration

**Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

**R1**

```bash
# R1 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R1 # Sets hostname to R1.

interface gigabitEthernet0/0 # Targets the LAN interface.
description R1 LAN to S1 with HSRP # Adds an interface description.
ip address 192.168.10.2 255.255.255.0 # Sets R1 LAN IP address.
no shutdown # Enables the interface.

standby 10 ip 192.168.10.1 # Creates HSRP group 10 and assigns the virtual IP address.
standby 10 priority 110 # Sets R1 priority higher than the default.
standby 10 preempt # Allows R1 to resume active status after recovery.
standby 10 name HSRP-GROUP10 # Sets the HSRP group name.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

### Step 2 — R2 HSRP Configuration

**R2**

```bash
# R2 Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname R2 # Sets hostname to R2.

interface gigabitEthernet0/0 # Targets the LAN interface.
description R2 LAN to S1 with HSRP # Adds an interface description.
ip address 192.168.10.3 255.255.255.0 # Sets R2 LAN IP address.
no shutdown # Enables the interface.

standby 10 ip 192.168.10.1 # Creates HSRP group 10 and assigns the virtual IP address.
standby 10 priority 100 # Sets R2 priority to 100. This is also the HSRP default.
standby 10 preempt # Allows R2 to preempt only if it has the highest priority.
standby 10 name HSRP-GROUP10 # Sets the HSRP group name.

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**Design note:** R1 is preferred because it has priority `110`. R2 remains standby with priority `100`. If R1 fails, R2 becomes active. When R1 returns, preemption allows R1 to reclaim the active role.

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

### Initial HSRP Verification

```bash
# R1 and R2 Verification Block
show ip interface brief # Confirm LAN interfaces are up/up with the expected IP addresses.
show standby # Confirm HSRP state, VIP, VMAC, priority, timers, preemption, and peer role.
```

### Switch Verification

```bash
# S1 Verification Block
show mac address-table | include 0000.0c07.ac0a # Confirm the HSRP virtual MAC is learned on S1.
```

### Client Verification

```bash
# C1 Verification Block
ping -c 5 192.168.10.1 # Confirm client connectivity to the HSRP virtual gateway.
```

### Simulated Failure Verification

```bash
# R1 Failure Simulation
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
interface gigabitEthernet0/0 # Targets R1 LAN interface.
shutdown # Simulates active router failure.

# R2 Takeover Verification
show standby # Confirm R2 becomes active for HSRP group 10.

# C1 Connectivity Test During Failover
ping -c 5 192.168.10.1 # Confirm the virtual gateway remains reachable.

# R1 Recovery
interface gigabitEthernet0/0 # Targets R1 LAN interface.
no shutdown # Restores the interface.
end # Returns to privileged EXEC mode.
show standby # Confirm R1 returns to active state through preemption.

# C1 Recovery Test
ping -c 5 192.168.10.1 # Confirm the virtual gateway remains reachable after recovery.
```

### Packet Captures

The following packet captures provide additional evidence for selected HSRP and ICMP flows:

| Capture                                                                          | Description                                |
| -------------------------------------------------------------------------------- | ------------------------------------------ |
| [`C1_to_VIP_ping_R1.pcap`](verification/captures/C1_to_VIP_ping_R1.pcap)         | C1 ping to the HSRP VIP while R1 is active |
| [`C1_to_VIP_ping_R2.pcap`](verification/captures/C1_to_VIP_ping_R2.pcap)         | C1 ping to the HSRP VIP after R2 takeover  |
| [`S1_to_R1_active_hsrp.pcap`](verification/captures/S1_to_R1_active_hsrp.pcap)   | HSRP traffic with R1 active and R2 standby |
| [`S1_to_R2_failure_hsrp.pcap`](verification/captures/S1_to_R2_failure_hsrp.pcap) | HSRP traffic after simulated R1 failure    |

---

## Troubleshooting

**Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive HSRP troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# HSRP state is stuck in Init, Listen, or Speak.
show standby # Confirm group number, virtual IP, priority, timers, and peer state.
show ip interface brief # Confirm LAN interfaces are up/up.
show interfaces gigabitEthernet0/0 # Confirm physical and line protocol state.
ping 192.168.10.3 source 192.168.10.2 # From R1, confirm reachability to R2 on the shared LAN.
ping 192.168.10.2 source 192.168.10.3 # From R2, confirm reachability to R1 on the shared LAN.

# Active and standby roles are wrong.
show standby # Confirm priority values and active/standby state.
show running-config interface gigabitEthernet0/0 # Confirm HSRP priority and preempt settings.
show running-config | section standby # Confirm HSRP group configuration.

# Client cannot reach the virtual gateway.
ping 192.168.10.1 # Confirm VIP reachability from the client.
show arp # Confirm ARP entries on routers.
show mac address-table | include 0000.0c07.ac0a # Confirm the HSRP virtual MAC is learned on S1.
show standby # Confirm one router is active for the group.

# Failover does not occur.
show standby # Confirm standby router sees the active router and has a valid state.
show ip interface brief # Confirm R2 LAN interface remains up/up.
show logging # Check for interface or HSRP state-change messages.

# Failover occurs, but recovery does not return R1 to active.
show standby # Confirm R1 priority is higher than R2.
show running-config interface gigabitEthernet0/0 # Confirm R1 has preemption enabled.
show standby brief # Summarize active and standby state.

# Failover takes longer than expected.
show standby # Confirm hello and hold timers.
show running-config interface gigabitEthernet0/0 # Confirm whether custom HSRP timers are configured.
```

---

## Artifacts

| Type            | Location                                                                         |
| --------------- | -------------------------------------------------------------------------------- |
| Configurations  | [`configs/`](configs/)                                                           |
| Diagram         | [`topology/diagram.svg`](topology/diagram.svg)                                   |
| Topology File   | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification    | [`verification/verification_commands.md`](verification/verification_commands.md) |
| Packet Captures | [`verification/captures/`](verification/captures/)                               |

---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | 2026-06-28    |
| Author       | Aaron Kindelt |
