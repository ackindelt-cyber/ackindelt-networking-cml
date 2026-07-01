# P2P Single-Area OSPF Verification Output

This file contains recorded verification output for the P2P Single-Area OSPF lab.

The checks below confirm interface state, OSPF neighbor adjacency, point-to-point OSPF network type, OSPF-learned routes, OSPF database contents, OSPF process configuration, and end-to-end connectivity between LAN segments.

---

## R1

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.1`.
* [x] `GigabitEthernet0/1` is up/up with IP address `192.168.0.1`.
* [x] Unused interfaces are administratively down.

```text
R1>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.1        YES NVRAM  up                    up      
GigabitEthernet0/1         192.168.0.1     YES NVRAM  up                    up      
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R2 appears as neighbor ID `2.2.2.2`.
* [x] Neighbor state is `FULL`.
* [x] Neighbor is reachable through `GigabitEthernet0/0`.

```text
R1>show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:33    10.0.0.2        GigabitEthernet0/0
```

### `show ip ospf interface gigabitEthernet0/0`

**Expected Results**

* [x] Interface is up/up.
* [x] Interface IP is `10.0.0.1/30`.
* [x] OSPF Area is `0`.
* [x] OSPF process ID is `1`.
* [x] Router ID is `1.1.1.1`.
* [x] Network type is `POINT_TO_POINT`.
* [x] Hello timer is `10`.
* [x] Dead timer is `40`.
* [x] Neighbor count is `1`.

```text
R1>show ip ospf interface gigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 10.0.0.1/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 1.1.1.1, Network Type POINT_TO_POINT, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:02
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 1 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 2.2.2.2
  Suppress hello for 0 neighbor(s)
```

### `show ip route ospf`

**Expected Results**

* [x] R2 LAN route `192.168.10.0/24` is learned through OSPF.
* [x] Next hop is `10.0.0.2`.
* [x] Route exits through `GigabitEthernet0/0`.

```text
R1>show ip route ospf
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

O     192.168.10.0/24 [110/2] via 10.0.0.2, 00:16:44, GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] R1 router LSA is present in Area 0.
* [x] R2 router LSA is present in Area 0.
* [x] Both routers appear with router-LSA link count `3`.

```text
R1>show ip ospf database

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1170        0x80000003 0x00B4CF 3
2.2.2.2         2.2.2.2         1165        0x80000003 0x008AEA 3
```

### `show run | section ospf`

**Expected Results**

* [x] OSPF network type is point-to-point on the transit interface.
* [x] OSPF process ID is `1`.
* [x] Router ID is `1.1.1.1`.
* [x] `GigabitEthernet0/1` is passive.
* [x] Transit network is advertised in Area 0.
* [x] R1 LAN network is advertised in Area 0.

```text
R1#show run | section ospf
 ip ospf network point-to-point
router ospf 1
 router-id 1.1.1.1
 passive-interface GigabitEthernet0/1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.0.0 0.0.0.255 area 0
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `1.1.1.1`.
* [x] Router has one OSPF area.
* [x] Transit network is advertised in Area 0.
* [x] R1 LAN network is advertised in Area 0.
* [x] `GigabitEthernet0/1` is passive.
* [x] Routing source includes R2 router ID `2.2.2.2`.
* [x] Administrative distance is `110`.

```text
R1#show ip protocols
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 1.1.1.1
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.0 0.0.0.3 area 0
    192.168.0.0 0.0.0.255 area 0
  Passive Interface(s):
    GigabitEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      00:21:37
  Distance: (default is 110)
```

### `ping 192.168.10.1 source 192.168.0.1 repeat 5`

**Expected Results**

* [x] R1 can reach R2 LAN gateway.
* [x] Ping succeeds using R1 LAN interface as the source.

```text
R1#ping 192.168.10.1 source 192.168.0.1 repeat 5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.0.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

### `traceroute 192.168.10.1 source 192.168.0.1`

**Expected Results**

* [x] Traceroute from R1 LAN source to R2 LAN gateway succeeds.
* [x] Next hop is R2 transit IP `10.0.0.2`.

```text
R1#traceroute 192.168.10.1 source 192.168.0.1
Type escape sequence to abort.
Tracing the route to 192.168.10.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.0.2 3 msec 3 msec *
```

---

## R2

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.2`.
* [x] `GigabitEthernet0/1` is up/up with IP address `192.168.10.1`.
* [x] Unused interfaces are administratively down.

```text
R2>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.2        YES NVRAM  up                    up      
GigabitEthernet0/1         192.168.10.1    YES NVRAM  up                    up      
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down   
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R1 appears as neighbor ID `1.1.1.1`.
* [x] Neighbor state is `FULL`.
* [x] Neighbor is reachable through `GigabitEthernet0/0`.

```text
R2>show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           0   FULL/  -        00:00:34    10.0.0.1        GigabitEthernet0/0
```

### `show ip ospf interface gigabitEthernet0/0`

**Expected Results**

* [x] Interface is up/up.
* [x] Interface IP is `10.0.0.2/30`.
* [x] OSPF Area is `0`.
* [x] OSPF process ID is `1`.
* [x] Router ID is `2.2.2.2`.
* [x] Network type is `POINT_TO_POINT`.
* [x] Hello timer is `10`.
* [x] Dead timer is `40`.
* [x] Neighbor count is `1`.

```text
R2>show ip ospf interface gigabitethernet0/0
GigabitEthernet0/0 is up, line protocol is up 
  Internet Address 10.0.0.2/30, Area 0, Attached via Network Statement
  Process ID 1, Router ID 2.2.2.2, Network Type POINT_TO_POINT, Cost: 1
  Topology-MTID    Cost    Disabled    Shutdown      Topology Name
        0           1         no          no            Base
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    oob-resync timeout 40
    Hello due in 00:00:00
  Supports Link-local Signaling (LLS)
  Cisco NSF helper support enabled
  IETF NSF helper support enabled
  Index 1/1/1, flood queue length 0
  Next 0x0(0)/0x0(0)/0x0(0)
  Last flood scan length is 1, maximum is 1
  Last flood scan time is 0 msec, maximum is 0 msec
  Neighbor Count is 1, Adjacent neighbor count is 1 
    Adjacent with neighbor 1.1.1.1
  Suppress hello for 0 neighbor(s)
```

### `show ip route ospf`

**Expected Results**

* [x] R1 LAN route `192.168.0.0/24` is learned through OSPF.
* [x] Next hop is `10.0.0.1`.
* [x] Route exits through `GigabitEthernet0/0`.

```text
R2>show ip route ospf
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

O     192.168.0.0/24 [110/2] via 10.0.0.1, 00:32:39, GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] R1 router LSA is present in Area 0.
* [x] R2 router LSA is present in Area 0.
* [x] Both routers appear with router-LSA link count `3`.

```text
R2>show ip ospf database

            OSPF Router with ID (2.2.2.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1954        0x80000003 0x00B4CF 3
2.2.2.2         2.2.2.2         1948        0x80000003 0x008AEA 3
```

### `show run | section ospf`

**Expected Results**

* [x] OSPF network type is point-to-point on the transit interface.
* [x] OSPF process ID is `1`.
* [x] Router ID is `2.2.2.2`.
* [x] `GigabitEthernet0/1` is passive.
* [x] Transit network is advertised in Area 0.
* [x] R2 LAN network is advertised in Area 0.

```text
R2#show run | section ospf
 ip ospf network point-to-point
router ospf 1
 router-id 2.2.2.2
 passive-interface GigabitEthernet0/1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `2.2.2.2`.
* [x] Router has one OSPF area.
* [x] Transit network is advertised in Area 0.
* [x] R2 LAN network is advertised in Area 0.
* [x] `GigabitEthernet0/1` is passive.
* [x] Routing source includes R1 router ID `1.1.1.1`.
* [x] Administrative distance is `110`.

```text
R2#show ip protocols 
*** IP Routing is NSF aware ***

Routing Protocol is "application"
  Sending updates every 0 seconds
  Invalid after 0 seconds, hold down 0, flushed after 0
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Maximum path: 32
  Routing for Networks:
  Routing Information Sources:
    Gateway         Distance      Last Update
  Distance: (default is 4)

Routing Protocol is "ospf 1"
  Outgoing update filter list for all interfaces is not set
  Incoming update filter list for all interfaces is not set
  Router ID 2.2.2.2
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.0 0.0.0.3 area 0
    192.168.10.0 0.0.0.255 area 0
  Passive Interface(s):
    GigabitEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
    1.1.1.1              110      00:36:18
  Distance: (default is 110)
```

### `ping 192.168.0.1 source 192.168.10.1 repeat 5`

**Expected Results**

* [x] R2 can reach R1 LAN gateway.
* [x] Ping succeeds using R2 LAN interface as the source.

```text
R2#ping 192.168.0.1 source 192.168.10.1 repeat 5 
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.0.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.10.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

### `traceroute 192.168.0.1 source 192.168.10.1`

**Expected Results**

* [x] Traceroute from R2 LAN source to R1 LAN gateway succeeds.
* [x] Next hop is R1 transit IP `10.0.0.1`.

```text
R2# traceroute 192.168.0.1 source 192.168.10.1 
Type escape sequence to abort.
Tracing the route to 192.168.0.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.0.1 2 msec 2 msec *
```

---

## C1

### `ping -w 5 192.168.10.2`

**Expected Results**

* [x] C1 receives ICMP replies from C2.
* [x] End-to-end connectivity succeeds across the OSPF-learned path.

```text
C1:~# ping -w 5 192.168.10.2
PING 192.168.10.2 (192.168.10.2): 56 data bytes
64 bytes from 192.168.10.2: seq=0 ttl=62 time=1.219 ms
64 bytes from 192.168.10.2: seq=1 ttl=62 time=1.312 ms
64 bytes from 192.168.10.2: seq=2 ttl=62 time=1.249 ms
64 bytes from 192.168.10.2: seq=3 ttl=62 time=1.400 ms
64 bytes from 192.168.10.2: seq=4 ttl=62 time=1.247 ms

--- 192.168.10.2 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.219/1.285/1.400 ms
```
