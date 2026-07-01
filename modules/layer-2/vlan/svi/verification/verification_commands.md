# SVI Inter-VLAN Routing Verification Output

This file contains recorded verification output for the SVI inter-VLAN routing lab.

The checks below confirm SVI state, VLAN creation, trunk behavior, native VLAN 99, connected SVI routes, access-port assignment, client gateway reachability, inter-VLAN connectivity, and routed paths with traceroute.

---

## D1

### `show ip interface brief`

**Expected Results**

* [x] SVIs for VLANs 10, 20, 30, and 40 are present.
* [x] SVIs are up/up.
* [x] Uplink interfaces Gi0/1, Gi0/2, Gi0/3, and Gi1/0 are up/up.
* [x] Physical Layer 3 IP addresses are not configured on the uplink interfaces.

```text
D1#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  down                  down    
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  up                    up      
GigabitEthernet0/3     unassigned      YES unset  up                    up      
GigabitEthernet1/0     unassigned      YES unset  up                    up      
GigabitEthernet1/1     unassigned      YES unset  down                  down    
GigabitEthernet1/2     unassigned      YES unset  down                  down    
GigabitEthernet1/3     unassigned      YES unset  down                  down    
Vlan10                 192.168.10.1    YES manual up                    up      
Vlan20                 192.168.20.1    YES manual up                    up      
Vlan30                 192.168.30.1    YES manual up                    up      
Vlan40                 192.168.40.1    YES manual up                    up     
```

### `show vlan brief`

**Expected Results**

* [x] VLANs 10, 20, 30, 40, and 99 are present.
* [x] VLAN 99 is present as the native VLAN.
* [x] No unexpected access ports are assigned to user VLANs on D1.

```text
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi1/1, Gi1/2, Gi1/3
10   USERS_10                         active    
20   USERS_20                         active    
30   USERS_30                         active    
40   USERS_40                         active    
99   NATIVE_99                        active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show interfaces trunk`

**Expected Results**

* [x] Trunk interfaces Gi0/1, Gi0/2, Gi0/3, and Gi1/0 are listed.
* [x] Encapsulation is 802.1Q.
* [x] Native VLAN is 99.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Required VLANs are forwarding and not pruned.

```text
D1#show interfaces trunk 

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      99
Gi0/2       on               802.1q         trunking      99
Gi0/3       on               802.1q         trunking      99
Gi1/0       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/1       10,20,30,40,99
Gi0/2       10,20,30,40,99
Gi0/3       10,20,30,40,99
Gi1/0       10,20,30,40,99

Port        Vlans allowed and active in management domain
Gi0/1       10,20,30,40,99
Gi0/2       10,20,30,40,99
Gi0/3       10,20,30,40,99
Gi1/0       10,20,30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       10,20,30,40,99
Gi0/2       10,20,30,40,99
Gi0/3       10,20,30,40,99
Gi1/0       10,20,30,40,99
```

### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 is participating in STP.
* [x] Required trunk interfaces are forwarding.
* [x] No required interface is blocking or inconsistent.

```text
D1#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     5254.0014.6d8d
             Cost        4
             Port        5 (GigabitEthernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.0061.e66a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p 
Gi0/3               Desg FWD 4         128.4    P2p 
Gi1/0               Root FWD 4         128.5    P2p 
```

### `show spanning-tree vlan 20`

**Expected Results**

* [x] VLAN 20 is participating in STP.
* [x] Required trunk interfaces are forwarding.
* [x] No required interface is blocking or inconsistent.

```text
D1#show spanning-tree vlan 20

VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    32788
             Address     5254.0014.6d8d
             Cost        4
             Port        5 (GigabitEthernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     5254.0061.e66a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p 
Gi0/3               Desg FWD 4         128.4    P2p 
Gi1/0               Root FWD 4         128.5    P2p 
```

### `show spanning-tree vlan 30`

**Expected Results**

* [x] VLAN 30 is participating in STP.
* [x] Required trunk interfaces are forwarding.
* [x] No required interface is blocking or inconsistent.

```text
D1#show spanning-tree vlan 30

VLAN0030
  Spanning tree enabled protocol ieee
  Root ID    Priority    32798
             Address     5254.0014.6d8d
             Cost        4
             Port        5 (GigabitEthernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     5254.0061.e66a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p 
Gi0/3               Desg FWD 4         128.4    P2p 
Gi1/0               Root FWD 4         128.5    P2p 
```

### `show spanning-tree vlan 40`

**Expected Results**

* [x] VLAN 40 is participating in STP.
* [x] Required trunk interfaces are forwarding.
* [x] No required interface is blocking or inconsistent.

```text
D1#show spanning-tree vlan 40 

VLAN0040
  Spanning tree enabled protocol ieee
  Root ID    Priority    32808
             Address     5254.0014.6d8d
             Cost        4
             Port        5 (GigabitEthernet1/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32808  (priority 32768 sys-id-ext 40)
             Address     5254.0061.e66a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p 
Gi0/3               Desg FWD 4         128.4    P2p 
Gi1/0               Root FWD 4         128.5    P2p 
```

### `show ip route`

**Expected Results**

* [x] Connected routes exist for VLANs 10, 20, 30, and 40.
* [x] Local /32 routes exist for each SVI IP.
* [x] Routes point to the matching SVI interfaces.

```text
D1#show ip route
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
C        192.168.10.0/24 is directly connected, Vlan10
L        192.168.10.1/32 is directly connected, Vlan10
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, Vlan20
L        192.168.20.1/32 is directly connected, Vlan20
      192.168.30.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.30.0/24 is directly connected, Vlan30
L        192.168.30.1/32 is directly connected, Vlan30
      192.168.40.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.40.0/24 is directly connected, Vlan40
L        192.168.40.1/32 is directly connected, Vlan40
```

### `show running-config interface gi0/1`

**Expected Results**

* [x] Interface description is correct.
* [x] Interface is configured as a trunk.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Native VLAN is 99.
* [x] BPDU Guard is disabled on the trunk.

```text
D1#show running-config interface gi0/1
Building configuration...

Current configuration : 253 bytes
!
interface GigabitEthernet0/1
 description Uplink to A1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 spanning-tree bpduguard disable
end
```

### `show running-config interface gi0/2`

**Expected Results**

* [x] Interface description is correct.
* [x] Interface is configured as a trunk.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Native VLAN is 99.
* [x] BPDU Guard is disabled on the trunk.

```text
D1#show running-config interface gi0/2
Building configuration...

Current configuration : 253 bytes
!
interface GigabitEthernet0/2
 description Uplink to A2
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 spanning-tree bpduguard disable
end
```

### `show running-config interface gi0/3`

**Expected Results**

* [x] Interface description is correct.
* [x] Interface is configured as a trunk.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Native VLAN is 99.
* [x] BPDU Guard is disabled on the trunk.

```text
D1#show running-config interface gi0/3
Building configuration...

Current configuration : 253 bytes
!
interface GigabitEthernet0/3
 description Uplink to A3
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 spanning-tree bpduguard disable
end
```

### `show running-config interface gi1/0`

**Expected Results**

* [x] Interface description is correct.
* [x] Interface is configured as a trunk.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Native VLAN is 99.
* [x] BPDU Guard is disabled on the trunk.

```text
D1#show running-config interface gi1/0
Building configuration...

Current configuration : 253 bytes
!
interface GigabitEthernet1/0
 description Uplink to A4
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 spanning-tree bpduguard disable
end
```

---

## Access Switches

### `show ip interface brief`

**Expected Results**

* [x] Gi0/0 uplink to D1 is up/up on each access switch.
* [x] Gi0/1 client-facing interface is up/up on each access switch.
* [x] Unused interfaces are down/down.

```text
A1>show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down    
```

```text
A2#show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down 
```

```text
A3>show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down    
```

```text
A4>show ip interface brief
Interface              IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0     unassigned      YES unset  up                    up      
GigabitEthernet0/1     unassigned      YES unset  up                    up      
GigabitEthernet0/2     unassigned      YES unset  down                  down    
GigabitEthernet0/3     unassigned      YES unset  down                  down    
```

### `show vlan brief`

**Expected Results**

* [x] VLANs 10, 20, 30, 40, and 99 are present.
* [x] The correct client-facing access port appears under the assigned VLAN on each access switch.
* [x] VLAN 99 is present as the native VLAN.

```text
A1>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3
10   USERS_10                         active    Gi0/1
20   USERS_20                         active    
30   USERS_30                         active    
40   USERS_40                         active    
99   NATIVE_99                        active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

```text
A2>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3
10   USERS_10                         active    
20   USERS_20                         active    Gi0/1
30   USERS_30                         active    
40   USERS_40                         active    
99   NATIVE_99                        active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

```text
A3>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3
10   USERS_10                         active    
20   USERS_20                         active    
30   USERS_30                         active    Gi0/1
40   USERS_40                         active    
99   NATIVE_99                        active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

```text
A4>show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3
10   USERS_10                         active    
20   USERS_20                         active    
30   USERS_30                         active    
40   USERS_40                         active    Gi0/1
99   NATIVE_99                        active    
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show interfaces trunk`

**Expected Results**

* [x] Gi0/0 is listed as a trunk on each access switch.
* [x] Encapsulation is 802.1Q.
* [x] Native VLAN is 99.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Required VLANs are forwarding and not pruned.

```text
A1>show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,40,99

Port        Vlans allowed and active in management domain
Gi0/0       10,20,30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10,20,30,40,99
```

```text
A2>show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,40,99

Port        Vlans allowed and active in management domain
Gi0/0       10,20,30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10,20,30,40,99
```

```text
A3>show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,40,99

Port        Vlans allowed and active in management domain
Gi0/0       10,20,30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10,20,30,40,99
```

```text
A4>show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      99

Port        Vlans allowed on trunk
Gi0/0       10,20,30,40,99

Port        Vlans allowed and active in management domain
Gi0/0       10,20,30,40,99

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10,20,30,40,99
```

### `show spanning-tree vlan 10`

**Expected Results**

* [x] A1 participates in VLAN 10 spanning tree.
* [x] Gi0/0 is Root and Forwarding.
* [x] Gi0/1 is Designated and Forwarding.

```text
A1>show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    32778
             Address     5254.0014.6d8d
             Cost        8
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.0076.b468
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### `show spanning-tree vlan 20`

**Expected Results**

* [x] A2 participates in VLAN 20 spanning tree.
* [x] Gi0/0 is Root and Forwarding.
* [x] Gi0/1 is Designated and Forwarding.

```text
A2>show spanning-tree vlan 20

VLAN0020
  Spanning tree enabled protocol ieee
  Root ID    Priority    32788
             Address     5254.0014.6d8d
             Cost        8
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32788  (priority 32768 sys-id-ext 20)
             Address     5254.00c7.88ea
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### `show spanning-tree vlan 30`

**Expected Results**

* [x] A3 participates in VLAN 30 spanning tree.
* [x] Gi0/0 is Root and Forwarding.
* [x] Gi0/1 is Designated and Forwarding.

```text
A3>show spanning-tree vlan 30

VLAN0030
  Spanning tree enabled protocol ieee
  Root ID    Priority    32798
             Address     5254.0014.6d8d
             Cost        8
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32798  (priority 32768 sys-id-ext 30)
             Address     5254.00c0.f725
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### `show spanning-tree vlan 40`

**Expected Results**

* [x] A4 participates in VLAN 40 spanning tree.
* [x] A4 is the root bridge for VLAN 40 in the recorded output.
* [x] Gi0/0 and Gi0/1 are Designated and Forwarding.

```text
A4>show spanning-tree vlan 40

VLAN0040
  Spanning tree enabled protocol ieee
  Root ID    Priority    32808
             Address     5254.0014.6d8d
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32808  (priority 32768 sys-id-ext 40)
             Address     5254.0014.6d8d
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### `show running-config interface gi0/0`

**Expected Results**

* [x] Access switch uplink is configured as a trunk.
* [x] Allowed VLANs are 10,20,30,40,99.
* [x] Native VLAN is 99.
* [x] BPDU Guard is disabled on the trunk.
* [x] CDP is disabled on the trunk in the captured configuration.

```text
A1#show running-config int gi0/0
Building configuration...

Current configuration : 268 bytes
!
interface GigabitEthernet0/0
 description Uplink to D1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 no cdp enable
 spanning-tree bpduguard disable
end
```

```text
A2#show running-config interface gi0/0
Building configuration...

Current configuration : 268 bytes
!
interface GigabitEthernet0/0
 description Uplink to D1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 no cdp enable
 spanning-tree bpduguard disable
end
```

```text
A3#show running-config interface Gi0/0
Building configuration...

Current configuration : 268 bytes
!
interface GigabitEthernet0/0
 description Uplink to D1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 no cdp enable
 spanning-tree bpduguard disable
end
```

```text
A4#show running-config interface gi0/0
Building configuration...

Current configuration : 268 bytes
!
interface GigabitEthernet0/0
 description uplink to D1
 switchport trunk allowed vlan 10,20,30,40,99
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 99
 switchport mode trunk
 negotiation auto
 no cdp enable
 spanning-tree bpduguard disable
end
```

---

## Connectivity

### D1 SVI Gateway Self-Test

**Expected Results**

* [x] VLAN 10 SVI `192.168.10.1` responds.
* [x] VLAN 20 SVI `192.168.20.1` responds.
* [x] VLAN 30 SVI `192.168.30.1` responds.
* [x] VLAN 40 SVI `192.168.40.1` responds.

```text
D1#ping 192.168.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

```text
D1#ping 192.168.20.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.20.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

```text
D1#ping 192.168.30.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.30.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

```text
D1#ping 192.168.40.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.40.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms
```

### Client to Gateway Tests

**Expected Results**

* [x] C1 reaches VLAN 10 gateway.
* [x] C2 reaches VLAN 20 gateway.
* [x] C3 reaches VLAN 30 gateway.
* [x] C4 reaches VLAN 40 gateway.

```text
C1:~# ping -w 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1): 56 data bytes
64 bytes from 192.168.10.1: seq=0 ttl=255 time=2.747 ms
64 bytes from 192.168.10.1: seq=1 ttl=255 time=2.592 ms
64 bytes from 192.168.10.1: seq=2 ttl=255 time=2.630 ms
64 bytes from 192.168.10.1: seq=3 ttl=255 time=3.019 ms
64 bytes from 192.168.10.1: seq=4 ttl=255 time=2.700 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.592/2.737/3.019 ms
```

```text
C2:~# ping -w 5 192.168.20.1
PING 192.168.20.1 (192.168.20.1): 56 data bytes
64 bytes from 192.168.20.1: seq=0 ttl=255 time=2.830 ms
64 bytes from 192.168.20.1: seq=1 ttl=255 time=2.357 ms
64 bytes from 192.168.20.1: seq=2 ttl=255 time=2.548 ms
64 bytes from 192.168.20.1: seq=3 ttl=255 time=2.644 ms
64 bytes from 192.168.20.1: seq=4 ttl=255 time=2.694 ms

--- 192.168.20.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.357/2.614/2.830 ms
```

```text
C3:~# ping -w 5 192.168.30.1
PING 192.168.30.1 (192.168.30.1): 56 data bytes
64 bytes from 192.168.30.1: seq=0 ttl=255 time=2.789 ms
64 bytes from 192.168.30.1: seq=1 ttl=255 time=2.775 ms
64 bytes from 192.168.30.1: seq=2 ttl=255 time=2.655 ms
64 bytes from 192.168.30.1: seq=3 ttl=255 time=2.629 ms
64 bytes from 192.168.30.1: seq=4 ttl=255 time=2.708 ms

--- 192.168.30.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.629/2.711/2.789 ms
```

```text
C4:~# ping -w 5 192.168.40.1
PING 192.168.40.1 (192.168.40.1): 56 data bytes
64 bytes from 192.168.40.1: seq=0 ttl=255 time=2.690 ms
64 bytes from 192.168.40.1: seq=1 ttl=255 time=2.630 ms
64 bytes from 192.168.40.1: seq=2 ttl=255 time=2.480 ms
64 bytes from 192.168.40.1: seq=3 ttl=255 time=2.647 ms
64 bytes from 192.168.40.1: seq=4 ttl=255 time=2.615 ms

--- 192.168.40.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.480/2.612/2.690 ms
```

### Inter-VLAN Client Tests

**Expected Results**

* [x] C1 reaches a host outside VLAN 10.
* [x] C2 reaches a host outside VLAN 20.
* [x] C3 reaches a host outside VLAN 30.
* [x] C4 reaches a host outside VLAN 40.

```text
C1:~# ping -w 5 192.168.20.20
PING 192.168.20.20 (192.168.20.20): 56 data bytes
64 bytes from 192.168.20.20: seq=0 ttl=63 time=4.672 ms
64 bytes from 192.168.20.20: seq=1 ttl=63 time=3.522 ms
64 bytes from 192.168.20.20: seq=2 ttl=63 time=3.934 ms
64 bytes from 192.168.20.20: seq=3 ttl=63 time=3.545 ms
64 bytes from 192.168.20.20: seq=4 ttl=63 time=3.964 ms

--- 192.168.20.20 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 3.522/3.927/4.672 ms
```

```text
C2:~# ping -w 5 192.168.30.30
PING 192.168.30.30 (192.168.30.30): 56 data bytes
64 bytes from 192.168.30.30: seq=0 ttl=63 time=3.875 ms
64 bytes from 192.168.30.30: seq=1 ttl=63 time=3.394 ms
64 bytes from 192.168.30.30: seq=2 ttl=63 time=3.861 ms
64 bytes from 192.168.30.30: seq=3 ttl=63 time=3.387 ms
64 bytes from 192.168.30.30: seq=4 ttl=63 time=4.242 ms

--- 192.168.30.30 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 3.387/3.751/4.242 ms
```

```text
C3:~# ping -w 5 192.168.40.40
PING 192.168.40.40 (192.168.40.40): 56 data bytes
64 bytes from 192.168.40.40: seq=0 ttl=63 time=3.978 ms
64 bytes from 192.168.40.40: seq=1 ttl=63 time=3.960 ms
64 bytes from 192.168.40.40: seq=2 ttl=63 time=4.425 ms
64 bytes from 192.168.40.40: seq=3 ttl=63 time=3.683 ms
64 bytes from 192.168.40.40: seq=4 ttl=63 time=4.260 ms

--- 192.168.40.40 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 3.683/4.061/4.425 ms
```

```text
C4:~# ping -w 5 192.168.20.20
PING 192.168.20.20 (192.168.20.20): 56 data bytes
64 bytes from 192.168.20.20: seq=0 ttl=63 time=4.291 ms
64 bytes from 192.168.20.20: seq=1 ttl=63 time=4.317 ms
64 bytes from 192.168.20.20: seq=2 ttl=63 time=3.936 ms
64 bytes from 192.168.20.20: seq=3 ttl=63 time=3.957 ms
64 bytes from 192.168.20.20: seq=4 ttl=63 time=3.945 ms

--- 192.168.20.20 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 3.936/4.089/4.317 ms
```

### Traceroute Tests

**Expected Results**

* [x] Traceroute from each client reaches the appropriate SVI gateway first.
* [x] The second hop is the destination client in the remote VLAN.
* [x] Routed paths match the intended SVI design.

```text
C1:~# traceroute 192.168.20.20
traceroute to 192.168.20.20 (192.168.20.20), 30 hops max, 46 byte packets
 1  192.168.10.1 (192.168.10.1)  2.693 ms  2.613 ms  2.677 ms
 2  192.168.20.20 (192.168.20.20)  4.204 ms  3.889 ms  3.158 ms
```

```text
C2:~# traceroute 192.168.30.30
traceroute to 192.168.30.30 (192.168.30.30), 30 hops max, 46 byte packets
 1  192.168.20.1 (192.168.20.1)  2.852 ms  2.654 ms  2.614 ms
 2  192.168.30.30 (192.168.30.30)  3.460 ms  3.449 ms  3.606 ms
```

```text
C3:~# traceroute 192.168.40.40
traceroute to 192.168.40.40 (192.168.40.40), 30 hops max, 46 byte packets
 1  192.168.30.1 (192.168.30.1)  3.004 ms  2.729 ms  2.575 ms
 2  192.168.40.40 (192.168.40.40)  3.596 ms  4.181 ms  3.545 ms
```

```text
C4:~# traceroute 192.168.20.20
traceroute to 192.168.20.20 (192.168.20.20), 30 hops max, 46 byte packets
 1  192.168.40.1 (192.168.40.1)  2.591 ms  2.512 ms  2.462 ms
 2  192.168.20.20 (192.168.20.20)  3.759 ms  4.008 ms  3.767 ms
```
