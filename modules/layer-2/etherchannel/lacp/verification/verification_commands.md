# LACP EtherChannel Verification Output

This file contains recorded verification output for the LACP EtherChannel lab.

The checks below confirm LACP negotiation, port-channel formation, bundled member interfaces, aggregated interface bandwidth, and STP behavior across logical port-channels.

**Scope note:** This file records the intended verification scope for this version of the lab. Member-link failure behavior is intentionally out of scope.

---

## S1

### `show etherchannel summary`

**Expected Results**

* [x] Po1 and Po2 exist and show `(SU)`.
* [x] Protocol shows `LACP`.
* [x] Gi0/0, Gi0/1, Gi0/2, and Gi0/3 show `(P)` bundled state.
* [x] No member links show standalone, down, or suspended states.

```text
S1#show etherchannel summary
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
1      Po1(SU)         LACP      Gi0/0(P)    Gi0/1(P)    
2      Po2(SU)         LACP      Gi0/2(P)    Gi0/3(P)    
```

### `show lacp neighbor`

**Expected Results**

* [x] Neighbors are listed for channel groups 1 and 2.
* [x] Each channel group shows two neighbors, one per physical member link.
* [x] Flags indicate active LACP participation.
* [x] Partner device ID is consistent within each channel group.
* [x] Operational key is consistent within each channel group.
* [x] Age values are present, confirming LACPDU exchange.

```text
S1#show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs 
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode     

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.000e.8000  20s    0x0    0x1    0x1     0x3D  
Gi0/1     SA      32768     5254.000e.8000   3s    0x0    0x1    0x2     0x3D  

Channel group 2 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.001b.8000  12s    0x0    0x2    0x1     0x3D  
Gi0/3     SA      32768     5254.001b.8000  18s    0x0    0x2    0x2     0x3D  
```

### `show interfaces port-channel 1`

**Expected Results**

* [x] Port-channel1 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S1#show interfaces port-channel 1
Port-channel1 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.00f6.116f (bia 5254.00f6.116f)
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
     1222 packets input, 71836 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     172 packets output, 15952 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 2`

**Expected Results**

* [x] Port-channel2 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S1#show interfaces port-channel 2
Port-channel2 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.004b.ecf2 (bia 5254.004b.ecf2)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:01, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     455 packets input, 26279 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     137 packets output, 10844 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po1 and Po2 as logical interfaces.
* [x] STP evaluates the port-channels rather than individual physical member links.
* [x] Blocking on one logical port-channel is expected in this looped triangle topology.

```text
S1#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.000e.91d7
             Cost        3
             Port        65 (Port-channel1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.00f6.116f
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Root FWD 3         128.65   P2p 
Po2                 Altn BLK 3         128.66   P2p 
```

### `show interfaces gigabitEthernet0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `1` and port-channel is `Po1`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S1>show int gi0/0 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/0     SA      bndl      32768         0x1       0x1     0x1         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.000e.8000  10s    0x0    0x1    0x1     0x3D  

Age of the port in the current state: 0d:01h:04m:31s
```

### `show interfaces gigabitEthernet0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `2` and port-channel is `Po2`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S1#show int gi0/2 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 2           Mode = Active          Gcchange = -
Port-channel  = Po2         GC   =   -             Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/2     SA      bndl      32768         0x2       0x2     0x3         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.001b.8000  20s    0x0    0x2    0x1     0x3D  

Age of the port in the current state: 0d:00h:09m:34s
```

---

## S2

### `show etherchannel summary`

**Expected Results**

* [x] Po1 and Po3 exist and show `(SU)`.
* [x] Protocol shows `LACP`.
* [x] Gi0/0, Gi0/1, Gi0/2, and Gi0/3 show `(P)` bundled state.
* [x] No member links show standalone, down, or suspended states.

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
1      Po1(SU)         LACP      Gi0/0(P)    Gi0/1(P)    
3      Po3(SU)         LACP      Gi0/2(P)    Gi0/3(P)     
```

### `show lacp neighbor`

**Expected Results**

* [x] Neighbors are listed for channel groups 1 and 3.
* [x] Each channel group shows two neighbors, one per physical member link.
* [x] Flags indicate active LACP participation.
* [x] Partner device ID is consistent within each channel group.
* [x] Operational key is consistent within each channel group.
* [x] Age values are present, confirming LACPDU exchange.

```text
S2>show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs 
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode     

Channel group 1 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.00f6.8000   8s    0x0    0x1    0x1     0x3D  
Gi0/1     SA      32768     5254.00f6.8000  15s    0x0    0x1    0x2     0x3D  

Channel group 3 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.001b.8000  20s    0x0    0x3    0x3     0x3D  
Gi0/3     SA      32768     5254.001b.8000  18s    0x0    0x3    0x4     0x3D  
```

### `show interfaces port-channel 1`

**Expected Results**

* [x] Port-channel1 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S2>show interfaces port-channel 1
Port-channel1 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.000e.91d7 (bia 5254.000e.91d7)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:10:16, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     4 packets input, 84 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     1609 packets output, 106676 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 3`

**Expected Results**

* [x] Port-channel3 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S2>show interfaces port-channel 3
Port-channel3 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.0088.32fa (bia 5254.0088.32fa)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:10:25, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     5 packets input, 179 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     756 packets output, 49824 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po1 and Po3 as logical interfaces.
* [x] S2 is the STP root bridge for VLAN 1 in the recorded output.
* [x] Po1 and Po3 are Designated and Forwarding.

```text
S2>show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.000e.91d7
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.000e.91d7
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po1                 Desg FWD 3         128.65   P2p 
Po3                 Desg FWD 3         128.66   P2p 
```

### `show interfaces gigabitEthernet0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `1` and port-channel is `Po1`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S2>show int gi0/0 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 1           Mode = Active          Gcchange = -
Port-channel  = Po1         GC   =   -             Pseudo port-channel = Po1
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/0     SA      bndl      32768         0x1       0x1     0x1         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.00f6.8000  23s    0x0    0x1    0x1     0x3D  

Age of the port in the current state: 0d:00h:25m:16s
```

### `show interfaces gigabitEthernet0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `3` and port-channel is `Po3`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S2>show int gi 0/2 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 3           Mode = Active          Gcchange = -
Port-channel  = Po3         GC   =   -             Pseudo port-channel = Po3
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/2     SA      bndl      32768         0x3       0x3     0x3         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.001b.8000  10s    0x0    0x3    0x3     0x3D  

Age of the port in the current state: 0d:00h:13m:03s
```

---

## S3

### `show etherchannel summary`

**Expected Results**

* [x] Po2 and Po3 exist and show `(SU)`.
* [x] Protocol shows `LACP`.
* [x] Gi0/0, Gi0/1, Gi0/2, and Gi0/3 show `(P)` bundled state.
* [x] No member links show standalone, down, or suspended states.

```text
S3#show etherchannel summary
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
2      Po2(SU)         LACP      Gi0/0(P)    Gi0/1(P)    
3      Po3(SU)         LACP      Gi0/2(P)    Gi0/3(P)    
```

### `show lacp neighbor`

**Expected Results**

* [x] Neighbors are listed for channel groups 2 and 3.
* [x] Each channel group shows two neighbors, one per physical member link.
* [x] Flags indicate active LACP participation.
* [x] Partner device ID is consistent within each channel group.
* [x] Operational key is consistent within each channel group.
* [x] Age values are present, confirming LACPDU exchange.

```text
S3#show lacp neighbor
Flags:  S - Device is requesting Slow LACPDUs 
        F - Device is requesting Fast LACPDUs
        A - Device is in Active mode       P - Device is in Passive mode     

Channel group 2 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.00f6.8000  24s    0x0    0x2    0x3     0x3D  
Gi0/1     SA      32768     5254.00f6.8000   2s    0x0    0x2    0x4     0x3D  

Channel group 3 neighbors

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.000e.8000  15s    0x0    0x3    0x3     0x3D  
Gi0/3     SA      32768     5254.000e.8000  18s    0x0    0x3    0x4     0x3D  
```

### `show interfaces port-channel 2`

**Expected Results**

* [x] Port-channel2 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S3#show int port-channel 2
Port-channel2 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.001b.7e31 (bia 5254.001b.7e31)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:14:09, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     64 packets input, 3712 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     985 packets output, 65036 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show interfaces port-channel 3`

**Expected Results**

* [x] Port-channel3 is up and line protocol is up.
* [x] Bandwidth reflects aggregation: `2000000 Kbit/sec` for two 1G member links.
* [x] No input errors, CRCs, output errors, or drops are observed.
* [x] Counters are present.

```text
S3#show int port-channel 3
Port-channel3 is up, line protocol is up (connected) 
  Hardware is EtherChannel, address is 5254.0045.eba8 (bia 5254.0045.eba8)
  MTU 1500 bytes, BW 2000000 Kbit/sec, DLY 10 usec, 
     reliability 255/255, txload 1/255, rxload 1/255
  Encapsulation ARPA, loopback not set
  Keepalive set (10 sec)
  ARP type: ARPA, ARP Timeout 04:00:00
  Last input 00:00:01, output never, output hang never
  Last clearing of "show interface" counters never
  Input queue: 0/2000/0/0 (size/max/drops/flushes); Total output drops: 0
  Queueing strategy: fifo
  Output queue: 0/40 (size/max)
  5 minute input rate 0 bits/sec, 0 packets/sec
  5 minute output rate 0 bits/sec, 0 packets/sec
     882 packets input, 51156 bytes, 0 no buffer
     Received 0 broadcasts (0 multicasts)
     0 runts, 0 giants, 0 throttles 
     0 input errors, 0 CRC, 0 frame, 0 overrun, 0 ignored
     0 input packets with dribble condition detected
     128 packets output, 11780 bytes, 0 underruns
     0 output errors, 0 collisions, 0 interface resets
     0 unknown protocol drops
     0 babbles, 0 late collision, 0 deferred
     0 lost carrier, 0 no carrier
     0 output buffer failures, 0 output buffers swapped out
```

### `show spanning-tree`

**Expected Results**

* [x] STP lists Po2 and Po3 as logical interfaces.
* [x] STP evaluates the port-channels rather than individual physical member links.
* [x] Po3 is the Root port and Forwarding.
* [x] Po2 is Designated and Forwarding.

```text
S3#show spanning-tree

VLAN0001
  Spanning tree enabled protocol ieee
  Root ID    Priority    32769
             Address     5254.000e.91d7
             Cost        3
             Port        66 (Port-channel3)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32769  (priority 32768 sys-id-ext 1)
             Address     5254.001b.7e31
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Po2                 Desg FWD 3         128.65   P2p 
Po3                 Root FWD 3         128.66   P2p 
```

### `show interfaces gigabitEthernet0/0 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `2` and port-channel is `Po2`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S3#show int gi0/0 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 2           Mode = Active          Gcchange = -
Port-channel  = Po2         GC   =   -             Pseudo port-channel = Po2
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/0     SA      bndl      32768         0x2       0x2     0x1         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/0     SA      32768     5254.00f6.8000   1s    0x0    0x2    0x3     0x3D  

Age of the port in the current state: 0d:00h:16m:13s
```

### `show interfaces gigabitEthernet0/2 etherchannel`

**Expected Results**

* [x] Port state shows `Up` and `In-Bndl`.
* [x] Channel group is `3` and port-channel is `Po3`.
* [x] Mode is `Active` and protocol is `LACP`.
* [x] Partner information is present.

```text
S3#show int gi0/2 etherchannel
Port state    = Up Mstr Assoc In-Bndl 
Channel group = 3           Mode = Active          Gcchange = -
Port-channel  = Po3         GC   =   -             Pseudo port-channel = Po3
Port index    = 0           Load = 0x00            Protocol =   LACP

Flags:  S - Device is sending Slow LACPDUs   F - Device is sending fast LACPDUs.
        A - Device is in active mode.        P - Device is in passive mode.

Local information:
                            LACP port     Admin     Oper    Port        Port
Port      Flags   State     Priority      Key       Key     Number      State
Gi0/2     SA      bndl      32768         0x3       0x3     0x3         0x3D  

Partner's information:

                  LACP port                        Admin  Oper   Port    Port
Port      Flags   Priority  Dev ID          Age    key    Key    Number  State
Gi0/2     SA      32768     5254.000e.8000  13s    0x0    0x3    0x3     0x3D  

Age of the port in the current state: 0d:00h:15m:43s
```
