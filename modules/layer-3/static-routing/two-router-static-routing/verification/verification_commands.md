# Static Two Router Verification Output

This file contains recorded verification output for the Static Two Router lab.

The checks below confirm interface state, connected routes, default static routes, absence of dynamic routing protocols, ARP resolution, CDP neighbor discovery, and routed reachability between LAN gateways.

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

### `show ip route`

**Expected Results**

* [x] Default static route `0.0.0.0/0` is present via `10.0.0.2`.
* [x] Point-to-point network `10.0.0.0/30` is directly connected.
* [x] Local point-to-point interface route `10.0.0.1/32` is present.
* [x] R1 LAN `192.168.0.0/24` is directly connected.
* [x] Local LAN interface route `192.168.0.1/32` is present.

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

Gateway of last resort is 10.0.0.2 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.2
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0
L        10.0.0.1/32 is directly connected, GigabitEthernet0/0
      192.168.0.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.0.0/24 is directly connected, GigabitEthernet0/1
L        192.168.0.1/32 is directly connected, GigabitEthernet0/1
```

### `show ip protocols`

**Expected Results**

* [x] No dynamic routing protocol is configured.
* [x] Output does not show OSPF, EIGRP, RIP, or BGP routing processes.

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
```

### `show arp`

**Expected Results**

* [x] R1 local point-to-point address `10.0.0.1` is present.
* [x] R2 next-hop address `10.0.0.2` is present.
* [x] R1 LAN gateway address `192.168.0.1` is present.

```text
R1>show arp
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.0.0.1                -   5254.00f8.fd6b  ARPA   GigabitEthernet0/0
Internet  10.0.0.2               14   5254.007c.9f5c  ARPA   GigabitEthernet0/0
Internet  192.168.0.1             -   5254.003d.5126  ARPA   GigabitEthernet0/1
```

### `show cdp neighbors`

**Expected Results**

* [x] R2 is discovered through CDP.
* [x] Local interface is `GigabitEthernet0/0`.
* [x] Remote port ID is `GigabitEthernet0/0`.

```text
R1>show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone, 
                  D - Remote, C - CVTA, M - Two-port Mac Relay 

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R2               Gig 0/0           133              R B             Gig 0/0

Total cdp entries displayed : 1
```

### `ping 192.168.10.1 source 192.168.0.1`

**Expected Results**

* [x] R1 can reach the R2 LAN gateway.
* [x] Ping succeeds using R1 LAN gateway as the source address.
* [x] Routed reachability across the static default route is successful.

```text
R1#ping 192.168.10.1 source 192.168.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.10.1, timeout is 2 seconds:
Packet sent with a source address of 192.168.0.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

### `traceroute 192.168.10.1 source 192.168.0.1`

**Expected Results**

* [x] Traceroute from R1 LAN gateway to R2 LAN gateway succeeds.
* [x] Next hop is R2 point-to-point address `10.0.0.2`.

```text
R1#traceroute 192.168.10.1 source 192.168.0.1
Type escape sequence to abort.
Tracing the route to 192.168.10.1
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.0.2 2 msec 2 msec * 
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

### `show ip route`

**Expected Results**

* [x] Default static route `0.0.0.0/0` is present via `10.0.0.1`.
* [x] Point-to-point network `10.0.0.0/30` is directly connected.
* [x] Local point-to-point interface route `10.0.0.2/32` is present.
* [x] R2 LAN `192.168.10.0/24` is directly connected.
* [x] Local LAN interface route `192.168.10.1/32` is present.

```text
R2#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area 
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 10.0.0.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 10.0.0.1
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0
L        10.0.0.2/32 is directly connected, GigabitEthernet0/0
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/1
L        192.168.10.1/32 is directly connected, GigabitEthernet0/1
```

### `show ip protocols`

**Expected Results**

* [x] No dynamic routing protocol is configured.
* [x] Output does not show OSPF, EIGRP, RIP, or BGP routing processes.

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
```

### `show arp`

**Expected Results**

* [x] R1 next-hop address `10.0.0.1` is present.
* [x] R2 local point-to-point address `10.0.0.2` is present.
* [x] R2 LAN gateway address `192.168.10.1` is present.

```text
R2>show arp 
Protocol  Address          Age (min)  Hardware Addr   Type   Interface
Internet  10.0.0.1               22   5254.00f8.fd6b  ARPA   GigabitEthernet0/0
Internet  10.0.0.2                -   5254.007c.9f5c  ARPA   GigabitEthernet0/0
Internet  192.168.10.1            -   5254.0031.9284  ARPA   GigabitEthernet0/1
```

### `show cdp neighbors`

**Expected Results**

* [x] R1 is discovered through CDP.
* [x] Local interface is `GigabitEthernet0/0`.
* [x] Remote port ID is `GigabitEthernet0/0`.

```text
R2>show cdp neighbors
Capability Codes: R - Router, T - Trans Bridge, B - Source Route Bridge
                  S - Switch, H - Host, I - IGMP, r - Repeater, P - Phone, 
                  D - Remote, C - CVTA, M - Two-port Mac Relay 

Device ID        Local Intrfce     Holdtme    Capability  Platform  Port ID
R1               Gig 0/0           161              R B             Gig 0/0

Total cdp entries displayed : 1
```
