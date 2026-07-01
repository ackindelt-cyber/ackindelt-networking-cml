# Static EtherChannel Verification Output

This file contains recorded verification output for the Static EtherChannel lab.

The checks below confirm that the Port-Channels formed successfully, member interfaces bundled as expected, STP recognized the Port-Channels as logical links, and no interface errors were observed during validation.

---

## S1

### `show etherchannel summary`

**Expected Results**

* [x] Po1 and Po2 exist and show `(SU)`.
* [x] Protocol shows `-`.
* [x] All member links show `(P)` bundled state: Gi0/0, Gi0/1, Gi0/2, Gi0/3.
* [x] No member links show `I`, `D`, or `s` state.

```text
S1>show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)          -        Gi0/0(P)    Gi0/1(P)    
2      Po2(SU)          -        Gi0/2(P)    Gi0/3(P)    
```

### `show interfaces port-channel 1`

**Expected Results**

* [x] Port-channel1 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present and incrementing.

```text
S1>show interfaces port-channel 1
Port-channel1 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.008c.2b4a (bia 5254.008c.2b4a)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:23:59, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     37532 packets input, 3364592 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     42362 packets output, 3712993 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 2`

**Expected Results**

* [x] Port-channel2 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present and incrementing.

```text
S1>show interfaces port-channel 2
Port-channel2 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.001e.bcb9 (bia 5254.001e.bcb9)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:22:13, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     43213 packets input, 3694079 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     45153 packets output, 3884775 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po1 and Po2 as logical interfaces.
* [x] S1 is the root bridge for VLAN 1.
* [x] Po1 and Po2 are forwarding.
* [x] Po1 and Po2 appear as point-to-point Port-Channel links.

```text
S1>show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.008c.2b4a
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.008c.2b4a
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   P2p 
Po2                 Desg FWD 3         128.66   P2p 
```

### `show int gi0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `1`.
* [x] Port-Channel is `Po1`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S1>show int gi0/0 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 1           Mode = On              Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:01h:17m:07s
```

### `show int gi0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `2`.
* [x] Port-Channel is `Po2`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S1>show int gi0/2 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 2           Mode = On              Gcchange = -
Port-channel  = Po2         GC   =   -             Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:01h:16m:51s
```

---

## S2

### `show etherchannel summary`

**Expected Results**

* [x] Po1 and Po3 exist and show `(SU)`.
* [x] Protocol shows `-`.
* [x] All member links show `(P)` bundled state: Gi0/0, Gi0/1, Gi0/2, Gi0/3.
* [x] No member links show `I`, `D`, or `s` state.

```text
S2>show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
1      Po1(SU)          -        Gi0/0(P)    Gi0/1(P)    
3      Po3(SU)          -        Gi0/2(P)    Gi0/3(P)    
```

### `show interfaces port-channel 1`

**Expected Results**

* [x] Port-channel1 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present and incrementing.

```text
S2>show interfaces port-channel 1
Port-channel1 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.00c0.3190 (bia 5254.00c0.3190)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:00, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     9194 packets input, 533252 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     963 packets output, 154004 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 3`

**Expected Results**

* [x] Port-channel3 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present and incrementing.

```text
S2>show interfaces port-channel 3
Port-channel3 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.0079.0a43 (bia 5254.0079.0a43)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:00, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     1038 packets input, 60204 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     108 packets output, 17238 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po1 and Po3 as logical interfaces.
* [x] S2 reaches the root bridge through Po1.
* [x] Po1 is the root port and forwarding.
* [x] Po3 is an alternate port and blocking.
* [x] STP is preventing a Layer 2 loop across the triangular EtherChannel topology.

```text
S2>show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.008c.2b4a
             Cost        3
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.00c0.3190
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Root FWD 3         128.65   P2p 
Po3                 Altn BLK 3         128.66   P2p 
```

### `show int gi0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `1`.
* [x] Port-Channel is `Po1`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S2>show int gi0/0 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 1           Mode = On              Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:02h:34m:22s
```

### `show int gi0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `3`.
* [x] Port-Channel is `Po3`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S2>show int gi0/2 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 3           Mode = On              Gcchange = -
Port-channel  = Po3         GC   =   -             Pseudo port-channel = Po3
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:00h:18m:29s
```

---

## S3

### `show etherchannel summary`

**Expected Results**

* [x] Po2 and Po3 exist and show `(SU)`.
* [x] Protocol shows `-`.
* [x] All member links show `(P)` bundled state: Gi0/0, Gi0/1, Gi0/2, Gi0/3.
* [x] No member links show `I`, `D`, or `s` state.

```text
S3>show etherchannel summary
Flags:  D - down        P - bundled in port-channel
        I - stand-alone s - suspended
        H - Hot-standby (LACP only)
        R - Layer3      S - Layer2
        U - in use      N - not in use, no aggregation
        f - failed to allocate aggregator

        M - not in use, minimum links not met
        m - not in use, port not aggregated due to minimum links not met
        u - unsuitable for bundling
        w - waiting to be aggregated
        d - default port

        A - formed by Auto LAG


Number of channel-groups in use: 2
Number of aggregators:           2

Group  Port-channel  Protocol    Ports
------+-------------+-----------+-----------------------------------------------
2      Po2(SU)          -        Gi0/0(P)    Gi0/1(P)    
3      Po3(SU)          -        Gi0/2(P)    Gi0/3(P)    
```

### `show interfaces port-channel 2`

**Expected Results**

* [x] Port-channel2 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present and incrementing.

```text
S3>show interfaces port-channel 2
Port-channel2 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.00a5.a664 (bia 5254.00a5.a664)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:00, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     1992 packets input, 115536 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     210 packets output, 33542 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 3`

**Expected Results**

* [x] Port-channel3 is `up`, line protocol is `up`.
* [x] Interface state shows `connected`.
* [x] Bandwidth reflects aggregation: `BW 2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, collisions, or drops were observed.
* [x] Counters are present; output counters show transmitted traffic.

```text
S3>show interfaces port-channel 3
Port-channel3 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.004f.7ad5 (bia 5254.004f.7ad5)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input never, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     0 packets input, 0 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     2108 packets output, 150114 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po2 and Po3 as logical interfaces.
* [x] S3 reaches the root bridge through Po2.
* [x] Po2 is the root port and forwarding.
* [x] Po3 is a designated port and forwarding.
* [x] Both Port-Channel links appear as point-to-point STP links.

```text
S3>show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.008c.2b4a
             Cost        3
             Port        65 (Port-channel2)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.00a5.a664
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po2                 Root FWD 3         128.65   P2p 
Po3                 Desg FWD 3         128.66   P2p 
```

### `show interfaces gi0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `2`.
* [x] Port-Channel is `Po2`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S3>show interfaces gi0/0 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 2           Mode = On              Gcchange = -
Port-channel  = Po2         GC   =   -             Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:00h:34m:44s
```

### `show int gi0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `3`.
* [x] Port-Channel is `Po3`.
* [x] Mode shows `On`.
* [x] Protocol shows `-`.

```text
S3>show int gi0/2 etherchannel
Port state    = Up Mstr In-Bndl 
Channel group = 3           Mode = On              Gcchange = -
Port-channel  = Po3         GC   =   -             Pseudo port-channel = Po3
Port index    = 0           Load = 0x00            Protocol =    -

Age of the port in the current state: 0d:00h:33m:25s
```
