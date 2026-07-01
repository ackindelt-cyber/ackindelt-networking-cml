# VLAN Creation Verification Output

This file contains recorded verification output for the VLAN Creation lab.

The checks below confirm VLAN creation, access-port assignment, STP forwarding state, PortFast and BPDU Guard configuration, MAC address learning, same-VLAN connectivity, and isolation between VLANs when no Layer 3 routing is configured.

---

## S1

### `show vlan brief`

**Expected Results**

* [x] VLAN 10 exists, is named `USERS_10`, and is active.
* [x] VLAN 20 exists, is named `USERS_20`, and is active.
* [x] Gi0/0 and Gi0/1 are assigned to VLAN 10.
* [x] Gi0/2 and Gi0/3 are assigned to VLAN 20.

```text
S1>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi1/0, Gi1/1, Gi1/2, Gi1/3
10   USERS_10                         active    Gi0/0, Gi0/1
20   USERS_20                         active    Gi0/2, Gi0/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show spanning-tree summary`

**Expected Results**

* [x] S1 is the root bridge for VLAN 1, VLAN 10, and VLAN 20 in this single-switch topology.
* [x] VLAN 1 shows four forwarding ports.
* [x] VLAN 10 shows two forwarding ports.
* [x] VLAN 20 shows two forwarding ports.
* [x] No ports are blocking.

```text
S1>show spanning-tree summary
Switch is in pvst mode
Root bridge for: VLAN0001, VLAN0010, VLAN0020
Extended system ID                      is enabled
Portfast Default                        is disabled
Portfast Edge BPDU Guard Default        is disabled
Portfast Edge BPDU Filter Default       is disabled
Loopguard Default                       is disabled
PVST Simulation Default                 is enabled but inactive in pvst mode
Bridge Assurance                        is enabled but inactive in pvst mode
EtherChannel misconfig guard            is enabled
Configured Pathcost method used is short
UplinkFast                              is disabled
BackboneFast                            is disabled

Name                   Blocking Listening Learning Forwarding STP Active
---------------------- -------- --------- -------- ---------- ----------
VLAN0001                     0         0        0          4          4
VLAN0010                     0         0        0          2          2
VLAN0020                     0         0        0          2          2
---------------------- -------- --------- -------- ---------- ----------
3 vlans                      0         0        0          8          8
```

### `show running-config | section interface`

**Expected Results**

* [x] Gi0/0 is an access port in VLAN 10 with PortFast and BPDU Guard enabled.
* [x] Gi0/1 is an access port in VLAN 10 with PortFast and BPDU Guard enabled.
* [x] Gi0/2 is an access port in VLAN 20 with PortFast and BPDU Guard enabled.
* [x] Gi0/3 is an access port in VLAN 20 with PortFast and BPDU Guard enabled.

```text
S1#show running-config | section interface
interface GigabitEthernet0/0
 switchport access vlan 10
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
interface GigabitEthernet0/1
 switchport access vlan 10
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
interface GigabitEthernet0/2
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
interface GigabitEthernet0/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
```

### `show mac address-table`

**Expected Results**

* [x] Client MAC addresses are learned in VLAN 10 on Gi0/0 and Gi0/1.
* [x] Client MAC addresses are learned in VLAN 20 on Gi0/2 and Gi0/3.
* [x] Four dynamic MAC addresses are present.

```text
S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    5254.001e.030b    DYNAMIC     Gi0/0
  10    5254.0082.b3ec    DYNAMIC     Gi0/1
  20    5254.007a.158b    DYNAMIC     Gi0/3
  20    5254.00f1.d40a    DYNAMIC     Gi0/2
Total Mac Addresses for this criterion: 4
```

---

## C1

### `ping 192.168.10.11`

**Expected Results**

* [x] ICMP replies are received from C2.
* [x] Same-VLAN connectivity within VLAN 10 is successful.

```text
C1:~# ping -w 5 192.168.10.11
PING 192.168.10.11 (192.168.10.11): 56 data bytes
64 bytes from 192.168.10.11: seq=0 ttl=64 time=1.146 ms
64 bytes from 192.168.10.11: seq=1 ttl=64 time=1.741 ms
64 bytes from 192.168.10.11: seq=2 ttl=64 time=2.005 ms
64 bytes from 192.168.10.11: seq=3 ttl=64 time=1.968 ms
64 bytes from 192.168.10.11: seq=4 ttl=64 time=1.171 ms

--- 192.168.10.11 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.146/1.606/2.005 ms
```

### `ping 192.168.20.10`

**Expected Results**

* [x] No ICMP replies are received from C3.
* [x] VLAN 10 cannot reach VLAN 20 because no Layer 3 routing is configured.

```text
C1:~# ping -w 5 192.168.20.10
PING 192.168.20.10 (192.168.20.10): 56 data bytes

--- 192.168.20.10 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

### `ping 192.168.20.11`

**Expected Results**

* [x] No ICMP replies are received from C4.
* [x] VLAN 10 cannot reach VLAN 20 because no Layer 3 routing is configured.

```text
C1:~# ping -w 5 192.168.20.11
PING 192.168.20.11 (192.168.20.11): 56 data bytes

--- 192.168.20.11 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

---

## C3

### `ping 192.168.20.11`

**Expected Results**

* [x] ICMP replies are received from C4.
* [x] Same-VLAN connectivity within VLAN 20 is successful.

```text
C3:~# ping -w 5 192.168.20.11
PING 192.168.20.11 (192.168.20.11): 56 data bytes
64 bytes from 192.168.20.11: seq=0 ttl=64 time=3.321 ms
64 bytes from 192.168.20.11: seq=1 ttl=64 time=2.087 ms
64 bytes from 192.168.20.11: seq=2 ttl=64 time=1.824 ms
64 bytes from 192.168.20.11: seq=3 ttl=64 time=1.957 ms
64 bytes from 192.168.20.11: seq=4 ttl=64 time=2.188 ms

--- 192.168.20.11 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.824/2.275/3.321 ms
```

### `ping 192.168.10.10`

**Expected Results**

* [x] No ICMP replies are received from C1.
* [x] VLAN 20 cannot reach VLAN 10 because no Layer 3 routing is configured.

```text
C3:~# ping -w 5  192.168.10.10
PING 192.168.10.10 (192.168.10.10): 56 data bytes

--- 192.168.10.10 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

### `ping 192.168.10.11`

**Expected Results**

* [x] No ICMP replies are received from C2.
* [x] VLAN 20 cannot reach VLAN 10 because no Layer 3 routing is configured.

```text
C3:~# ping -w 5  192.168.10.11
PING 192.168.10.11 (192.168.10.11): 56 data bytes

--- 192.168.10.11 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```
