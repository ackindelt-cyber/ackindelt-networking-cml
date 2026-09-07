# Lab Guide — BGP Fundamentals

## Overview
*Briefly describe what the lab demonstrates and why it’s relevant.*
Example: This lab demonstrates how to configure and verify OSPFv2 adjacency and route exchange between two routers.

Lab Status:
* `Validated` — topology, configurations, and verification are complete
* `In Progress` — lab exists but documentation or validation is still being refined
* `Planned` — lab has not been built or is not ready for review
* `Archived` — retained for reference but no longer part of the active portfolio path
Difficulty: Beginner / Intermediate / Advanced
End-to-End Verification: Successful / Unsuccessful

---

## Objectives
*List the key goals or outcomes of the lab. Keep it to 3–5 clear objectives.*
- [ ] Configure interface IP addressing
- [ ] Establish OSPF adjacency
- [ ] Verify learned routes

---

## Topology
*Provide a quick visual reference and a table summarizing interfaces and addressing.*

![Topology Diagram](/config_labs)

---

## Addressing Tables

**Device Table**
| Device  | Interface  | IP Address / Prefix | Connected To  | Description                         |
|---------|------------|---------------------|---------------|-------------------------------------|
| R1      | G0/0       | 10.0.0.1 /30        | R2 G0/0       | Point-to-point link between routers |
| R2      | G0/0       | 10.0.0.2 /30        | R1 G0/0       | Point-to-point link between routers |
| R2      | G0/1       | 192.168.10.1 /24    | CLI1 eth0     | LAN segment                         |
| CLI1    | eth0       | 192.168.10.100 /24  | R2 G0/1       | Test client on LAN                  |

**Other Table**

---

## Configuration Steps
*Note: CLI input in this document is written using bash formatting. For unannotated versions of the configs for this lab please see [/configs](/config_labs)

**R1**
```bash
#Example configuration block
 conf t #Places router in config mode
 interface g0/0 #Sets target interface
 ip address 10.0.0.1 255.255.255.252 #Assigns IP to target interface
 no shut #Sets target interface to active
 router ospf 1 #Creates new OSPF process on router with ID of 1
 network 10.0.0.0 0.0.0.3 area 0 #Subnet that will propagate to neighbors using wildcard mask
 ```

---

 ## Verification
 *Commands and results that confirm the lab works as intended.*

 See [verification_commands](/config_labs) for command outputs.

**R1**
 ```bash
 #Example verification block
 show ip ospf neighbor #Checks OSPF neighbor relationships
 show ip route ospf #Verifies learned routes
```

**Packet Capture(s)**

- Capture confirming end to end connectivity from [](/config_labs)


---

## Troubleshooting
*Note: These troubleshooting steps are generalized to this lab and may not be valid for all <> troubleshooting scenarios. Once an issue is identified and resolved please re-run verification steps to confirm full functionality.

---

## Artifacts
| Type              | Location                                  |
|-------------------|-------------------------------------------|
| Configurations    | [configs/](/config_labs)                  |
| Diagram           | [diagram.png](/config_labs)               |
| Topology File     | [topology.yaml](/config_labs)             |
| Packet Capture(s) | [captures/](/config_labs)                 |
| Verification      | [verification_commands.txt](/config_labs) |

---

**Template Version Info**
 
| Field          | Value         |
|----------------|---------------|
| Lab Version    | 1.0           |
| Last Updated   | 2025-11-03    |
| Author         | Aaron Kindelt |