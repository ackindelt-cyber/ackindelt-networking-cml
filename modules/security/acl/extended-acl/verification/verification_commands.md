# Extended ACL Verification Output

This file contains recorded verification output for the Extended ACL lab.

The checks below confirm that the named extended ACL exists, the permit and deny statements match traffic, the ACL is applied inbound on R1’s inside interface, R1 has a default route toward the outside network, TCP/8080 traffic is permitted, and ICMP traffic is denied and logged.

---

## R1

### `show ip access-lists EXT_OUTBOUND_FILTER`

**Expected Results**

* [x] ACL `EXT_OUTBOUND_FILTER` is present.
* [x] TCP traffic from `192.168.10.0/24` to host `203.0.113.100` on port `8080` is permitted.
* [x] Permit counter increments after netcat test traffic.
* [x] Explicit `deny ip any any log` statement is present.
* [x] Deny counter increments after denied ICMP traffic.

```text
R1#show ip access-lists EXT_OUTBOUND_FILTER
Extended IP access list EXT_OUTBOUND_FILTER
    10 permit tcp 192.168.10.0 0.0.0.255 host 203.0.113.100 eq 8080 (5 matches)
    20 deny ip any any log (79 matches)
```

### `show ip interface gigabitEthernet0/0`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up.
* [x] Interface IP address is `192.168.10.1/24`.
* [x] Inbound ACL is `EXT_OUTBOUND_FILTER`.
* [x] No outbound ACL is applied.

```text
R1#show ip interface gigabitEthernet0/0
GigabitEthernet0/0 is up, line protocol is up
  Internet address is 192.168.10.1/24
  Broadcast address is 255.255.255.255
  Address determined by setup command
  MTU is 1500 bytes
  Helper address is not set
  Directed broadcast forwarding is disabled
  Outgoing access list is not set
  Inbound  access list is EXT_OUTBOUND_FILTER
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
  Input features: Access List, MCI Check
  IPv4 WCCP Redirect outbound is disabled
  IPv4 WCCP Redirect inbound is disabled
  IPv4 WCCP Redirect exclude is disabled
```

### `show ip route`

**Expected Results**

* [x] Default route exists.
* [x] Default route points to next hop `198.51.100.1`.
* [x] Inside network `192.168.10.0/24` is directly connected.
* [x] WAN network `198.51.100.0/30` is directly connected.

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

### `echo HTTP Traffic Confirmed | nc 203.0.113.100 8080`

**Expected Results**

* [x] TCP connection to `203.0.113.100` on port `8080` succeeds.
* [x] Payload is delivered to S1.
* [x] ACL permit counter increments.

```text
C1:~# echo HTTP Traffic Confirmed | nc 203.0.113.100 8080

cisco@S1:~$ busybox nc -l -p 8080
HTTP Traffic Confirmed
```

---

## C2

### `ping -w 5 203.0.113.100`

**Expected Results**

* [x] ICMP traffic to `203.0.113.100` is denied.
* [x] No ICMP replies are received.
* [x] ACL deny counter increments.
* [x] R1 logs the denied ICMP traffic.

```text
C2:~# ping -w 5 203.0.113.100
PING 203.0.113.100 (203.0.113.100): 56 data bytes

--- 203.0.113.100 ping statistics ---
5 packets transmitted, 0 packets received, 100% packet loss

R1#
*Dec 22 00:10:58.905: %SEC-6-IPACCESSLOGDP: list EXT_OUTBOUND_FILTER denied icmp 192.168.10.20 -> 203.0.113.100 (8/0), 1 packet  
```
