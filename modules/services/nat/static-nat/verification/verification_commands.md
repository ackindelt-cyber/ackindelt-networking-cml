# Static NAT Verification Output

This file contains recorded verification output for the Static NAT lab.

The checks below confirm the static one-to-one NAT mapping, NAT inside and outside interface roles, NAT translation counters, default routing, inside-to-outside reachability, and outside-to-inside reachability using the inside global address.

---

## R1

### `show ip nat translations`

**Expected Results**

* [x] Static NAT entry is present.
* [x] Inside local address `192.168.10.10` is mapped to inside global address `203.0.113.66`.
* [x] Active ICMP translation appears after traffic is generated.
* [x] Outside host address `203.0.114.100` appears in the active translation.

```text
R1>show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 203.0.113.66:1518 192.168.10.10:1518 203.0.114.100:1518 203.0.114.100:1518
--- 203.0.113.66       192.168.10.10      ---                ---
```

---

### `show ip nat statistics`

**Expected Results**

* [x] NAT is enabled.
* [x] Outside interface is `GigabitEthernet0/1`.
* [x] Inside interface is `GigabitEthernet0/0`.
* [x] Static translation is counted.
* [x] NAT hit counter increments after traffic is generated.

```text
R1>show ip nat statistics
Total active translations: 2 (1 static, 1 dynamic; 1 extended)
Peak translations: 3, occurred 00:36:17 ago
Outside interfaces:
  GigabitEthernet0/1
Inside interfaces: 
  GigabitEthernet0/0
Hits: 2749  Misses: 0
CEF Translated packets: 2744, CEF Punted packets: 5
Expired translations: 6
Dynamic mappings:

Total doors: 0
Appl doors: 0
Normal doors: 0
Queued Packets: 0
```

---

### `show ip route`

**Expected Results**

* [x] Default route exists.
* [x] Default route points to next hop `198.51.100.1`.
* [x] Inside LAN `192.168.10.0/24` is directly connected.
* [x] WAN network `198.51.100.0/30` is directly connected.

```text
R1>show ip route
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

### `ping -w 5 203.0.114.100`

**Expected Results**

* [x] C1 receives replies from outside host `203.0.114.100`.
* [x] Inside-to-outside connectivity succeeds through Static NAT.
* [x] Traffic sourced from `192.168.10.10` is translated to `203.0.113.66`.

```text
cisco@C1:/etc/netplan$ ping -w 5 203.0.114.100
PING 203.0.114.100 (203.0.114.100) 56(84) bytes of data.
64 bytes from 203.0.114.100: icmp_seq=1 ttl=62 time=3.00 ms
64 bytes from 203.0.114.100: icmp_seq=2 ttl=62 time=2.30 ms
64 bytes from 203.0.114.100: icmp_seq=3 ttl=62 time=2.48 ms
64 bytes from 203.0.114.100: icmp_seq=4 ttl=62 time=2.57 ms
64 bytes from 203.0.114.100: icmp_seq=5 ttl=62 time=2.16 ms

--- 203.0.114.100 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 2.163/2.502/2.999/0.285 ms
```

---

## C2

### `ping -w 5 203.0.113.66`

**Expected Results**

* [x] C2 receives replies from inside global address `203.0.113.66`.
* [x] Outside-to-inside connectivity succeeds using the Static NAT mapping.
* [x] Traffic destined for `203.0.113.66` is translated to inside local address `192.168.10.10`.

```text
cisco@C2:/etc/netplan$ ping -w 5 203.0.113.66
PING 203.0.113.66 (203.0.113.66) 56(84) bytes of data.
64 bytes from 203.0.113.66: icmp_seq=1 ttl=62 time=3.40 ms
64 bytes from 203.0.113.66: icmp_seq=2 ttl=62 time=2.48 ms
64 bytes from 203.0.113.66: icmp_seq=3 ttl=62 time=2.63 ms
64 bytes from 203.0.113.66: icmp_seq=4 ttl=62 time=2.07 ms
64 bytes from 203.0.113.66: icmp_seq=5 ttl=62 time=2.47 ms

--- 203.0.113.66 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4006ms
rtt min/avg/max/mdev = 2.071/2.609/3.401/0.436 ms
```
