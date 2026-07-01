# P2P Multi-Area OSPF Verification Output

This file contains recorded verification output for the P2P Multi-Area OSPF lab.

The checks below confirm interface state, OSPF area participation, point-to-point OSPF neighbor adjacencies, ABR behavior, Type 3 LSA propagation, intra-area and inter-area route learning, and routed reachability across Areas 1, 0, and 2.

---

## R1

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.1`.
* [x] `GigabitEthernet0/1` is up/up with IP address `192.168.10.1`.
* [x] Unused interfaces are administratively down.

```text
R1>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.1        YES NVRAM  up                    up      
GigabitEthernet0/1         192.168.10.1    YES NVRAM  up                    up      
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down  
```

### `show ip ospf interface brief`

**Expected Results**

* [x] R1 LAN interface participates in Area 1.
* [x] R1-R2 point-to-point interface participates in Area 1.
* [x] Gi0/0 is in point-to-point state with one full neighbor.
* [x] Gi0/1 has no OSPF neighbors because it is a passive LAN interface.

```text
R1>show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/1        1     1               192.168.10.1/24    1     DR    0/0
Gi0/0        1     1               10.0.0.1/30        1     P2P   1/1
```

### `show ip route ospf`

**Expected Results**

* [x] R2 LAN is learned as an intra-area OSPF route.
* [x] Area 0 and Area 2 routes are learned as inter-area `O IA` routes.
* [x] Remote routes use R2 `10.0.0.2` as the next hop.

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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
O IA     10.0.0.4/30 [110/2] via 10.0.0.2, 00:12:54, GigabitEthernet0/0
O IA     10.0.0.8/30 [110/3] via 10.0.0.2, 00:12:54, GigabitEthernet0/0
O IA     10.0.0.12/30 [110/4] via 10.0.0.2, 00:12:42, GigabitEthernet0/0
O     192.168.20.0/24 [110/2] via 10.0.0.2, 00:12:54, GigabitEthernet0/0
O IA  192.168.30.0/24 [110/3] via 10.0.0.2, 00:12:54, GigabitEthernet0/0
O IA  192.168.40.0/24 [110/4] via 10.0.0.2, 00:12:42, GigabitEthernet0/0
O IA  192.168.50.0/24 [110/5] via 10.0.0.2, 00:12:38, GigabitEthernet0/0
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R2 appears as neighbor ID `2.2.2.2`.
* [x] Neighbor state is `FULL`.
* [x] The adjacency forms over `GigabitEthernet0/0`.

```text
R1>show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
2.2.2.2           0   FULL/  -        00:00:37    10.0.0.2        GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] Area 1 router LSAs are present for R1 and R2.
* [x] Type 3 summary LSAs are present for Area 0 and Area 2 networks.
* [x] Summary LSAs are advertised into Area 1 by ABR R2.

```text
R1>show ip ospf database

            OSPF Router with ID (1.1.1.1) (Process ID 1)

                Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         817         0x80000003 0x000F6B 3
2.2.2.2         2.2.2.2         829         0x80000003 0x00E782 3

                Summary Net Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.4        2.2.2.2         868         0x80000001 0x009A8E
10.0.0.8        2.2.2.2         863         0x80000001 0x007CA7
10.0.0.12       2.2.2.2         841         0x80000001 0x005EC0
192.168.30.0    2.2.2.2         863         0x80000001 0x006447
192.168.40.0    2.2.2.2         841         0x80000001 0x00FFA0
192.168.50.0    2.2.2.2         837         0x80000001 0x009BF9
```

### `show ip ospf border-routers`

**Expected Results**

* [x] R2 is known as an ABR from R1.
* [x] R2 is reachable through `10.0.0.2` over `GigabitEthernet0/0`.

```text
R1>show ip ospf border-routers

            OSPF Router with ID (1.1.1.1) (Process ID 1)


                Base Topology (MTID 0)

Internal Router Routing Table
Codes: i - Intra-area route, I - Inter-area route

i 2.2.2.2 [1] via 10.0.0.2, GigabitEthernet0/0, ABR, Area 1, SPF 2
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `1.1.1.1`.
* [x] R1 participates in one OSPF area.
* [x] R1 advertises the R1-R2 point-to-point link and R1 LAN into Area 1.
* [x] `GigabitEthernet0/1` is passive.

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
    10.0.0.0 0.0.0.3 area 1
    192.168.10.0 0.0.0.255 area 1
  Passive Interface(s):
    GigabitEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
    2.2.2.2              110      00:21:42
  Distance: (default is 110)
```

### `ping 192.168.50.1 source 192.168.10.1 repeat 5`

**Expected Results**

* [x] R1 can reach the R5 LAN gateway.
* [x] Traffic can route from Area 1 to Area 2.

```text
R1#ping 192.168.50.1 source 192.168.10.1 repeat 5
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.50.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.10.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

### `traceroute 192.168.50.1 source 192.168.10.1`

**Expected Results**

* [x] Path follows R1 to R2.
* [x] Path follows R2 to R3.
* [x] Path follows R3 to R4.
* [x] Path reaches R5.

```text
R1#traceroute 192.168.50.1 source 192.168.10.1
Type escape sequence to abort.
Tracing the route to 192.168.50.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.0.2 2 msec 2 msec 2 msec
  2 10.0.0.6 2 msec 2 msec 2 msec
  3 10.0.0.10 3 msec 3 msec 3 msec
  4 10.0.0.14 4 msec 4 msec * 
```

---

## R2

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.2`.
* [x] `GigabitEthernet0/1` is up/up with IP address `10.0.0.5`.
* [x] `GigabitEthernet0/2` is up/up with IP address `192.168.20.1`.

```text
R2>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.2        YES NVRAM  up                    up      
GigabitEthernet0/1         10.0.0.5        YES NVRAM  up                    up      
GigabitEthernet0/2         192.168.20.1    YES NVRAM  up                    up      
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down  
```

### `show ip ospf interface brief`

**Expected Results**

* [x] R2 participates in Area 1 on Gi0/0 and Gi0/2.
* [x] R2 participates in Area 0 on Gi0/1.
* [x] Gi0/0 and Gi0/1 are point-to-point OSPF links.
* [x] Gi0/2 has no OSPF neighbors because it is a passive LAN interface.

```text
R2>show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/1        1     0               10.0.0.5/30        1     P2P   1/1
Gi0/2        1     1               192.168.20.1/24    1     DR    0/0
Gi0/0        1     1               10.0.0.2/30        1     P2P   1/1
```

### `show ip route ospf`

**Expected Results**

* [x] Area 1 routes from R1 are learned as intra-area routes.
* [x] Area 0 routes from R3 are learned as intra-area routes.
* [x] Area 2 routes are learned as inter-area `O IA` routes through R3.

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

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O        10.0.0.8/30 [110/2] via 10.0.0.6, 00:27:38, GigabitEthernet0/1
O IA     10.0.0.12/30 [110/3] via 10.0.0.6, 00:27:15, GigabitEthernet0/1
O     192.168.10.0/24 [110/2] via 10.0.0.1, 00:27:25, GigabitEthernet0/0
O     192.168.30.0/24 [110/2] via 10.0.0.6, 00:27:38, GigabitEthernet0/1
O IA  192.168.40.0/24 [110/3] via 10.0.0.6, 00:27:15, GigabitEthernet0/1
O IA  192.168.50.0/24 [110/4] via 10.0.0.6, 00:27:10, GigabitEthernet0/1
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R3 appears as neighbor ID `3.3.3.3`.
* [x] R1 appears as neighbor ID `1.1.1.1`.
* [x] Both neighbors are in `FULL` state.

```text
R2>show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:39    10.0.0.6        GigabitEthernet0/1
1.1.1.1           0   FULL/  -        00:00:33    10.0.0.1        GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] R2 has Area 0 and Area 1 databases.
* [x] Type 1 router LSAs are present for routers in each local area.
* [x] Type 3 summary LSAs are present for networks learned from other areas.

```text
R2>show ip ospf database

            OSPF Router with ID (2.2.2.2) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         1711        0x80000002 0x00B634 2
3.3.3.3         3.3.3.3         1682        0x80000004 0x0016E3 5
4.4.4.4         4.4.4.4         1702        0x80000002 0x00BC15 2

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        2.2.2.2         1716        0x80000001 0x00C26A
10.0.0.12       4.4.4.4         1712        0x80000001 0x000E0B
192.168.10.0    2.2.2.2         1698        0x80000001 0x00417E
192.168.20.0    2.2.2.2         1716        0x80000001 0x00C8ED
192.168.40.0    4.4.4.4         1712        0x80000001 0x00AFEA
192.168.50.0    4.4.4.4         1685        0x80000001 0x004B44

                Router Link States (Area 1)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         1666        0x80000003 0x000F6B 3
2.2.2.2         2.2.2.2         1675        0x80000003 0x00E782 3
          
                Summary Net Link States (Area 1)
          
Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.4        2.2.2.2         1716        0x80000001 0x009A8E
10.0.0.8        2.2.2.2         1711        0x80000001 0x007CA7
10.0.0.12       2.2.2.2         1688        0x80000001 0x005EC0
192.168.30.0    2.2.2.2         1711        0x80000001 0x006447
192.168.40.0    2.2.2.2         1688        0x80000001 0x00FFA0
192.168.50.0    2.2.2.2         1683        0x80000001 0x009BF9
```

### `show ip ospf border-routers`

**Expected Results**

* [x] R4 is known as an ABR from R2.
* [x] R4 is reachable through R3 over Area 0.

```text
R2>show ip ospf border-routers

            OSPF Router with ID (2.2.2.2) (Process ID 1)


                Base Topology (MTID 0)

Internal Router Routing Table
Codes: i - Intra-area route, I - Inter-area route

i 4.4.4.4 [2] via 10.0.0.6, GigabitEthernet0/1, ABR, Area 0, SPF 4
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `2.2.2.2`.
* [x] R2 is an ABR.
* [x] R2 participates in two OSPF areas.
* [x] R2 advertises networks into Area 1 and Area 0.
* [x] `GigabitEthernet0/2` is passive.

```text
R2>show ip protocols
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
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.0 0.0.0.3 area 1
    10.0.0.4 0.0.0.3 area 0
    192.168.20.0 0.0.0.255 area 1
  Passive Interface(s):
    GigabitEthernet0/2
  Routing Information Sources:
    Gateway         Distance      Last Update
    4.4.4.4              110      00:29:57
    1.1.1.1              110      00:30:12
    3.3.3.3              110      00:30:24
  Distance: (default is 110)
```

---

## R3

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.6`.
* [x] `GigabitEthernet0/1` is up/up with IP address `10.0.0.9`.
* [x] `GigabitEthernet0/2` is up/up with IP address `192.168.30.1`.

```text
R3>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.6        YES NVRAM  up                    up      
GigabitEthernet0/1         10.0.0.9        YES NVRAM  up                    up      
GigabitEthernet0/2         192.168.30.1    YES NVRAM  up                    up      
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down  
```

### `show ip ospf interface brief`

**Expected Results**

* [x] R3 participates only in Area 0.
* [x] Gi0/0 and Gi0/1 are point-to-point OSPF links.
* [x] Gi0/2 has no OSPF neighbors because it is a passive LAN interface.

```text
R3>show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/2        1     0               192.168.30.1/24    1     DR    0/0
Gi0/1        1     0               10.0.0.9/30        1     P2P   1/1
Gi0/0        1     0               10.0.0.6/30        1     P2P   1/1
```

### `show ip route ospf`

**Expected Results**

* [x] Area 1 routes are learned as inter-area `O IA` routes through R2.
* [x] Area 2 routes are learned as inter-area `O IA` routes through R4.

```text
R3>show ip route ospf
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

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.0.0/30 [110/2] via 10.0.0.5, 00:37:44, GigabitEthernet0/0
O IA     10.0.0.12/30 [110/2] via 10.0.0.10, 00:37:34, GigabitEthernet0/1
O IA  192.168.10.0/24 [110/3] via 10.0.0.5, 00:37:34, GigabitEthernet0/0
O IA  192.168.20.0/24 [110/2] via 10.0.0.5, 00:37:44, GigabitEthernet0/0
O IA  192.168.40.0/24 [110/2] via 10.0.0.10, 00:37:34, GigabitEthernet0/1
O IA  192.168.50.0/24 [110/3] via 10.0.0.10, 00:37:22, GigabitEthernet0/1
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R4 appears as neighbor ID `4.4.4.4`.
* [x] R2 appears as neighbor ID `2.2.2.2`.
* [x] Both neighbors are in `FULL` state.

```text
R3>show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           0   FULL/  -        00:00:35    10.0.0.10       GigabitEthernet0/1
2.2.2.2           0   FULL/  -        00:00:31    10.0.0.5        GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] Area 0 router LSAs are present for R2, R3, and R4.
* [x] Type 3 summary LSAs are present for Area 1 and Area 2 networks.

```text
R3>show ip ospf database

            OSPF Router with ID (3.3.3.3) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         295         0x80000003 0x00B435 2
3.3.3.3         3.3.3.3         318         0x80000005 0x0014E4 5
4.4.4.4         4.4.4.4         304         0x80000003 0x00BA16 2

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        2.2.2.2         295         0x80000002 0x00C06B
10.0.0.12       4.4.4.4         304         0x80000002 0x000C0C
192.168.10.0    2.2.2.2         295         0x80000002 0x003F7F
192.168.20.0    2.2.2.2         295         0x80000002 0x00C6EE
192.168.40.0    4.4.4.4         304         0x80000002 0x00ADEB
192.168.50.0    4.4.4.4         304         0x80000002 0x004945
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `3.3.3.3`.
* [x] R3 participates in one OSPF area.
* [x] R3 advertises all configured networks into Area 0.
* [x] `GigabitEthernet0/2` is passive.

```text
R3>show ip protocols
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
  Router ID 3.3.3.3
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.4 0.0.0.3 area 0
    10.0.0.8 0.0.0.3 area 0
    192.168.30.0 0.0.0.255 area 0
  Passive Interface(s):
    GigabitEthernet0/2
  Routing Information Sources:
    Gateway         Distance      Last Update
    4.4.4.4              110      00:41:50
    2.2.2.2              110      00:42:03
  Distance: (default is 110)
```

---

## R4

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.10`.
* [x] `GigabitEthernet0/1` is up/up with IP address `10.0.0.13`.
* [x] `GigabitEthernet0/2` is up/up with IP address `192.168.40.1`.

```text
R4>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.10       YES NVRAM  up                    up      
GigabitEthernet0/1         10.0.0.13       YES NVRAM  up                    up      
GigabitEthernet0/2         192.168.40.1    YES NVRAM  up                    up      
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down   
```

### `show ip ospf interface brief`

**Expected Results**

* [x] R4 participates in Area 0 on Gi0/0.
* [x] R4 participates in Area 2 on Gi0/1 and Gi0/2.
* [x] Gi0/0 and Gi0/1 are point-to-point OSPF links.
* [x] Gi0/2 has no OSPF neighbors because it is a passive LAN interface.

```text
R4>show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/0        1     0               10.0.0.10/30       1     P2P   1/1
Gi0/2        1     2               192.168.40.1/24    1     DR    0/0
Gi0/1        1     2               10.0.0.13/30       1     P2P   1/1
```

### `show ip route ospf`

**Expected Results**

* [x] Area 0 routes from R3 are learned as intra-area routes.
* [x] Area 2 routes from R5 are learned as intra-area routes.
* [x] Area 1 routes are learned as inter-area `O IA` routes through R3.

```text
R4>show ip route ospf
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

      10.0.0.0/8 is variably subnetted, 6 subnets, 2 masks
O IA     10.0.0.0/30 [110/3] via 10.0.0.9, 00:45:57, GigabitEthernet0/0
O        10.0.0.4/30 [110/2] via 10.0.0.9, 00:45:57, GigabitEthernet0/0
O IA  192.168.10.0/24 [110/4] via 10.0.0.9, 00:45:57, GigabitEthernet0/0
O IA  192.168.20.0/24 [110/3] via 10.0.0.9, 00:45:57, GigabitEthernet0/0
O     192.168.30.0/24 [110/2] via 10.0.0.9, 00:45:57, GigabitEthernet0/0
O     192.168.50.0/24 [110/2] via 10.0.0.14, 00:45:45, GigabitEthernet0/1
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R3 appears as neighbor ID `3.3.3.3`.
* [x] R5 appears as neighbor ID `5.5.5.5`.
* [x] Both neighbors are in `FULL` state.

```text
R4>show ip ospf neighbor 

Neighbor ID     Pri   State           Dead Time   Address         Interface
3.3.3.3           0   FULL/  -        00:00:31    10.0.0.9        GigabitEthernet0/0
5.5.5.5           0   FULL/  -        00:00:34    10.0.0.14       GigabitEthernet0/1
```

### `show ip ospf database`

**Expected Results**

* [x] R4 has Area 0 and Area 2 databases.
* [x] Type 1 router LSAs are present for routers in each local area.
* [x] Type 3 summary LSAs are present for networks learned from other areas.

```text
R4>show ip ospf database

            OSPF Router with ID (4.4.4.4) (Process ID 1)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
2.2.2.2         2.2.2.2         800         0x80000003 0x00B435 2
3.3.3.3         3.3.3.3         824         0x80000005 0x0014E4 5
4.4.4.4         4.4.4.4         807         0x80000003 0x00BA16 2

                Summary Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        2.2.2.2         800         0x80000002 0x00C06B
10.0.0.12       4.4.4.4         807         0x80000002 0x000C0C
192.168.10.0    2.2.2.2         800         0x80000002 0x003F7F
192.168.20.0    2.2.2.2         800         0x80000002 0x00C6EE
192.168.40.0    4.4.4.4         807         0x80000002 0x00ADEB
192.168.50.0    4.4.4.4         807         0x80000002 0x004945

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
4.4.4.4         4.4.4.4         807         0x80000004 0x00041A 3
5.5.5.5         5.5.5.5         795         0x80000004 0x00D639 3
          
                Summary Net Link States (Area 2)
          
Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        4.4.4.4         807         0x80000002 0x009889
10.0.0.4        4.4.4.4         807         0x80000002 0x0066B8
10.0.0.8        4.4.4.4         807         0x80000002 0x0034E7
192.168.10.0    4.4.4.4         807         0x80000002 0x00179D
192.168.20.0    4.4.4.4         807         0x80000002 0x009E0D
192.168.30.0    4.4.4.4         807         0x80000002 0x00267C
```

### `show ip ospf border-routers`

**Expected Results**

* [x] R2 is known as an ABR from R4.
* [x] R2 is reachable through R3 over Area 0.

```text
R4>show ip ospf border-routers

            OSPF Router with ID (4.4.4.4) (Process ID 1)


                Base Topology (MTID 0)

Internal Router Routing Table
Codes: i - Intra-area route, I - Inter-area route

i 2.2.2.2 [2] via 10.0.0.9, GigabitEthernet0/0, ABR, Area 0, SPF 2
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `4.4.4.4`.
* [x] R4 is an ABR.
* [x] R4 participates in two OSPF areas.
* [x] R4 advertises networks into Area 0 and Area 2.
* [x] `GigabitEthernet0/2` is passive.

```text
R4>show ip protocols
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
  Router ID 4.4.4.4
  It is an area border router
  Number of areas in this router is 2. 2 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.8 0.0.0.3 area 0
    10.0.0.12 0.0.0.3 area 2
    192.168.40.0 0.0.0.255 area 2
  Passive Interface(s):
    GigabitEthernet0/2
  Routing Information Sources:
    Gateway         Distance      Last Update
    5.5.5.5              110      00:47:24
    2.2.2.2              110      00:47:36
    3.3.3.3              110      00:47:37
  Distance: (default is 110)
```

---

## R5

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up with IP address `10.0.0.14`.
* [x] `GigabitEthernet0/1` is up/up with IP address `192.168.50.1`.
* [x] Unused interfaces are administratively down.

```text
R5>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.0.0.14       YES NVRAM  up                    up      
GigabitEthernet0/1         192.168.50.1    YES NVRAM  up                    up      
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down 
```

### `show ip ospf interface brief`

**Expected Results**

* [x] R5 participates only in Area 2.
* [x] Gi0/0 is a point-to-point OSPF link.
* [x] Gi0/1 has no OSPF neighbors because it is a passive LAN interface.

```text
R5>show ip ospf interface brief
Interface    PID   Area            IP Address/Mask    Cost  State Nbrs F/C
Gi0/1        1     2               192.168.50.1/24    1     DR    0/0
Gi0/0        1     2               10.0.0.14/30       1     P2P   1/1
```

### `show ip route ospf`

**Expected Results**

* [x] R4 LAN is learned as an intra-area route.
* [x] Area 0 and Area 1 routes are learned as inter-area `O IA` routes through R4.

```text
R5>show ip route ospf
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

      10.0.0.0/8 is variably subnetted, 5 subnets, 2 masks
O IA     10.0.0.0/30 [110/4] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O IA     10.0.0.4/30 [110/3] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O IA     10.0.0.8/30 [110/2] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O IA  192.168.10.0/24 [110/5] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O IA  192.168.20.0/24 [110/4] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O IA  192.168.30.0/24 [110/3] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
O     192.168.40.0/24 [110/2] via 10.0.0.13, 00:49:10, GigabitEthernet0/0
```

### `show ip ospf neighbor`

**Expected Results**

* [x] R4 appears as neighbor ID `4.4.4.4`.
* [x] Neighbor state is `FULL`.
* [x] The adjacency forms over `GigabitEthernet0/0`.

```text
R5>show ip ospf neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
4.4.4.4           0   FULL/  -        00:00:38    10.0.0.13       GigabitEthernet0/0
```

### `show ip ospf database`

**Expected Results**

* [x] Area 2 router LSAs are present for R4 and R5.
* [x] Type 3 summary LSAs are present for Area 0 and Area 1 networks.
* [x] Summary LSAs are advertised into Area 2 by ABR R4.

```text
R5>show ip ospf database

            OSPF Router with ID (5.5.5.5) (Process ID 1)

                Router Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum Link count
4.4.4.4         4.4.4.4         1051        0x80000004 0x00041A 3
5.5.5.5         5.5.5.5         1036        0x80000004 0x00D639 3

                Summary Net Link States (Area 2)

Link ID         ADV Router      Age         Seq#       Checksum
10.0.0.0        4.4.4.4         1051        0x80000002 0x009889
10.0.0.4        4.4.4.4         1051        0x80000002 0x0066B8
10.0.0.8        4.4.4.4         1051        0x80000002 0x0034E7
192.168.10.0    4.4.4.4         1051        0x80000002 0x00179D
192.168.20.0    4.4.4.4         1051        0x80000002 0x009E0D
192.168.30.0    4.4.4.4         1051        0x80000002 0x00267C
```

### `show ip ospf border-routers`

**Expected Results**

* [x] R4 is known as an ABR from R5.
* [x] R4 is reachable through `10.0.0.13` over `GigabitEthernet0/0`.

```text
R5>show ip ospf border-routers

            OSPF Router with ID (5.5.5.5) (Process ID 1)


                Base Topology (MTID 0)

Internal Router Routing Table
Codes: i - Intra-area route, I - Inter-area route

i 4.4.4.4 [1] via 10.0.0.13, GigabitEthernet0/0, ABR, Area 2, SPF 2
```

### `show ip protocols`

**Expected Results**

* [x] OSPF process `1` is running.
* [x] Router ID is `5.5.5.5`.
* [x] R5 participates in one OSPF area.
* [x] R5 advertises the R4-R5 point-to-point link and R5 LAN into Area 2.
* [x] `GigabitEthernet0/1` is passive.

```text
R5>show ip protocols
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
  Router ID 5.5.5.5
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
  Maximum path: 4
  Routing for Networks:
    10.0.0.12 0.0.0.3 area 2
    192.168.50.0 0.0.0.255 area 2
  Passive Interface(s):
    GigabitEthernet0/1
  Routing Information Sources:
    Gateway         Distance      Last Update
    4.4.4.4              110      00:52:24
  Distance: (default is 110)
```
