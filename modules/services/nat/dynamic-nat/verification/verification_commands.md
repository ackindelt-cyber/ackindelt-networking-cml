# Dynamic NAT Verification Output

This file contains recorded verification output for the Dynamic NAT lab.

The checks below confirm NAT inside and outside interface roles, Dynamic NAT pool configuration, ACL-to-pool binding, NAT pool allocation, active translations, default routing, and inside-to-outside reachability through NAT.

---

## R1

### `show running-config | section nat`

**Expected Results**

* [x] `GigabitEthernet0/0` is configured as NAT inside.
* [x] `GigabitEthernet0/1` is configured as NAT outside.
* [x] NAT pool `NAT_POOL` is configured.
* [x] ACL `1` is bound to NAT pool `NAT_POOL`.

```text
R1#show run | section nat
 ip nat inside
 ip nat outside
ip nat pool NAT_POOL 203.0.113.66 203.0.113.70 netmask 255.255.255.248
ip nat inside source list 1 pool NAT_POOL
```

---

### `show ip nat statistics`

**Expected Results**

* [x] `GigabitEthernet0/1` is listed as the outside interface.
* [x] `GigabitEthernet0/0` is listed as the inside interface.
* [x] Dynamic mapping uses ACL `1` and pool `NAT_POOL`.
* [x] Pool start and end addresses are correct.
* [x] Pool shows five total addresses.
* [x] Pool allocation increases when traffic creates a translation.
* [x] NAT hit counter increments during active translated traffic.

```text
R1#show ip nat statistics
Total active translations: 2 (0 static, 2 dynamic; 1 extended)
Peak translations: 2, occurred 01:36:06 ago
Outside interfaces:
  GigabitEthernet0/1
Inside interfaces: 
  GigabitEthernet0/0
Hits: 3756  Misses: 0
CEF Translated packets: 3752, CEF Punted packets: 4
Expired translations: 2
Dynamic mappings:
-- Inside Source
[Id: 1] access-list 1 pool NAT_POOL refcount 2
 pool NAT_POOL: netmask 255.255.255.248
        start 203.0.113.66 end 203.0.113.70
        type generic, total addresses 5, allocated 1 (20%), misses 0

Total doors: 0
Appl doors: 0
Normal doors: 0
Queued Packets: 0
```

---

### `show ip nat translations`

**Expected Results**

* [x] Dynamic NAT translation is present.
* [x] Inside local address `192.168.10.10` is translated.
* [x] Inside global address `203.0.113.66` is used from the NAT pool.
* [x] Outside host is `203.0.114.100`.

```text
R1#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 203.0.113.66:1406 192.168.10.10:1406 203.0.114.100:1406 203.0.114.100:1406
--- 203.0.113.66       192.168.10.10      ---                ---
```

---

### `show ip route`

**Expected Results**

* [x] Default route exists.
* [x] Default route points to next hop `198.51.100.1`.
* [x] Inside LAN `192.168.10.0/24` is directly connected.
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

### `ping -w 5 203.0.114.100`

**Expected Results**

* [x] C1 receives replies from outside host `203.0.114.100`.
* [x] Inside-to-outside connectivity succeeds through Dynamic NAT.
* [x] NAT translation is created on R1.

```text
cisco@C1:/etc/netplan$ ping -w 5 203.0.114.100
PING 203.0.114.100 (203.0.114.100) 56(84) bytes of data.
64 bytes from 203.0.114.100: icmp_seq=1 ttl=62 time=2.88 ms
64 bytes from 203.0.114.100: icmp_seq=2 ttl=62 time=2.42 ms
64 bytes from 203.0.114.100: icmp_seq=3 ttl=62 time=2.25 ms
64 bytes from 203.0.114.100: icmp_seq=4 ttl=62 time=2.35 ms
64 bytes from 203.0.114.100: icmp_seq=5 ttl=62 time=2.25 ms

--- 203.0.114.100 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4007ms
rtt min/avg/max/mdev = 2.247/2.430/2.879/0.233 ms
```
