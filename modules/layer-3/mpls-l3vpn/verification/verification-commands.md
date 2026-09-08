# MPLS L3VPN Service Provider Lab Verification Output

This file contains recorded verification output for the MPLS L3VPN Service Provider Lab.

The checks below confirm customer LAN operation, customer-edge routing, IS-IS provider underlay reachability, LDP label distribution, MPLS forwarding state, MP-BGP VPNv4 route exchange and VPN label assignment, VRF route installation, provider-core customer-route isolation, and end-to-end customer connectivity across the MPLS L3VPN.

---

## Customer LAN Verification

### Site A — Layer 2 Verification

**TSA1**

### `show interfaces GigabitEthernet0/0 status`

**Expected Results**

* [x] `GigabitEthernet0/0` is in a `connected` state.
* [x] The physical link toward TSD1 is operational.

```text
TSA1#show interfaces GigabitEthernet0/0 status

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi0/0     LINK_TO_TSD1       connected    10         a-full   auto RJ45
```

### `show interfaces GigabitEthernet0/0 switchport`

**Expected Results**

* [x] `GigabitEthernet0/0` is operating as an access port.
* [x] The access VLAN is `10`.

```text
TSA1#show interfaces GigabitEthernet0/0 switchport

Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (SITE_A_VLAN)

---Output Omitted---

```

### `show vlan brief`

**Expected Results**

* [x] VLAN `10` exists and is active.
* [x] `GigabitEthernet0/0` is assigned to VLAN `10`.

```text
TSA1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/1, Gi0/2, Gi0/3, Gi1/0
                                                Gi1/1, Gi1/2, Gi1/3, Gi2/0
                                                Gi2/1, Gi2/2, Gi2/3, Gi3/0
                                                Gi3/1, Gi3/2, Gi3/3
10   SITE_A_VLAN                      active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

**TSD1**

### `show interfaces GigabitEthernet0/0 status`

**Expected Results**

* [x] `GigabitEthernet0/0` is in a `connected` state.
* [x] The physical link toward TSA1 is operational.

```text
TSD1#show interfaces GigabitEthernet0/0 status

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi0/0     LINK_TO_TSA1       connected    10         a-full   auto RJ45
```

### `show interfaces GigabitEthernet0/0 switchport`

**Expected Results**

* [x] `GigabitEthernet0/0` is operating as an access port.
* [x] The access VLAN is `10`.

```text
TSD1#show interfaces GigabitEthernet0/0 switchport

Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (SITE_A_VLAN)

---Output Omitted---
```

### `show vlan brief`

**Expected Results**

* [x] VLAN `10` exists and is active.
* [x] `GigabitEthernet0/0` is assigned to VLAN `10`.

```text
TSD1#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
10   SITE_A_VLAN                      active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### Site A — Layer 3 Verification

**TSA1**

### `show ip interface brief`

**Expected Results**

* [x] `Vlan10` is `up/up`.
* [x] `Vlan10` has IP address `10.10.10.10`.

```text
TSA1#show ip interface brief

---Output Omitted---
Vlan10                 10.10.10.10     YES manual up                    up 
---Output Omitted---
```

### `show running-config | include ip default-gateway`

**Expected Results**

* [x] The configured default gateway is `10.10.10.1`.
* [x] TSA1 uses TSD1 as its Layer 3 gateway.

```text
TSA1#show running-config | include ip default-gateway

ip default-gateway 10.10.10.1
```

### `ping 10.10.10.1`

**Expected Results**

* [x] ICMP echo requests to `10.10.10.1` succeed.
* [x] TSA1 has Layer 3 reachability to TSD1's VLAN 10 gateway.

```text
TSA1#ping 10.10.10.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/5 ms
```

**TSD1**

### `show ip interface brief`

**Expected Results**

* [x] `Vlan10` is `up/up` with IP address `10.10.10.1`.
* [x] The routed interface toward CE1 is `up/up` with IP address `10.10.255.1`.

```text
TSD1#show ip interface brief

---Output Omitted---
GigabitEthernet0/1     10.10.255.1     YES manual up                    up 
---Output Omitted---
Vlan10                 10.10.10.1      YES manual up                    up   
---Output Omitted---
```

### `show ip route 0.0.0.0`

**Expected Results**

* [x] A default route for `0.0.0.0/0` is installed.
* [x] The next hop is CE1 at `10.10.255.2`.

```text
TSD1#show ip route 0.0.0.0

Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 10.10.255.2
      Route metric is 0, traffic share count is 1
```

### `ping 10.10.255.2`

**Expected Results**

* [x] ICMP echo requests to `10.10.255.2` succeed.
* [x] TSD1 has Layer 3 reachability to CE1.

```text
TSD1#ping 10.10.255.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.255.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

### Site B — Layer 2 Verification

**TSA2**

### `show interfaces GigabitEthernet0/0 status`

**Expected Results**

* [x] `GigabitEthernet0/0` is in a `connected` state.
* [x] The physical link toward TSD2 is operational.

```text
TSA2#show interfaces GigabitEthernet0/0 status

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi0/0     LINK_TO_TSD2         connected    20         a-full   auto RJ45
```

### `show interfaces GigabitEthernet0/0 switchport`

**Expected Results**

* [x] `GigabitEthernet0/0` is operating as an access port.
* [x] The access VLAN is `20`.

```text
TSA2#show interfaces GigabitEthernet0/0 switchport

Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 20 (SITE_B_LAN)
---Output Omitted---
```

### `show vlan brief`

**Expected Results**

* [x] VLAN `20` exists and is active.
* [x] `GigabitEthernet0/0` is assigned to VLAN `20`.

```text
TSA2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/1, Gi0/2, Gi0/3, Gi1/0
                                                Gi1/1, Gi1/2, Gi1/3, Gi2/0
                                                Gi2/1, Gi2/2, Gi2/3, Gi3/0
                                                Gi3/1, Gi3/2, Gi3/3
20   SITE_B_LAN                       active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

**TSD2**

### `show interfaces GigabitEthernet0/0 status`

**Expected Results**

* [x] `GigabitEthernet0/0` is in a `connected` state.
* [x] The physical link toward TSA2 is operational.

```text
TSD2#show interfaces GigabitEthernet0/0 status

Port      Name               Status       Vlan       Duplex  Speed Type 
Gi0/0     LINK_TO_TSA2       connected    20         a-full   auto RJ45
```

### `show interfaces GigabitEthernet0/0 switchport`

**Expected Results**

* [x] `GigabitEthernet0/0` is operating as an access port.
* [x] The access VLAN is `20`.

```text
TSD2#show interfaces GigabitEthernet0/0 switchport

Name: Gi0/0
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: negotiate
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 20 (SITE_B_VLAN)
---Output Omitted---
```

### `show vlan brief`

**Expected Results**

* [x] VLAN `20` exists and is active.
* [x] `GigabitEthernet0/0` is assigned to VLAN `20`.

```text
TSD2#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/2, Gi0/3, Gi1/0, Gi1/1
                                                Gi1/2, Gi1/3, Gi2/0, Gi2/1
                                                Gi2/2, Gi2/3, Gi3/0, Gi3/1
                                                Gi3/2, Gi3/3
20   SITE_B_VLAN                      active    Gi0/0
1002 fddi-default                     act/unsup 
1003 token-ring-default               act/unsup 
1004 fddinet-default                  act/unsup 
1005 trnet-default                    act/unsup 
```

### Site B — Layer 3 Verification

**TSA2**

### `show ip interface brief`

**Expected Results**

* [x] `Vlan20` is `up/up`.
* [x] `Vlan20` has IP address `10.20.20.10`.

```text
TSA2#show ip interface brief

Vlan20                 10.20.20.10     YES manual up                    up  
```

### `show running-config | include ip default-gateway`

**Expected Results**

* [x] The configured default gateway is `10.20.20.1`.
* [x] TSA2 uses TSD2 as its Layer 3 gateway.

```text
TSA2#show running-config | include ip default-gateway

ip default-gateway 10.20.20.1
```

### `ping 10.20.20.1`

**Expected Results**

* [x] ICMP echo requests to `10.20.20.1` succeed.
* [x] TSA2 has Layer 3 reachability to TSD2's VLAN 20 gateway.

```text
TSA2#ping 10.20.20.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.20.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/4/9 ms
```

**TSD2**

### `show ip interface brief`

**Expected Results**

* [x] `Vlan20` is `up/up` with IP address `10.20.20.1`.
* [x] The routed interface toward CE2 is `up/up` with IP address `10.20.255.1`.

```text
TSD2#show ip interface brief

---Output Omitted---
GigabitEthernet0/1     10.20.255.1     YES manual up                    up     
---Output Omitted---
Vlan20                 10.20.20.1      YES manual up                    up        
```

### `show ip route 0.0.0.0`

**Expected Results**

* [x] A default route for `0.0.0.0/0` is installed.
* [x] The next hop is CE2 at `10.20.255.2`.

```text
TSD2#show ip route 0.0.0.0

Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 10.20.255.2
      Route metric is 0, traffic share count is 1
```

### `ping 10.20.255.2`

**Expected Results**

* [x] ICMP echo requests to `10.20.255.2` succeed.
* [x] TSD2 has Layer 3 reachability to CE2.

```text
TSD2#ping 10.20.255.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.255.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/3/5 ms
```

---

## Customer Edge Verification

### CE1 Verification

### `show ip interface brief`

**Expected Results**

* [x] The interface toward TSD1 is `up/up` with IP address `10.10.255.2`.
* [x] The interface toward PE1 is `up/up` with IP address `172.16.1.1`.

```text
CE1#show ip interface brief

Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.10.255.2     YES manual up                    up      
GigabitEthernet0/1         172.16.1.1      YES manual up                    up
---Output Omitted---    
```

### `show ip route 10.10.10.0`

**Expected Results**

* [x] A route to `10.10.10.0/24` is installed.
* [x] The next hop is TSD1 at `10.10.255.1`.

```text
CE1#show ip route 10.10.10.0

Routing entry for 10.10.10.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.10.255.1
      Route metric is 0, traffic share count is 1
```

### `show ip route 0.0.0.0`

**Expected Results**

* [x] A default route for `0.0.0.0/0` is installed.
* [x] The next hop is PE1 at `172.16.1.2`.

```text
CE1#show ip route 0.0.0.0

Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 172.16.1.2
      Route metric is 0, traffic share count is 1
```

### `ping 10.10.255.1`

**Expected Results**

* [x] ICMP echo requests to `10.10.255.1` succeed.
* [x] CE1 has Layer 3 reachability to TSD1.

```text
CE1#ping 10.10.255.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.255.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

### `ping 172.16.1.2`

**Expected Results**

* [x] ICMP echo requests to `172.16.1.2` succeed.
* [x] The CE1-to-PE1 provider handoff is operational.

```text
CE1#ping 172.16.1.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.1.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
```

### CE2 Verification

### `show ip interface brief`

**Expected Results**

* [x] The interface toward TSD2 is `up/up` with IP address `10.20.255.2`.
* [x] The interface toward PE2 is `up/up` with IP address `172.16.2.2`.

```text
CE2#show ip interface brief

Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         10.20.255.2     YES manual up                    up      
GigabitEthernet0/1         172.16.2.2      YES manual up                    up 
```

### `show ip route 10.20.20.0`

**Expected Results**

* [x] A route to `10.20.20.0/24` is installed.
* [x] The next hop is TSD2 at `10.20.255.1`.

```text
CE2#show ip route 10.20.20.0

Routing entry for 10.20.20.0/24
  Known via "static", distance 1, metric 0
  Routing Descriptor Blocks:
  * 10.20.255.1
      Route metric is 0, traffic share count is 1
```

### `show ip route 0.0.0.0`

**Expected Results**

* [x] A default route for `0.0.0.0/0` is installed.
* [x] The next hop is PE2 at `172.16.2.1`.

```text
CE2#show ip route 0.0.0.0

Routing entry for 0.0.0.0/0, supernet
  Known via "static", distance 1, metric 0, candidate default path
  Routing Descriptor Blocks:
  * 172.16.2.1
      Route metric is 0, traffic share count is 1
```

### `ping 10.20.255.1`

**Expected Results**

* [x] ICMP echo requests to `10.20.255.1` succeed.
* [x] CE2 has Layer 3 reachability to TSD2.

```text
CE2#ping 10.20.255.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.255.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/4/6 ms
```

### `ping 172.16.2.1`

**Expected Results**

* [x] ICMP echo requests to `172.16.2.1` succeed.
* [x] The CE2-to-PE2 provider handoff is operational.

```text
CE2#ping 172.16.2.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.16.2.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/4 ms
```

---

## Provider Verification

### IS-IS Verification

**PE1**

### `show isis neighbors`

**Expected Results**

* [x] PE1 has formed an IS-IS adjacency with P1.
* [x] The adjacency is operational at IS-IS Level 2.

```text
PE1#show isis neighbors

Tag PROVIDER_CORE:
System Id       Type Interface     IP Address      State Holdtime Circuit Id
P1              L2   Gi0/1         10.0.12.2       UP    7        P1.01   
```

### `show ip route isis`

**Expected Results**

* [x] PE1 is learning provider routes through IS-IS.
* [x] PE2's loopback `3.3.3.3/32` is reachable through IS-IS.

```text
PE1#show ip route isis

---Output Omitted---
Gateway of last resort is not set

      2.0.0.0/32 is subnetted, 1 subnets
i L2     2.2.2.2 [115/20] via 10.0.12.2, 01:36:25, GigabitEthernet0/1
      3.0.0.0/32 is subnetted, 1 subnets
i L2     3.3.3.3 [115/30] via 10.0.12.2, 01:28:07, GigabitEthernet0/1
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
i L2     10.0.23.0/30 [115/20] via 10.0.12.2, 01:36:25, GigabitEthernet0/1
```

### `ping 3.3.3.3 source 1.1.1.1`

**Expected Results**

* [x] ICMP echo requests sourced from `1.1.1.1` to `3.3.3.3` succeed.
* [x] End-to-end provider underlay reachability from PE1 to PE2 is operational.

```text
PE1#ping 3.3.3.3 source 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
Packet sent with a source address of 1.1.1.1 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 3/3/4 ms
```

**P1**

### `show isis neighbors`

**Expected Results**

* [x] P1 has formed an IS-IS adjacency with PE1.
* [x] P1 has formed an IS-IS adjacency with PE2.
* [x] Both adjacencies are operational at IS-IS Level 2.

```text
P1#show isis neighbors

Tag PROVIDER_CORE:
System Id       Type Interface     IP Address      State Holdtime Circuit Id
PE1             L2   Gi0/0         10.0.12.1       UP    23       P1.01              
PE2             L2   Gi0/1         10.0.23.2       UP    9        PE2.01
```

### `show ip route isis`

**Expected Results**

* [x] P1 is learning provider routes through IS-IS.
* [x] PE1's loopback `1.1.1.1/32` is reachable.
* [x] PE2's loopback `3.3.3.3/32` is reachable.

```text
P1#show ip route isis

---Output Omitted---
Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
i L2     1.1.1.1 [115/20] via 10.0.12.1, 01:40:58, GigabitEthernet0/0
      3.0.0.0/32 is subnetted, 1 subnets
i L2     3.3.3.3 [115/20] via 10.0.23.2, 01:32:40, GigabitEthernet0/1
```

### `ping 1.1.1.1 source 2.2.2.2`

**Expected Results**

* [x] ICMP echo requests sourced from `2.2.2.2` to `1.1.1.1` succeed.
* [x] P1 has provider underlay reachability to PE1.

```text
P1#ping 1.1.1.1 source 2.2.2.2

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
Packet sent with a source address of 2.2.2.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
```

### `ping 3.3.3.3 source 2.2.2.2`

**Expected Results**

* [x] ICMP echo requests sourced from `2.2.2.2` to `3.3.3.3` succeed.
* [x] P1 has provider underlay reachability to PE2.

```text
P1#ping 3.3.3.3 source 2.2.2.2

Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
Packet sent with a source address of 2.2.2.2 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 2/2/3 ms
```

**PE2**

### `show isis neighbors`

**Expected Results**

* [x] PE2 has formed an IS-IS adjacency with P1.
* [x] The adjacency is operational at IS-IS Level 2.

```text
PE2#show isis neighbors

Tag PROVIDER_CORE:
System Id       Type Interface     IP Address      State Holdtime Circuit Id
P1              L2   Gi0/0         10.0.23.1       UP    24       PE2.01 
```

### `show ip route isis`

**Expected Results**

* [x] PE2 is learning provider routes through IS-IS.
* [x] PE1's loopback `1.1.1.1/32` is reachable through IS-IS.

```text
PE2#show ip route isis

---Output Omitted---

Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
i L2     1.1.1.1 [115/30] via 10.0.23.1, 01:36:16, GigabitEthernet0/0
      2.0.0.0/32 is subnetted, 1 subnets
i L2     2.2.2.2 [115/20] via 10.0.23.1, 01:36:22, GigabitEthernet0/0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
i L2     10.0.12.0/30 [115/20] via 10.0.23.1, 01:36:22, GigabitEthernet0/0
```

### `ping 1.1.1.1 source 3.3.3.3`

**Expected Results**

* [x] ICMP echo requests sourced from `3.3.3.3` to `1.1.1.1` succeed.
* [x] End-to-end provider underlay reachability from PE2 to PE1 is operational.

```text
PE2#ping 1.1.1.1 source 3.3.3.3

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
Packet sent with a source address of 3.3.3.3 
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/4/5 ms
```

### LDP Verification

**PE1**

### `show mpls ldp neighbor`

**Expected Results**

* [x] PE1 has formed an LDP neighbor relationship with P1.
* [x] The LDP session is operational.

```text
PE1#show mpls ldp neighbor

    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 1.1.1.1:0
        TCP connection: 2.2.2.2.18729 - 1.1.1.1.646
        State: Oper; Msgs sent/rcvd: 130/128; Downstream
        Up time: 01:46:07
        LDP discovery sources:
          GigabitEthernet0/1, Src IP addr: 10.0.12.2
        Addresses bound to peer LDP Ident:
          2.2.2.2         10.0.12.2       10.0.23.1
```

### `show mpls ldp bindings`

**Expected Results**

* [x] PE1 has local label bindings for provider FECs.
* [x] PE1 has remote label bindings learned through LDP.
* [x] Provider loopback FECs have associated label information.

```text
PE1#show mpls ldp bindings

  lib entry: 1.1.1.1/32, rev 2
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: 16
  lib entry: 2.2.2.2/32, rev 6
        local binding:  label: 17
        remote binding: lsr: 2.2.2.2:0, label: imp-null
  lib entry: 3.3.3.3/32, rev 10
        local binding:  label: 19
        remote binding: lsr: 2.2.2.2:0, label: 17
  lib entry: 10.0.12.0/30, rev 4
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: imp-null
  lib entry: 10.0.23.0/30, rev 8
        local binding:  label: 18
        remote binding: lsr: 2.2.2.2:0, label: imp-null
```

**P1**

### `show mpls ldp neighbor`

**Expected Results**

* [x] P1 has formed an LDP neighbor relationship with PE1.
* [x] P1 has formed an LDP neighbor relationship with PE2.

```text
P1#show mpls ldp neighbor

    Peer LDP Ident: 1.1.1.1:0; Local LDP Ident 2.2.2.2:0
        TCP connection: 1.1.1.1.646 - 2.2.2.2.18729
        State: Oper; Msgs sent/rcvd: 179/179; Downstream
        Up time: 02:30:17
        LDP discovery sources:
          GigabitEthernet0/0, Src IP addr: 10.0.12.1
        Addresses bound to peer LDP Ident:
          10.0.12.1       1.1.1.1         
    Peer LDP Ident: 3.3.3.3:0; Local LDP Ident 2.2.2.2:0
        TCP connection: 3.3.3.3.43261 - 2.2.2.2.646
        State: Oper; Msgs sent/rcvd: 169/169; Downstream
        Up time: 02:22:03
        LDP discovery sources:
          GigabitEthernet0/1, Src IP addr: 10.0.23.2
        Addresses bound to peer LDP Ident:
          10.0.23.2       3.3.3.3   
```

### `show mpls ldp bindings`

**Expected Results**

* [x] P1 has local label bindings for provider FECs.
* [x] P1 has remote label bindings learned from PE1 and PE2.
* [x] Provider loopback FECs have associated label information.

```text
P1#show mpls ldp bindings

  lib entry: 1.1.1.1/32, rev 2
        local binding:  label: 16
        remote binding: lsr: 1.1.1.1:0, label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 19
  lib entry: 2.2.2.2/32, rev 4
        local binding:  label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 17
        remote binding: lsr: 3.3.3.3:0, label: 17
  lib entry: 3.3.3.3/32, rev 10
        local binding:  label: 17
        remote binding: lsr: 3.3.3.3:0, label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 19
  lib entry: 10.0.12.0/30, rev 6
        local binding:  label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: imp-null
        remote binding: lsr: 3.3.3.3:0, label: 18
  lib entry: 10.0.23.0/30, rev 8
        local binding:  label: imp-null
        remote binding: lsr: 1.1.1.1:0, label: 18
        remote binding: lsr: 3.3.3.3:0, label: imp-null
```

**PE2**

### `show mpls ldp neighbor`

**Expected Results**

* [x] PE2 has formed an LDP neighbor relationship with P1.
* [x] The LDP session is operational.

```text
PE2#show mpls ldp neighbor

    Peer LDP Ident: 2.2.2.2:0; Local LDP Ident 3.3.3.3:0
        TCP connection: 2.2.2.2.646 - 3.3.3.3.43261
        State: Oper; Msgs sent/rcvd: 123/123; Downstream
        Up time: 01:41:28
        LDP discovery sources:
          GigabitEthernet0/0, Src IP addr: 10.0.23.1
        Addresses bound to peer LDP Ident:
          2.2.2.2         10.0.12.2       10.0.23.1  
```

### `show mpls ldp bindings`

**Expected Results**

* [x] PE2 has local label bindings for provider FECs.
* [x] PE2 has remote label bindings learned through LDP.
* [x] Provider loopback FECs have associated label information.

```text
PE2#show mpls ldp bindings

  lib entry: 1.1.1.1/32, rev 10
        local binding:  label: 19
        remote binding: lsr: 2.2.2.2:0, label: 16
  lib entry: 2.2.2.2/32, rev 6
        local binding:  label: 17
        remote binding: lsr: 2.2.2.2:0, label: imp-null
  lib entry: 3.3.3.3/32, rev 2
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: 17
  lib entry: 10.0.12.0/30, rev 8
        local binding:  label: 18
        remote binding: lsr: 2.2.2.2:0, label: imp-null
  lib entry: 10.0.23.0/30, rev 4
        local binding:  label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: imp-null
```

### MPLS Forwarding Verification

**PE1**

### `show mpls forwarding-table`

**Expected Results**

* [x] PE1 has active MPLS forwarding entries for provider FECs.
* [x] Valid outgoing label operations and next hops are present.

```text
PE1#show mpls forwarding-table

Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         No Label   10.10.10.0/24[V] 1972          Gi0/0      172.16.1.1  
17         Pop Label  2.2.2.2/32       0             Gi0/1      10.0.12.2   
18         Pop Label  10.0.23.0/30     0             Gi0/1      10.0.12.2   
19         17         3.3.3.3/32       0             Gi0/1      10.0.12.2 
```

**P1**

### `show mpls forwarding-table`

**Expected Results**

* [x] P1 has active MPLS forwarding entries.
* [x] Label-switching state exists toward both provider edges.
* [x] Appropriate label swap or pop operations are present.

```text
P1#show mpls forwarding-table

Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         Pop Label  1.1.1.1/32       19094         Gi0/0      10.0.12.1   
17         Pop Label  3.3.3.3/32       18950         Gi0/1      10.0.23.2 
```

**PE2**

### `show mpls forwarding-table`

**Expected Results**

* [x] PE2 has active MPLS forwarding entries for provider FECs.
* [x] Valid outgoing label operations and next hops are present.

```text
PE2#show mpls forwarding-table

Local      Outgoing   Prefix           Bytes Label   Outgoing   Next Hop    
Label      Label      or Tunnel Id     Switched      interface              
16         No Label   10.20.20.0/24[V] 1762          Gi0/1      172.16.2.2  
17         Pop Label  2.2.2.2/32       0             Gi0/0      10.0.23.1   
18         Pop Label  10.0.12.0/30     0             Gi0/0      10.0.23.1   
19         16         1.1.1.1/32       0             Gi0/0      10.0.23.1   
```

### MP-BGP VPNv4 Verification

**PE1**

### `show ip bgp vpnv4 all summary`

**Expected Results**

* [x] The VPNv4 iBGP session with neighbor `3.3.3.3` is established.
* [x] PE1 is receiving VPNv4 prefixes from PE2.

```text
PE1#show ip bgp vpnv4 all summary

BGP router identifier 1.1.1.1, local AS number 65000
BGP table version is 4, main routing table version 4
3 network entries using 468 bytes of memory
3 path entries using 252 bytes of memory
2/2 BGP path/bestpath attribute entries using 336 bytes of memory
1 BGP extended community entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1080 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
3.3.3.3         4        65000     128     127        4    0    0 01:50:51        1
```

### `show ip bgp vpnv4 all`

**Expected Results**

* [x] The VPNv4 table contains Site A prefix `10.10.10.0/24`.
* [x] The VPNv4 table contains Site B prefix `10.20.20.0/24`.
* [x] Customer routes are represented as RD-qualified VPNv4 prefixes.

```text
PE1#show ip bgp vpnv4 all
   
BGP table version is 4, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
              t secondary path, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 65000:101 (default for vrf CUSTOMER_A)
 *>   10.10.10.0/24    172.16.1.1               0         32768 ?
 *>i  10.20.20.0/24    3.3.3.3                  0    100      0 ?
Route Distinguisher: 65000:102
 *>i  10.20.20.0/24    3.3.3.3                  0    100      0 ?
```

### `show ip bgp vpnv4 all labels`

**Expected Results**

* [x] CUSTOMER_A VPNv4 routes have associated MPLS VPN/service labels.
* [x] VPN/service label information is present for customer routes exchanged through MP-BGP.
* [x] The VPN/service labels are separate from the LDP transport labels used across the provider core.

```text
PE1#show ip bgp vpnv4 all labels

   Network          Next Hop      In label/Out label
Route Distinguisher: 65000:101 (CUSTOMER_A)
   10.10.10.0/24    172.16.1.1      16/nolabel
   10.20.20.0/24    3.3.3.3         nolabel/16
Route Distinguisher: 65000:102
   10.20.20.0/24    3.3.3.3         nolabel/16

```

**PE2**

### `show ip bgp vpnv4 all summary`

**Expected Results**

* [x] The VPNv4 iBGP session with neighbor `1.1.1.1` is established.
* [x] PE2 is receiving VPNv4 prefixes from PE1.

```text
PE2#show ip bgp vpnv4 all summary

BGP router identifier 3.3.3.3, local AS number 65000
BGP table version is 4, main routing table version 4
3 network entries using 468 bytes of memory
3 path entries using 252 bytes of memory
2/2 BGP path/bestpath attribute entries using 336 bytes of memory
1 BGP extended community entries using 24 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1080 total bytes of memory
BGP activity 3/0 prefixes, 3/0 paths, scan interval 60 secs

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4        65000     134     134        4    0    0 01:56:36        1
```

### `show ip bgp vpnv4 all`

**Expected Results**

* [x] The VPNv4 table contains Site B prefix `10.20.20.0/24`.
* [x] The VPNv4 table contains Site A prefix `10.10.10.0/24`.
* [x] Customer routes are represented as RD-qualified VPNv4 prefixes.

```text
PE2#show ip bgp vpnv4 all

BGP table version is 4, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal, 
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter, 
              x best-external, a additional-path, c RIB-compressed, 
              t secondary path, 
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 65000:101
 *>i  10.10.10.0/24    1.1.1.1                  0    100      0 ?
Route Distinguisher: 65000:102 (default for vrf CUSTOMER_A)
 *>i  10.10.10.0/24    1.1.1.1                  0    100      0 ?
 *>   10.20.20.0/24    172.16.2.2               0         32768 ?
```

### `show ip bgp vpnv4 all labels`

**Expected Results**

* [x] CUSTOMER_A VPNv4 routes have associated MPLS VPN/service labels.
* [x] VPN/service label information is present for customer routes exchanged through MP-BGP.
* [x] The VPN/service labels are separate from the LDP transport labels used across the provider core.

```text
PE2#show ip bgp vpnv4 all labels

   Network          Next Hop      In label/Out label
Route Distinguisher: 65000:101
   10.10.10.0/24    1.1.1.1         nolabel/16
Route Distinguisher: 65000:102 (CUSTOMER_A)
   10.10.10.0/24    1.1.1.1         nolabel/16
   10.20.20.0/24    172.16.2.2      16/nolabel
```

### VRF Route Verification

**PE1**

### `show ip route vrf CUSTOMER_A`

**Expected Results**

* [x] CUSTOMER_A contains local Site A prefix `10.10.10.0/24`.
* [x] CUSTOMER_A contains remote Site B prefix `10.20.20.0/24`.
* [x] The remote Site B route has been successfully imported into the VRF.

```text
PE1#show ip route vrf CUSTOMER_A

Routing Table: CUSTOMER_A
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

      10.0.0.0/24 is subnetted, 2 subnets
S        10.10.10.0 [1/0] via 172.16.1.1
B        10.20.20.0 [200/0] via 3.3.3.3, 01:58:28
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.1.0/30 is directly connected, GigabitEthernet0/0
L        172.16.1.2/32 is directly connected, GigabitEthernet0/0
```

**PE2**

### `show ip route vrf CUSTOMER_A`

**Expected Results**

* [x] CUSTOMER_A contains local Site B prefix `10.20.20.0/24`.
* [x] CUSTOMER_A contains remote Site A prefix `10.10.10.0/24`.
* [x] The remote Site A route has been successfully imported into the VRF.

```text
PE2#show ip route vrf CUSTOMER_A

Routing Table: CUSTOMER_A
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

      10.0.0.0/24 is subnetted, 2 subnets
B        10.10.10.0 [200/0] via 1.1.1.1, 01:59:45
S        10.20.20.0 [1/0] via 172.16.2.2
      172.16.0.0/16 is variably subnetted, 2 subnets, 2 masks
C        172.16.2.0/30 is directly connected, GigabitEthernet0/1
L        172.16.2.1/32 is directly connected, GigabitEthernet0/1
```

### Provider Core Customer-Route Isolation Verification

**P1**

### `show ip route 10.10.10.0`

**Expected Results**

* [x] Site A customer prefix `10.10.10.0/24` is not present in P1's global routing table.
* [x] P1 does not require Site A customer routing information to transport MPLS L3VPN traffic.

```text
P1#show ip route 10.10.10.0

% Subnet not in table
```

### `show ip route 10.20.20.0`

**Expected Results**

* [x] Site B customer prefix `10.20.20.0/24` is not present in P1's global routing table.
* [x] P1 does not require Site B customer routing information to transport MPLS L3VPN traffic.
* [x] Customer routing information remains confined to the provider edge devices.

```text
P1#show ip route 10.20.20.0

% Subnet not in table
```

---

## End-to-End Customer Verification

**TSA1 → TSA2**

### `ping 10.20.20.10`

**Expected Results**

* [x] ICMP echo requests to `10.20.20.10` succeed.
* [x] Site A can reach Site B across the MPLS L3VPN.
* [x] End-to-end CUSTOMER_A forwarding is operational from PE1 through the MPLS core to PE2.

```text
TSA1#ping 10.20.20.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.20.20.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 12/13/14 ms
```

**TSA2 → TSA1**

### `ping 10.10.10.10`

**Expected Results**

* [x] ICMP echo requests to `10.10.10.10` succeed.
* [x] Site B can reach Site A across the MPLS L3VPN.
* [x] End-to-end CUSTOMER_A forwarding is operational from PE2 through the MPLS core to PE1.

```text
TSA2#ping 10.10.10.10

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.10.10.10, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 11/12/14 ms
```