# Edge Firewall HA Verification Output

This file contains recorded verification output for the Edge Firewall HA lab.

The checks below confirm the minimum normal-state evidence required for interface and VLAN state, routing, HSRP, Spanning Tree, ASAv policy and failover health, dynamic PAT, and end-to-end connectivity.

---

## ISP

### `show ip interface brief`

**Expected Results**

* [x] `Gi0/0` is `198.51.100.1` and `Loopback0` is `192.0.2.100`; both are `up/up`.

```text
ISP#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         198.51.100.1    YES NVRAM  up                    up      
GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down    
Loopback0                  192.0.2.100     YES NVRAM  up                    up
```

### `show ip route static`

**Expected Results**

* [x] The return route for `203.0.113.0/29` is installed via `198.51.100.2`.

```text
ISP#show ip route static

Gateway of last resort is not set

      203.0.113.0/29 is subnetted, 1 subnets
S        203.0.113.0 [1/0] via 198.51.100.2
```

### `ping 203.0.113.2`

**Expected Results**

* [x] The active firewall `outside` address is reachable through R1.

```text
ISP#ping 203.0.113.2
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 203.0.113.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

---

## R1

### `show ip interface brief`

**Expected Results**

* [x] `Gi0/0` is `203.0.113.1` and `Gi0/1` is `198.51.100.2`; both are `up/up`.

```text
R1#show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         203.0.113.1     YES NVRAM  up                    up      
GigabitEthernet0/1         198.51.100.2    YES NVRAM  up                    up      
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down
```

### `show ip route static`

**Expected Results**

* [x] The default route `0.0.0.0/0` is installed via `198.51.100.1`.

```text
R1#show ip route static

Gateway of last resort is 198.51.100.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 198.51.100.1
```

### `ping 192.0.2.100`

**Expected Results**

* [x] The simulated external destination is reachable through the ISP.

```text
R1#ping 192.0.2.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.0.2.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

---

## OS1

### `show vlan brief`

**Expected Results**

* [x] VLAN `100` exists as `OUTSIDE_TRANSIT`, with `Gi0/0`, `Gi0/1`, and `Gi0/2` assigned.

```text
OS1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3
100  OUTSIDE_TRANSIT                  active    Gi0/0, Gi0/1, Gi0/2
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup
```

### `show spanning-tree vlan 100`

**Expected Results**

* [x] OS1 is the VLAN `100` root bridge and `Gi0/0`, `Gi0/1`, and `Gi0/2` are forwarding.

```text
OS1#show spanning-tree vlan 100 

VLAN0100
  Spanning tree enabled protocol ieee
  Root ID    Priority    32868
             Address     5254.0006.01bb
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32868  (priority 32768 sys-id-ext 100)
             Address     5254.0006.01bb
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p
```

---

## FW1

### `show interface ip brief`

**Expected Results**

* [x] `Gi0/0` uses `203.0.113.2`, `Gi0/1` uses `10.255.255.1`, and `Gi0/2` uses `10.255.0.4`; all three are `up/up`.

```text
FW-HA/pri/act# show interface ip brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         203.0.113.2     YES CONFIG up                    up  
GigabitEthernet0/1         10.255.255.1    YES unset  up                    up  
GigabitEthernet0/2         10.255.0.4      YES CONFIG up                    up  
GigabitEthernet0/3         unassigned      YES unset  administratively down down
GigabitEthernet0/4         unassigned      YES unset  administratively down down
Internal-Data0/0           169.254.1.1     YES unset  up                    up  
Management0/0              unassigned      YES unset  administratively down down
```

### `show route`

**Expected Results**

* [x] `203.0.113.0/29` is directly connected through `outside`.
* [x] `10.255.0.0/29` is directly connected through `inside`.
* [x] The default route `0.0.0.0/0` uses `203.0.113.1` through `outside`.
* [x] The internal route `10.10.10.0/24` uses `10.255.0.1` through `inside`.

```text
FW-HA/pri/act# show route

Gateway of last resort is 203.0.113.1 to network 0.0.0.0

S*       0.0.0.0 0.0.0.0 [1/0] via 203.0.113.1, outside
S        10.10.10.0 255.255.255.0 [1/0] via 10.255.0.1, inside
C        10.255.0.0 255.255.255.248 is directly connected, inside
L        10.255.0.4 255.255.255.255 is directly connected, inside
C        10.255.255.0 255.255.255.252 is directly connected, FAILOVER
L        10.255.255.1 255.255.255.255 is directly connected, FAILOVER
C        203.0.113.0 255.255.255.248 is directly connected, outside
L        203.0.113.2 255.255.255.255 is directly connected, outside
```

### `show nat detail`

**Expected Results**

* [x] Dynamic PAT translates `10.10.10.0/24` from `inside` to the `outside` interface.

```text
FW-HA/pri/act# show nat detail

Auto NAT Policies (Section 2)
1 (inside) to (outside) source dynamic INTERNAL_TEST interface 
    translate_hits = 0, untranslate_hits = 0
    Source - Origin: 10.10.10.0/24, Translated: 203.0.113.2/29
```

### `show running-config policy-map`

**Expected Results**

* [x] `inspect icmp` is configured under `global_policy`.

```text
FW-HA/pri/act# show running-config policy-map
!
policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map global_policy
 class inspection_default
  inspect ip-options 
  inspect netbios 
  inspect rtsp 
  inspect sunrpc 
  inspect tftp 
  inspect dns preset_dns_map 
  inspect ftp 
  inspect h323 h225 
  inspect h323 ras 
  inspect rsh 
  inspect esmtp 
  inspect sqlnet 
  inspect sip  
  inspect skinny 
  inspect icmp 
policy-map type inspect dns migrated_dns_map_2
 parameters   
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map type inspect dns migrated_dns_map_1
 parameters   
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
!
```

### `show running-config service-policy`

**Expected Results**

* [x] `global_policy` is applied globally.

```text
FW-HA/pri/act# show running-config service-policy
service-policy global_policy global
```

### `show failover`

**Expected Results**

* [x] Failover is enabled with FW1 `Primary - Active` and FW2 `Secondary - Standby Ready`.
* [x] The `FAILOVER` link is up.
* [x] The `inside` and `outside` interfaces report `Normal (Monitored)` on both units.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 16:51:23 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 11830 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 0 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

### `ping 203.0.113.1`

**Expected Results**

* [x] R1 is reachable through the `outside` interface.

```text
FW-HA/pri/act# ping 203.0.113.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 203.0.113.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/10 ms
```

### `ping 10.10.10.10`

**Expected Results**

* [x] A1 is reachable through the `inside` path and distribution layer.

```text
FW-HA/pri/act# ping 10.10.10.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/10 ms
```

---

## FW2

### `show interface ip brief`

**Expected Results**

* [x] `Gi0/0` uses `203.0.113.3`, `Gi0/1` uses `10.255.255.2`, and `Gi0/2` uses `10.255.0.5`; all three are `up/up`.

```text
FW-HA/sec/stby# show interface ip brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         203.0.113.3     YES CONFIG up                    up  
GigabitEthernet0/1         10.255.255.2    YES unset  up                    up  
GigabitEthernet0/2         10.255.0.5      YES CONFIG up                    up  
GigabitEthernet0/3         unassigned      YES unset  administratively down down
GigabitEthernet0/4         unassigned      YES unset  administratively down down
Internal-Data0/0           169.254.1.1     YES unset  up                    up  
Management0/0              unassigned      YES unset  administratively down down
```

### `show route`

**Expected Results**

* [x] `203.0.113.0/29` is directly connected through `outside`.
* [x] `10.255.0.0/29` is directly connected through `inside`.
* [x] The default route `0.0.0.0/0` uses `203.0.113.1` through `outside`.
* [x] The internal route `10.10.10.0/24` uses `10.255.0.1` through `inside`.

```text
FW-HA/sec/stby# show route

Gateway of last resort is 203.0.113.1 to network 0.0.0.0

S*       0.0.0.0 0.0.0.0 [1/0] via 203.0.113.1, outside
S        10.10.10.0 255.255.255.0 [1/0] via 10.255.0.1, inside
C        10.255.0.0 255.255.255.248 is directly connected, inside
L        10.255.0.5 255.255.255.255 is directly connected, inside
C        10.255.255.0 255.255.255.252 is directly connected, FAILOVER
L        10.255.255.2 255.255.255.255 is directly connected, FAILOVER
C        203.0.113.0 255.255.255.248 is directly connected, outside
L        203.0.113.3 255.255.255.255 is directly connected, outside
```

### `show nat detail`

**Expected Results**

* [x] The synchronized dynamic PAT rule translates `10.10.10.0/24` from `inside` to the `outside` interface.

```text
FW-HA/sec/stby# show nat detail

Auto NAT Policies (Section 2)
1 (inside) to (outside) source dynamic INTERNAL_TEST interface 
    translate_hits = 0, untranslate_hits = 0
    Source - Origin: 10.10.10.0/24, Translated: 203.0.113.3/29
```

### `show running-config policy-map`

**Expected Results**

* [x] The synchronized `global_policy` includes `inspect icmp`.

```text
FW-HA/sec/stby# show running-config policy-map
!
policy-map type inspect dns preset_dns_map
 parameters
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map global_policy
 class inspection_default
  inspect ip-options 
  inspect netbios 
  inspect rtsp 
  inspect sunrpc 
  inspect tftp 
  inspect dns preset_dns_map 
  inspect ftp 
  inspect h323 h225 
  inspect h323 ras 
  inspect rsh 
  inspect esmtp 
  inspect sqlnet 
  inspect sip  
  inspect skinny 
  inspect icmp 
policy-map type inspect dns migrated_dns_map_2
 parameters   
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
policy-map type inspect dns migrated_dns_map_1
 parameters   
  message-length maximum client auto
  message-length maximum 512
  no tcp-inspection
!
```

### `show running-config service-policy`

**Expected Results**

* [x] The synchronized `global_policy` is applied globally.

```text
FW-HA/sec/stby# show running-config service-policy
service-policy global_policy global
```

### `show failover`

**Expected Results**

* [x] Failover is enabled with FW2 `Secondary - Standby Ready` and FW1 `Primary - Active`.
* [x] The `FAILOVER` link is up.
* [x] The `inside` and `outside` interfaces report `Normal (Monitored)` on both units.

```text
FW-HA/sec/stby# show failover
Failover On 
Failover unit Secondary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ARTESW8ALK, Mate 9ABP74674HW
Last Failover at: 16:51:09 UTC Aug 8 2026
        This host: Secondary - Standby Ready 
                Active time: 0 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Primary - Active 
                Active time: 12121 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### `ping 203.0.113.1`

**Expected Results**

* [x] R1 is reachable through FW2's Standby-role `outside` interface.

```text
FW-HA/sec/stby# ping 203.0.113.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 203.0.113.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/4/10 ms
```

### `ping 10.10.10.10`

**Expected Results**

* [x] A1 is reachable through FW2's Standby-role `inside` path and distribution layer.

```text
FW-HA/sec/stby# ping 10.10.10.10
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/6/10 ms
```

---

## D1

### `show vlan brief`

**Expected Results**

* [x] VLANs `10` and `99` are active, and `Gi0/0` is assigned to VLAN `99`.

```text
D1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3, Gi2/0, Gi2/1, Gi2/2
                                                Gi2/3, Gi3/0, Gi3/1, Gi3/2
                                                Gi3/3
10   INTERNAL_TEST                    active    
99   FIREWALL_TRANSIT                 active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup
```

### `show interfaces trunk`

**Expected Results**

* [x] `Gi0/1` trunks VLANs `10,99`, and `Gi0/2` trunks VLAN `10`.

```text
D1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1
Gi0/2       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       10,99
Gi0/2       10

Port        Vlans allowed and active in management domain
Gi0/1       10,99
Gi0/2       10

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       10,99
Gi0/2       10
```

### `show ip interface brief | include Vlan`

**Expected Results**

* [x] `Vlan10` is `10.10.10.2` and `Vlan99` is `10.255.0.2`; both are `up/up`.

```text
D1#show ip interface brief | include Vlan
Vlan10                 10.10.10.2      YES NVRAM  up                    up      
Vlan99                 10.255.0.2      YES NVRAM  up                    up
```

### `show standby brief`

**Expected Results**

* [x] D1 is `Active` for HSRP groups `10` and `99` with priority `110`; D2 is the Standby peer.

```text
D1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   110 P Active  local           10.10.10.3      10.10.10.1
Vl99        99   110 P Active  local           10.255.0.3      10.255.0.1
```

### `show spanning-tree vlan 10`

**Expected Results**

* [x] D1 is the VLAN `10` root bridge and its participating ports are forwarding.

```text
D1#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     5254.002f.9faa
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    10     (priority 0 sys-id-ext 10)
             Address     5254.002f.9faa
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Desg FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p
```

### `show ip route static`

**Expected Results**

* [x] The default route is installed via the active firewall address `10.255.0.4`.

```text
D1#show ip route static

Gateway of last resort is 10.255.0.4 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.255.0.4
```

---

## D2

### `show vlan brief`

**Expected Results**

* [x] VLANs `10` and `99` are active, and `Gi0/0` is assigned to VLAN `99`.

```text
D2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/3, Gi1/0, Gi1/1, Gi1/2
                                                Gi1/3, Gi2/0, Gi2/1, Gi2/2
                                                Gi2/3, Gi3/0, Gi3/1, Gi3/2
                                                Gi3/3
10   INTERNAL_TEST                    active    
99   FIREWALL_TRANSIT                 active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### `show interfaces trunk`

**Expected Results**

* [x] `Gi0/1` trunks VLANs `10,99`, and `Gi0/2` trunks VLAN `10`.

```text
D2#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/1       on               802.1q         trunking      1
Gi0/2       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/1       10,99
Gi0/2       10

Port        Vlans allowed and active in management domain
Gi0/1       10,99
Gi0/2       10

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/1       10,99
Gi0/2       10
```

### `show ip interface brief | include Vlan`

**Expected Results**

* [x] `Vlan10` is `10.10.10.3` and `Vlan99` is `10.255.0.3`; both are `up/up`.

```text
D2#show ip interface brief | i Vlan
Vlan10                 10.10.10.3      YES NVRAM  up                    up      
Vlan99                 10.255.0.3      YES NVRAM  up                    up    
```

### `show standby brief`

**Expected Results**

* [x] D2 is `Standby` for HSRP groups `10` and `99` with priority `100`; D1 is the Active peer.

```text
D2#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   100 P Standby 10.10.10.2      local           10.10.10.1
Vl99        99   100 P Standby 10.255.0.2      local           10.255.0.1    
```

### `show spanning-tree vlan 10`

**Expected Results**

* [x] D2 is the intended secondary root bridge; `Gi0/1` is the forwarding root port and `Gi0/2` is forwarding as designated.

```text
D2#show spanning-tre vlan 10

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     5254.002f.9faa
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106   (priority 4096 sys-id-ext 10)
             Address     5254.00b9.b683
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    P2p 
Gi0/2               Desg FWD 4         128.3    P2p 
```

### `show ip route static`

**Expected Results**

* [x] The default route is installed via the active firewall address `10.255.0.4`.

```text
D2#show ip route static

Gateway of last resort is 10.255.0.4 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.255.0.4
```

---

## A1

### `show interfaces trunk`

**Expected Results**

* [x] `Gi0/0` and `Gi0/1` are trunking VLAN `10`.

```text
A1#show interfaces trunk

Port        Mode             Encapsulation  Status        Native vlan
Gi0/0       on               802.1q         trunking      1
Gi0/1       on               802.1q         trunking      1

Port        Vlans allowed on trunk
Gi0/0       10
Gi0/1       10

Port        Vlans allowed and active in management domain
Gi0/0       10
Gi0/1       10

Port        Vlans in spanning tree forwarding state and not pruned
Gi0/0       10
Gi0/1       none
```

### `show spanning-tree vlan 10`

**Expected Results**

* [x] The D1-facing uplink is forwarding and the D2-facing uplink is alternate/blocking.

```text
A1#show spanning-tree vlan 10 

VLAN0010
  Spanning tree enabled protocol ieee
  Root ID    Priority    10
             Address     5254.002f.9faa
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.0028.7118
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Altn BLK 4         128.2    P2p 
```

### `show ip interface brief | include Vlan10`

**Expected Results**

* [x] `Vlan10` is `10.10.10.10` and is `up/up`.

```text
A1#show ip interface brief | i Vlan10 
Vlan10                 10.10.10.10     YES NVRAM  up                    up  
```

### `show running-config | include ^ip default-gateway`

**Expected Results**

* [x] The default gateway is the HSRP VIP `10.10.10.1`.

```text
A1#show running-config | i ^ip default-gateway
ip default-gateway 10.10.10.1
```

### `ping 192.0.2.100`

**Expected Results**

* [x] End-to-end connectivity succeeds through HSRP, the active firewall, PAT, R1, and the ISP.

```text
A1#ping 192.0.2.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.0.2.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/208/1013 ms
```
