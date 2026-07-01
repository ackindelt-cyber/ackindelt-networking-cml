# Router-on-a-Stick Verification Output

This file contains recorded verification output for the Router-on-a-Stick lab.

The checks below confirm router subinterfaces, 802.1Q encapsulation, connected routes, ARP entries, VLAN creation, access-port assignment, trunk behavior, spanning-tree state, and end-to-end ICMP connectivity.

---

## R1

### `show ip interface brief`

**Expected Results**

* [x] Physical interface Gi0/0 is up/up.
* [x] Subinterfaces Gi0/0.10 and Gi0/0.20 are up/up.
* [x] Subinterfaces have the expected IP addresses.

```text
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         unassigned      YES unset  up                    up      
GigabitEthernet0/0.10      192.168.10.1    YES manual up                    up      
GigabitEthernet0/0.20      192.168.20.1    YES manual up                    up      
GigabitEthernet0/1         unassigned      YES unset  administratively down down    
GigabitEthernet0/2         unassigned      YES unset  administratively down down    
GigabitEthernet0/3         unassigned      YES unset  administratively down down  
```

### `show interfaces gigabitEthernet0/0.10`

**Expected Results**

* [x] Interface is up/up.
* [x] IP address is `192.168.10.1/24`.
* [x] 802.1Q encapsulation is enabled.
* [x] VLAN ID is `10`.

```text
R1#show interfaces gigabitEthernet 0/0.10
GigabitEthernet0/0.10 is up, line protocol is up 
  Hardware is iGbE, address is 5254.0091.3f79 (bia 5254.0091.3f79)
  Internet address is 192.168.10.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID  10.
  ARP type: ARPA, ARP Timeout 04:00:00
  Keepalive set (10 sec)
  Last clearing of "show interface" counters never
```

### `show interfaces gigabitEthernet0/0.20`

**Expected Results**

* [x] Interface is up/up.
* [x] IP address is `192.168.20.1/24`.
* [x] 802.1Q encapsulation is enabled.
* [x] VLAN ID is `20`.

```text
R1#show interfaces gigabitEthernet 0/0.20
GigabitEthernet0/0.20 is up, line protocol is up 
  Hardware is iGbE, address is 5254.0091.3f79 (bia 5254.0091.3f79)
  Internet address is 192.168.20.1/24
  MTU 1500 bytes, BW 1000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation 802.1Q Virtual LAN, Vlan ID  20.
  ARP type: ARPA, ARP Timeout 04:00:00
  Keepalive set (10 sec)
  Last clearing of "show interface" counters never
```

### `show running-config interface gigabitEthernet0/0.10`

**Expected Results**

* [x] `encapsulation dot1Q 10` is present.
* [x] Correct IP address and subnet mask are configured.
* [x] No unexpected configuration is present.

```text
R1#show running-config interface gigabitEthernet 0/0.10
Building configuration...

Current configuration : 102 bytes
!
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
end
```

### `show running-config interface gigabitEthernet0/0.20`

**Expected Results**

* [x] `encapsulation dot1Q 20` is present.
* [x] Correct IP address and subnet mask are configured.
* [x] No unexpected configuration is present.

```text
R1#show running-config interface gigabitEthernet 0/0.20
Building configuration...

Current configuration : 102 bytes
!
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
end
```

### `show ip route`

**Expected Results**

* [x] Connected route exists for `192.168.10.0/24`.
* [x] Connected route exists for `192.168.20.0/24`.
* [x] Local /32 routes exist for both subinterface IP addresses.

```text
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L        192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L        192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
```

### `show arp`

**Expected Results**

* [x] ARP entries exist for hosts in VLAN 10.
* [x] ARP entries exist for hosts in VLAN 20.
* [x] MAC addresses map to the expected router subinterfaces.

```text
R1#show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  192.168.10.1            -   5254.0091.3f79  ARPA   GigabitEthernet0/0.10
Internet  192.168.10.2           18   5254.00ee.16d5  ARPA   GigabitEthernet0/0.10
Internet  192.168.10.3           28   5254.0008.af5e  ARPA   GigabitEthernet0/0.10
Internet  192.168.20.1            -   5254.0091.3f79  ARPA   GigabitEthernet0/0.20
Internet  192.168.20.2           17   5254.00c0.7a7b  ARPA   GigabitEthernet0/0.20
Internet  192.168.20.3           22   5254.00d8.5dd8  ARPA   GigabitEthernet0/0.20
```

---

## S1

### `show ip interface brief`

**Expected Results**

* [x] Expected active interfaces are up/up.
* [x] No intended production-facing lab interface is administratively down.

```text
S1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up      
```

### `show vlan brief`

**Expected Results**

* [x] VLAN 10 exists and is active.
* [x] VLAN 20 exists and is active.
* [x] Access ports are assigned to the correct VLANs.

```text
S1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    
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
* [x] VLANs 10 and 20 are forwarding.
* [x] No unexpected blocking ports are present.

```text
S1#show spanning-tree summary
Switch is in pvst mode
Root bridge for: VLAN0010, VLAN0020
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
VLAN0010                     0         0        0          3          3
VLAN0020                     0         0        0          3          3
---------------------- -------- --------- -------- ---------- ----------
2 vlans                      0         0        0          6          6
```

### `show running-config interface gigabitEthernet0/0`

**Expected Results**

* [x] Interface is configured as a trunk.
* [x] 802.1Q encapsulation is configured.
* [x] VLANs 10 and 20 are allowed on the trunk.

```text
S1#show running-config interface gigabitEthernet 0/0
Building configuration...

Current configuration : 152 bytes
!
interface GigabitEthernet0/0
 switchport trunk allowed vlan 10,20
 switchport trunk encapsulation dot1q
 switchport mode trunk
 negotiation auto
end
```

### `show running-config interface gigabitEthernet0/1`

**Expected Results**

* [x] Interface is configured as a trunk.
* [x] 802.1Q encapsulation is configured.
* [x] VLANs 10 and 20 are allowed on the trunk.

```text
S1#show running-config interface gigabitEthernet 0/1
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

### `show running-config interface gigabitEthernet0/2`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 10.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S1#show running-config interface gigabitEthernet 0/2
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
S1#show running-config interface gigabitEthernet 0/3
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

### `show interfaces switchport`

**Expected Results**

* [x] Trunk ports show operational mode trunk.
* [x] Access ports show operational mode static access.
* [x] Correct VLAN assignments are displayed.
* [x] 802.1Q encapsulation is confirmed on trunk ports.

```text
S1#show interfaces switchport
Name: Gi0/0
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

* [x] Expected active interfaces Gi0/1, Gi0/2, and Gi0/3 are up/up.
* [x] Gi0/0 is unused in this topology and is down/down.

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

* [x] VLAN 10 exists and is active.
* [x] VLAN 20 exists and is active.
* [x] Access ports are assigned to the correct VLANs.

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
* [x] VLANs 10 and 20 are forwarding.
* [x] No unexpected blocking ports are present.

```text
S2#show spanning-tree summary
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

### `show running-config interface gigabitEthernet0/1`

**Expected Results**

* [x] Interface is configured as a trunk.
* [x] 802.1Q encapsulation is configured.
* [x] VLANs 10 and 20 are allowed on the trunk.

```text
S2#show running-config interface gigabitEthernet 0/1
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

### `show running-config interface gigabitEthernet0/2`

**Expected Results**

* [x] Interface is configured as an access port.
* [x] Interface is assigned to VLAN 10.
* [x] PortFast edge behavior is enabled.
* [x] BPDU Guard is enabled.

```text
S2#show running-config interface gigabitEthernet 0/2
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
S2#show running-config interface gigabitEthernet 0/3
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

### `show interfaces switchport`

**Expected Results**

* [x] Trunk port shows operational mode trunk.
* [x] Access ports show operational mode static access.
* [x] Correct VLAN assignments are displayed.
* [x] 802.1Q encapsulation is confirmed on the trunk port.
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

### `ping 192.168.10.3`

**Expected Results**

* [x] ICMP replies are received from C3.
* [x] Same-VLAN connectivity within VLAN 10 is successful.

```text
C1:/etc/network# ping -w 5 192.168.10.3
PING 192.168.10.3 (192.168.10.3): 56 data bytes
64 bytes from 192.168.10.3: seq=0 ttl=64 time=2.361 ms
64 bytes from 192.168.10.3: seq=1 ttl=64 time=2.322 ms
64 bytes from 192.168.10.3: seq=2 ttl=64 time=2.597 ms
64 bytes from 192.168.10.3: seq=3 ttl=64 time=2.411 ms
64 bytes from 192.168.10.3: seq=4 ttl=64 time=2.190 ms
```

### `ping 192.168.20.2`

**Expected Results**

* [x] ICMP replies are received from C2.
* [x] Inter-VLAN connectivity from VLAN 10 to VLAN 20 is successful.

```text
C1:/etc/network# ping -w 5 192.168.20.2
PING 192.168.20.2 (192.168.20.2): 56 data bytes
64 bytes from 192.168.20.2: seq=0 ttl=63 time=2.756 ms
64 bytes from 192.168.20.2: seq=1 ttl=63 time=2.787 ms
64 bytes from 192.168.20.2: seq=2 ttl=63 time=2.785 ms
64 bytes from 192.168.20.2: seq=3 ttl=63 time=2.554 ms
64 bytes from 192.168.20.2: seq=4 ttl=63 time=3.036 ms
```

### `ping 192.168.20.3`

**Expected Results**

* [x] ICMP replies are received from C4.
* [x] Inter-VLAN connectivity from VLAN 10 to VLAN 20 across both switches is successful.

```text
C1:/etc/network# ping -w 5 192.168.20.3
PING 192.168.20.3 (192.168.20.3): 56 data bytes
64 bytes from 192.168.20.3: seq=0 ttl=63 time=3.821 ms
64 bytes from 192.168.20.3: seq=1 ttl=63 time=4.344 ms
64 bytes from 192.168.20.3: seq=2 ttl=63 time=3.711 ms
64 bytes from 192.168.20.3: seq=3 ttl=63 time=3.772 ms
64 bytes from 192.168.20.3: seq=4 ttl=63 time=3.185 ms
```
