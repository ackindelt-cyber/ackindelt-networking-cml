# VLAN Trunking Verification Output

This file contains recorded verification output for the VLAN Trunking lab.

The checks below confirm VLAN creation, access-port assignment, trunk configuration, allowed VLAN behavior, MAC address learning across the trunk, same-VLAN connectivity across switches, and isolation between VLANs when no Layer 3 routing is configured.

---

## S1

### `show ip interface brief`

**Expected Results**

* [x] Gi0/1 trunk link is up/up.
* [x] Gi0/2 VLAN 10 access port is up/up.
* [x] Gi0/3 VLAN 20 access port is up/up.
* [x] Gi0/0 is unused and down/down.

```text
S1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  down                  down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up    
```

### `show vlan brief`

**Expected Results**

* [x] VLAN 10 exists, is named `USERS_10`, and is active.
* [x] VLAN 20 exists, is named `USERS_20`, and is active.
* [x] Gi0/2 is assigned to VLAN 10.
* [x] Gi0/3 is assigned to VLAN 20.

```text
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0
10   USERS_10                         active    Gi0/2
20   USERS_20                         active    Gi0/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show spanning-tree summary`

**Expected Results**

* [x] Switch is running PVST.
* [x] VLANs 1, 10, and 20 have active forwarding ports.
* [x] No ports are in blocking, listening, or learning state.

```text
S1#show spanning-tree summary
Switch is in pvst mode
Root bridge for: VLAN0001
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
VLAN0001                     0         0        0          1          1
VLAN0010                     0         0        0          2          2
VLAN0020                     0         0        0          2          2
---------------------- -------- --------- -------- ---------- ----------
3 vlans                      0         0        0          5          5
```

### `show running-config interface gigabitEthernet0/2`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 10.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S1#show running-config interface gigabitethernet 0/2
Building configuration...

Current configuration : 166 bytes
!
interface GigabitEthernet0/2
 switchport access vlan 10
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
end
```

### `show running-config interface gigabitEthernet0/3`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 20.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S1#show running-config interface gigabitethernet 0/3
Building configuration...

Current configuration : 166 bytes
!
interface GigabitEthernet0/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
end
```

### `show running-config interface gigabitEthernet0/1`

**Expected Results**

* [x] Interface is configured as a trunk.
* [x] 802.1Q encapsulation is configured.
* [x] VLANs 10 and 20 are allowed on the trunk.
* [x] Native VLAN is the default VLAN 1.

```text
S1#show running-config interface gigabitethernet 0/1
Building configuration...

Current configuration : 152 bytes
!
interface GigabitEthernet0/1
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
end
```

### `show mac address-table`

**Expected Results**

* [x] Local VLAN 10 client MAC is learned on Gi0/2.
* [x] Remote VLAN 10 client MAC is learned through trunk Gi0/1.
* [x] Local VLAN 20 client MAC is learned on Gi0/3.
* [x] Remote VLAN 20 client MAC is learned through trunk Gi0/1.
* [x] Dynamic MAC addresses are learned in the expected VLANs.

```text
S1#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    5254.0038.e872    DYNAMIC     Gi0/2
  10    5254.006f.7617    DYNAMIC     Gi0/1
  10    5254.008a.cb59    DYNAMIC     Gi0/1
  20    5254.0018.6587    DYNAMIC     Gi0/3
  20    5254.006f.7617    DYNAMIC     Gi0/1
  20    5254.00c6.7e4e    DYNAMIC     Gi0/1
Total Mac Addresses for this criterion: 6
```

### `show interfaces switchport`

**Expected Results**

* [x] Gi0/1 is operationally trunking.
* [x] Gi0/1 allows VLANs 10 and 20.
* [x] Gi0/2 is an access port in VLAN 10.
* [x] Gi0/3 is an access port in VLAN 20.
* [x] Gi0/0 is unused and operationally down.

```text
S1#show interfaces switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: down
Administrative Trunking Encapsulation: negotiate
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: 10,20
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/2
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (USERS_10)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/3
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 20 (USERS_20)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none
```

---

## S2

### `show ip interface brief`

**Expected Results**

* [x] Gi0/1 trunk link is up/up.
* [x] Gi0/2 VLAN 10 access port is up/up.
* [x] Gi0/3 VLAN 20 access port is up/up.
* [x] Gi0/0 is unused and down/down.

```text
S2#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  down                  down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up      
```

### `show vlan brief`

**Expected Results**

* [x] VLAN 10 exists, is named `USERS_10`, and is active.
* [x] VLAN 20 exists, is named `USERS_20`, and is active.
* [x] Gi0/2 is assigned to VLAN 10.
* [x] Gi0/3 is assigned to VLAN 20.

```text
S2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0
10   USERS_10                         active    Gi0/2
20   USERS_20                         active    Gi0/3
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show spanning-tree summary`

**Expected Results**

* [x] Switch is running PVST.
* [x] S2 is root for VLANs 1, 10, and 20 in the recorded output.
* [x] VLANs 1, 10, and 20 have active forwarding ports.
* [x] No ports are in blocking, listening, or learning state.

```text
S2#show spanning-tree summary
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
VLAN0001                     0         0        0          1          1
VLAN0010                     0         0        0          2          2
VLAN0020                     0         0        0          2          2
---------------------- -------- --------- -------- ---------- ----------
3 vlans                      0         0        0          5          5
```

### `show running-config interface gigabitEthernet0/2`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 10.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S2#show running-config interface gigabitethernet 0/2
Building configuration...

Current configuration : 166 bytes
!
interface GigabitEthernet0/2
 switchport access vlan 10
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
end
```

### `show running-config interface gigabitEthernet0/3`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 20.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S2#show running-config interface gigabitethernet 0/3
Building configuration...

Current configuration : 166 bytes
!
interface GigabitEthernet0/3
 switchport access vlan 20
 switchport mode access
 negotiation auto
 spanning-tree portfast edge
 spanning-tree bpduguard enable
end
```

### `show running-config interface gigabitEthernet0/1`

**Expected Results**

* [x] Interface is configured as a trunk.
* [x] 802.1Q encapsulation is configured.
* [x] VLANs 10 and 20 are allowed on the trunk.
* [x] Native VLAN is the default VLAN 1.

```text
S2#show running-config interface gigabitethernet 0/1
Building configuration...

Current configuration : 152 bytes
!
interface GigabitEthernet0/1
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
end
```

### `show mac address-table`

**Expected Results**

* [x] Remote VLAN 10 client MAC is learned through trunk Gi0/1.
* [x] Local VLAN 10 client MAC is learned on Gi0/2.
* [x] Remote VLAN 20 client MAC is learned through trunk Gi0/1.
* [x] Local VLAN 20 client MAC is learned on Gi0/3.
* [x] Dynamic MAC addresses are learned in the expected VLANs.

**Note:** The absence of a separate S1 trunk-interface MAC entry is not treated as a failure here. The required validation is client MAC learning across the trunk.

```text
S2#show mac address-table
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    5254.0038.e872    DYNAMIC     Gi0/1
  10    5254.008a.cb59    DYNAMIC     Gi0/2
  20    5254.0018.6587    DYNAMIC     Gi0/1
  20    5254.00c6.7e4e    DYNAMIC     Gi0/3
Total Mac Addresses for this criterion: 4
```

### `show interfaces switchport`

**Expected Results**

* [x] Gi0/1 is operationally trunking.
* [x] Gi0/1 allows VLANs 10 and 20.
* [x] Gi0/2 is an access port in VLAN 10.
* [x] Gi0/3 is an access port in VLAN 20.
* [x] Gi0/0 is unused and operationally down.

```text
S2#show interfaces switchport
Name: Gi0/0
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: down
Administrative Trunking Encapsulation: negotiate
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/1
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: 10,20
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/2
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (USERS_10)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none

Name: Gi0/3
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 20 (USERS_20)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
Administrative private-vlan host-association: none 
Administrative private-vlan mapping: none 
Administrative private-vlan trunk native VLAN: none
Administrative private-vlan trunk Native VLAN tagging: enabled
Administrative private-vlan trunk encapsulation: dot1q
Administrative private-vlan trunk normal VLANs: none
Administrative private-vlan trunk associations: none
Administrative private-vlan trunk mappings: none
Operational private-vlan: none
Trunking VLANs Enabled: ALL
Pruning VLANs Enabled: 2-1001
Capture Mode Disabled
Capture VLANs Allowed: ALL

Protected: false
Appliance trust: none
```

---

## C1

### `ping 192.168.20.2`

**Expected Results**

* [x] No ICMP replies are received from C2.
* [x] VLAN 10 cannot reach VLAN 20 because no Layer 3 routing is configured.

```text
C1:~# ping -w 5 192.168.20.2
PING 192.168.20.2 (192.168.20.2): 56 data bytes

--- 192.168.20.2 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

### `ping 192.168.10.3`

**Expected Results**

* [x] ICMP replies are received from C3.
* [x] VLAN 10 connectivity works across the trunk.

```text
C1:~# ping -w 5 192.168.10.3
PING 192.168.10.3 (192.168.10.3): 56 data bytes
64 bytes from 192.168.10.3: seq=0 ttl=64 time=2.405 ms
64 bytes from 192.168.10.3: seq=1 ttl=64 time=2.096 ms
64 bytes from 192.168.10.3: seq=2 ttl=64 time=2.631 ms
64 bytes from 192.168.10.3: seq=3 ttl=64 time=2.542 ms
64 bytes from 192.168.10.3: seq=4 ttl=64 time=2.203 ms

--- 192.168.10.3 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.096/2.375/2.631 ms
```

### `ping 192.168.20.3`

**Expected Results**

* [x] No ICMP replies are received from C4.
* [x] VLAN 10 cannot reach VLAN 20 because no Layer 3 routing is configured.

```text
C1:~# ping -w 5 192.168.20.3
PING 192.168.20.3 (192.168.20.3): 56 data bytes

--- 192.168.20.3 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

---

## C2

### `ping 192.168.10.2`

**Expected Results**

* [x] No ICMP replies are received from C1.
* [x] VLAN 20 cannot reach VLAN 10 because no Layer 3 routing is configured.

```text
PING 192.168.10.2 (192.168.10.2): 56 data bytes

--- 192.168.10.2 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

### `ping 192.168.10.3`

**Expected Results**

* [x] No ICMP replies are received from C3.
* [x] VLAN 20 cannot reach VLAN 10 because no Layer 3 routing is configured.

```text
C2:~# ping -w 5 192.168.10.3
PING 192.168.10.3 (192.168.10.3): 56 data bytes

--- 192.168.10.3 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss
```

### `ping 192.168.20.3`

**Expected Results**

* [x] ICMP replies are received from C4.
* [x] VLAN 20 connectivity works across the trunk.

```text
PING 192.168.20.3 (192.168.20.3): 56 data bytes
64 bytes from 192.168.20.3: seq=0 ttl=64 time=2.239 ms
64 bytes from 192.168.20.3: seq=1 ttl=64 time=2.292 ms
64 bytes from 192.168.20.3: seq=2 ttl=64 time=2.475 ms
64 bytes from 192.168.20.3: seq=3 ttl=64 time=1.969 ms
64 bytes from 192.168.20.3: seq=4 ttl=64 time=2.630 ms

--- 192.168.20.3 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.969/2.321/2.630 ms
C2:~# 
```
