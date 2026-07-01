# Standard ACL Verification Output

This file contains recorded verification output for the Standard ACL lab.

The checks below confirm that standard ACL `10` exists, the ACL permits traffic sourced from `192.168.10.0/24`, match counters increment after inside client traffic, the ACL is applied outbound on R1’s WAN interface, R1 has a default route toward R2, and inside clients can reach the outside host.

---

## R1

### `show ip access-lists 10`

**Expected Results**

* [x] Standard ACL `10` is present.
* [x] Permit statement for `192.168.10.0/24` is present.
* [x] Match counter increments after traffic is generated from C1 and C3.
* [x] No unexpected explicit ACL entries are present.

```text
R1>show ip access-lists 10
Standard IP access list 10
    10 permit 192.168.10.0, wildcard bits 0.0.0.255 (23 matches)
```

### `show ip interface gigabitEthernet0/1`

**Expected Results**

* [x] `GigabitEthernet0/1` is up/up.
* [x] Interface IP address is `198.51.100.2/30`.
* [x] Outgoing access list is `10`.
* [x] Inbound access list is not set.

```text
R1>show ip interface gigabitEthernet0/1
GigabitEthernet0/1 is up, line protocol is up
  Internet address is 198.51.100.2/30
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is 10
  Inbound  access list is not set
  Proxy ARP is enabled
  Local Proxy ARP is disabled
  Security level is default
  Split horizon is enabled
  ICMP redirects are always sent
  ICMP unreachables are always sent
  ICMP mask replies are never sent
  IP fast switching is enabled
  IP fast switching on the same interface is disabled
  IP Flow switching is disabled
  IP CEF switching is enabled
  IP CEF switching turbo vector
  IP multicast fast switching is enabled
  IP multicast distributed fast switching is disabled
  IP route-cache flags are Fast, CEF
  Router Discovery is disabled
  IP output packet accounting is disabled
  IP access violation accounting is disabled
  TCP/IP header compression is disabled
  RTP/IP header compression is disabled
  Policy routing is disabled
  Network address translation is disabled
  BGP Policy Mapping is disabled
  Input features: MCI Check
  Output features: Access List
  IPv4 WCCP Redirect outbound is disabled
  IPv4 WCCP Redirect inbound is disabled
  IPv4 WCCP Redirect exclude is disabled
```

### `show ip route`

**Expected Results**

* [x] Default route points toward R2 at `198.51.100.1`.
* [x] Inside network `192.168.10.0/24` is directly connected.
* [x] WAN network `198.51.100.0/30` is directly connected.
* [x] No unexpected dynamic routing entries are present.

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

Gateway of last resort is 198.51.100.1 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 198.51.100.1
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/0
L        192.168.10.1/32 is directly connected, GigabitEthernet0/0
      198.51.100.0/24 is variably subnetted, 2 subnets, 2 masks
C        198.51.100.0/30 is directly connected, GigabitEthernet0/1
L        198.51.100.2/32 is directly connected, GigabitEthernet0/1
```

---

## C1

### `ping -w 5 203.0.113.100`

**Expected Results**

* [x] C1 receives ICMP replies from `203.0.113.100`.
* [x] Traffic sourced from C1 matches ACL 10.
* [x] End-to-end reachability from C1 to the outside host is successful.

```text
C1:~# ping -w 5 203.0.113.100
PING 203.0.113.100 (203.0.113.100): 56 data bytes
64 bytes from 203.0.113.100: seq=0 ttl=62 time=2.303 ms
64 bytes from 203.0.113.100: seq=1 ttl=62 time=2.295 ms
64 bytes from 203.0.113.100: seq=2 ttl=62 time=2.173 ms
64 bytes from 203.0.113.100: seq=3 ttl=62 time=2.261 ms
64 bytes from 203.0.113.100: seq=4 ttl=62 time=2.174 ms

--- 203.0.113.100 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.173/2.241/2.303 ms
```

---

## C3

### `ping -w 5 203.0.113.100`

**Expected Results**

* [x] C3 receives ICMP replies from `203.0.113.100`.
* [x] Traffic sourced from C3 matches ACL 10.
* [x] End-to-end reachability from C3 to the outside host is successful.

```text
C3:~# ping -w 5 203.0.113.100
PING 203.0.113.100 (203.0.113.100): 56 data bytes
64 bytes from 203.0.113.100: seq=0 ttl=62 time=2.220 ms
64 bytes from 203.0.113.100: seq=1 ttl=62 time=2.432 ms
64 bytes from 203.0.113.100: seq=2 ttl=62 time=2.548 ms
64 bytes from 203.0.113.100: seq=3 ttl=62 time=2.664 ms
64 bytes from 203.0.113.100: seq=4 ttl=62 time=2.808 ms

--- 203.0.113.100 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 2.220/2.534/2.808 ms
```
