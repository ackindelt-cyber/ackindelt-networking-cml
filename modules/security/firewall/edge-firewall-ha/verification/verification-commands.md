# Edge Firewall HA Verification Output

This file contains recorded verification output for the Edge Firewall HA lab.

The checks below confirm device preparation, interface and VLAN state, routing, HSRP, Spanning Tree, ASAv policy and failover synchronization, dynamic PAT, and end-to-end connectivity under the normal operating state.

---

## ISP

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured ISP hostname.
* [ ] Output includes `hostname ISP`.

```text
ISP#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
ISP#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show ip interface brief`

**Expected Results**

* [ ] Output confirms `Gi0/0` and `Loopback0` are up with the expected addresses.
* [ ] `Gi0/0` is `198.51.100.1` and `Loopback0` is `192.0.2.100`; both are operational.

```text
ISP#show ip interface brief
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/0`

**Expected Results**

* [ ] Output confirms the R1-facing interface configuration.

```text
ISP#show running-config interface gigabitethernet0/0
<paste recorded command output here>
```

### `show running-config interface loopback0`

**Expected Results**

* [ ] Output confirms the external test loopback configuration.

```text
ISP#show running-config interface loopback0
<paste recorded command output here>
```

### `show ip route connected`

**Expected Results**

* [ ] Output confirms the directly connected R1 link and loopback route.

```text
ISP#show ip route connected
<paste recorded command output here>
```

### `show ip route static`

**Expected Results**

* [ ] Output confirms the return route toward the firewall `outside` subnet.

```text
ISP#show ip route static
<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] Output confirms `203.0.113.0/29` resolves through R1.
* [ ] The route resolves to next hop `198.51.100.2`.

```text
ISP#show ip route 203.0.113.0
<paste recorded command output here>
```

### `ping 198.51.100.2`

**Expected Results**

* [ ] Output confirms direct connectivity to R1 `Gi0/1`.

```text
ISP#ping 198.51.100.2
<paste recorded command output here>
```

### `ping 203.0.113.1`

**Expected Results**

* [ ] Output confirms routed connectivity to R1's shared `outside` interface.

```text
ISP#ping 203.0.113.1
<paste recorded command output here>
```

### `ping 203.0.113.2`

**Expected Results**

* [ ] Output confirms connectivity to the active firewall `outside` address.

```text
ISP#ping 203.0.113.2
<paste recorded command output here>
```

---

## R1

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured R1 hostname.
* [ ] Output includes `hostname R1`.

```text
R1#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
R1#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show ip interface brief`

**Expected Results**

* [ ] Output confirms `Gi0/0` and `Gi0/1` are up with the expected addresses.
* [ ] `Gi0/0` is `203.0.113.1` and `Gi0/1` is `198.51.100.2`; both are `up/up`.

```text
R1#show ip interface brief
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/0`

**Expected Results**

* [ ] Output confirms the OS1-facing `outside`-transit configuration.

```text
R1#show running-config interface gigabitethernet0/0
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/1`

**Expected Results**

* [ ] Output confirms the ISP-facing point-to-point configuration.

```text
R1#show running-config interface gigabitethernet0/1
<paste recorded command output here>
```

### `show ip route connected`

**Expected Results**

* [ ] Output confirms the directly connected ISP and `outside`-transit networks.

```text
R1#show ip route connected
<paste recorded command output here>
```

### `show ip route static`

**Expected Results**

* [ ] Output confirms the default route toward the ISP.

```text
R1#show ip route static
<paste recorded command output here>
```

### `show ip route 192.0.2.100`

**Expected Results**

* [ ] Output confirms the external test destination resolves through the default route.
* [ ] The destination resolves through the default route toward `198.51.100.1`.

```text
R1#show ip route 192.0.2.100
<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] Output confirms the firewall `outside` subnet is directly connected.

```text
R1#show ip route 203.0.113.0
<paste recorded command output here>
```

### `ping 198.51.100.1`

**Expected Results**

* [ ] Output confirms direct connectivity to the ISP router.

```text
R1#ping 198.51.100.1
<paste recorded command output here>
```

### `ping 192.0.2.100`

**Expected Results**

* [ ] Output confirms routed connectivity to the simulated external destination.

```text
R1#ping 192.0.2.100
<paste recorded command output here>
```

### `ping 203.0.113.2`

**Expected Results**

* [ ] Output confirms Layer 3 reachability to the active firewall `outside` address.

```text
R1#ping 203.0.113.2
<paste recorded command output here>
```

### `show ip arp 203.0.113.2`

**Expected Results**

* [ ] Output confirms R1 has resolved the active firewall address to a MAC address.

```text
R1#show ip arp 203.0.113.2
<paste recorded command output here>
```

---

## OS1

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured OS1 hostname.

```text
OS1#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
OS1#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show vlan brief`

**Expected Results**

* [ ] Output confirms VLAN 100 exists and `Gi0/0` through `Gi0/2` are assigned to it.
* [ ] VLAN `100` exists as `OUTSIDE_TRANSIT`, with `Gi0/0`, `Gi0/1`, and `Gi0/2` assigned.

```text
OS1#show vlan brief
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/0 switchport`

**Expected Results**

* [ ] Output confirms the R1-facing port is a static access port in VLAN 100.

```text
OS1#show interfaces gigabitethernet0/0 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/1 switchport`

**Expected Results**

* [ ] Output confirms the FW1-facing port is a static access port in VLAN 100.

```text
OS1#show interfaces gigabitethernet0/1 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/2 switchport`

**Expected Results**

* [ ] Output confirms the FW2-facing port is a static access port in VLAN 100.

```text
OS1#show interfaces gigabitethernet0/2 switchport
<paste recorded command output here>
```

### `show interfaces status`

**Expected Results**

* [ ] Output confirms all three `outside`-transit links are physically connected.
* [ ] `Gi0/0`, `Gi0/1`, and `Gi0/2` report a connected state.

```text
OS1#show interfaces status
<paste recorded command output here>
```

### `show mac address-table vlan 100`

**Expected Results**

* [ ] Output confirms OS1 learns MAC addresses from R1 and the firewall pair.

```text
OS1#show mac address-table vlan 100
<paste recorded command output here>
```

### `show spanning-tree vlan 100`

**Expected Results**

* [ ] Output confirms the VLAN 100 ports are forwarding and no unexpected STP blocking exists.

```text
OS1#show spanning-tree vlan 100
<paste recorded command output here>
```

---

## FW1

### `show running-config hostname`

**Expected Results**

* [ ] Output confirms the shared FW-HA hostname.

```text
FW-HA/pri/act#show running-config hostname
<paste recorded command output here>
```

### `show running-config prompt`

**Expected Results**

* [ ] Output confirms the prompt displays priority and active/standby state.

```text
FW-HA/pri/act#show running-config prompt
<paste recorded command output here>
```

### `show interface ip brief`

**Expected Results**

* [ ] Output confirms the `outside` and `inside` interfaces are enabled with the expected addresses.
* [ ] `outside` uses active-role address `203.0.113.2`; `inside` uses active-role address `10.255.0.4`.

```text
FW-HA/pri/act#show interface ip brief
<paste recorded command output here>
```

### `show nameif`

**Expected Results**

* [ ] Output confirms `Gi0/0` is named `outside` and `Gi0/2` is named `inside`.

```text
FW-HA/pri/act#show nameif
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/0`

**Expected Results**

* [ ] Output confirms the `outside` interface and active/standby addressing.

```text
FW-HA/pri/act#show running-config interface gigabitethernet0/0
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/2`

**Expected Results**

* [ ] Output confirms the `inside` interface and active/standby addressing.

```text
FW-HA/pri/act#show running-config interface gigabitethernet0/2
<paste recorded command output here>
```

### `show route`

**Expected Results**

* [ ] Output confirms the connected, default, and internal static routes.

```text
FW-HA/pri/act#show route
<paste recorded command output here>
```

### `show route 0.0.0.0`

**Expected Results**

* [ ] Output confirms the default route points `outside` through `203.0.113.1`.
* [ ] The default route points to `203.0.113.1` through `outside`.

```text
FW-HA/pri/act#show route 0.0.0.0
<paste recorded command output here>
```

### `show route 10.10.10.0`

**Expected Results**

* [ ] Output confirms VLAN 10 is reachable `inside` through `10.255.0.1`.
* [ ] The internal route points to `10.255.0.1` through `inside`.

```text
FW-HA/pri/act#show route 10.10.10.0
<paste recorded command output here>
```

### `show running-config object network INTERNAL_TEST`

**Expected Results**

* [ ] Output confirms the `10.10.10.0/24` network object.

```text
FW-HA/pri/act#show running-config object network INTERNAL_TEST
<paste recorded command output here>
```

### `show nat detail`

**Expected Results**

* [ ] Output confirms dynamic PAT from `inside` to the `outside` interface.

```text
FW-HA/pri/act#show nat detail
<paste recorded command output here>
```

### `show xlate`

**Expected Results**

* [ ] Output displays active translations after test traffic is generated.

```text
FW-HA/pri/act#show xlate
<paste recorded command output here>
```

### `show conn`

**Expected Results**

* [ ] Output displays active firewall connections after test traffic is generated.

```text
FW-HA/pri/act#show conn
<paste recorded command output here>
```

### `show running-config policy-map`

**Expected Results**

* [ ] Output confirms inspect icmp is present under `global_policy`.

```text
FW-HA/pri/act#show running-config policy-map
<paste recorded command output here>
```

### `show running-config service-policy`

**Expected Results**

* [ ] Output confirms `global_policy` is applied globally.

```text
FW-HA/pri/act#show running-config service-policy
<paste recorded command output here>
```

### `show service-policy`

**Expected Results**

* [ ] Output confirms the inspection policy is active and displays runtime counters.

```text
FW-HA/pri/act#show service-policy
<paste recorded command output here>
```

### `show running-config failover`

**Expected Results**

* [ ] Output confirms FW1's failover and state-link configuration.

```text
FW-HA/pri/act#show running-config failover
<paste recorded command output here>
```

### `show failover`

**Expected Results**

* [ ] Output confirms unit identity, active/standby state, peer health, interfaces, and state-link status.
* [ ] FW1 is `Primary/Active`; FW2 is `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

### `show failover state`

**Expected Results**

* [ ] Output summarizes the current state of both failover units.
* [ ] Both units report healthy baseline failover states.

```text
FW-HA/pri/act#show failover state
<paste recorded command output here>
```

### `show monitor-interface`

**Expected Results**

* [ ] Output confirms the `inside` and `outside` interfaces are monitored.
* [ ] Both `inside` and `outside` are monitored.

```text
FW-HA/pri/act#show monitor-interface
<paste recorded command output here>
```

### `show interface gigabitethernet0/1`

**Expected Results**

* [ ] Output confirms the dedicated failover/state interface is operational.

```text
FW-HA/pri/act#show interface gigabitethernet0/1
<paste recorded command output here>
```

### `ping 203.0.113.1`

**Expected Results**

* [ ] Output confirms connectivity to R1 through the `outside` interface.

```text
FW-HA/pri/act#ping 203.0.113.1
<paste recorded command output here>
```

### `ping 10.255.0.1`

**Expected Results**

* [ ] Output confirms connectivity to the VLAN 99 HSRP virtual IP.

```text
FW-HA/pri/act#ping 10.255.0.1
<paste recorded command output here>
```

### `ping 10.10.10.10`

**Expected Results**

* [ ] Output confirms routed connectivity to A1 through the distribution layer.

```text
FW-HA/pri/act#ping 10.10.10.10
<paste recorded command output here>
```

---

## FW2

### `show running-config hostname`

**Expected Results**

* [ ] Output confirms FW2 received the shared FW-HA hostname from FW1.

```text
FW-HA/sec/stby#show running-config hostname
<paste recorded command output here>
```

### `show running-config prompt`

**Expected Results**

* [ ] Output confirms FW2 received the shared prompt configuration.

```text
FW-HA/sec/stby#show running-config prompt
<paste recorded command output here>
```

### `show running-config failover`

**Expected Results**

* [ ] Output confirms FW2 retains its secondary identity and matching failover-link configuration.

```text
FW-HA/sec/stby#show running-config failover
<paste recorded command output here>
```

### `show failover`

**Expected Results**

* [ ] Output confirms FW2 is Secondary, `Standby Ready`, and communicating with FW1.
* [ ] FW2 is `Secondary/Standby Ready`; FW1 is `Primary/Active`.

```text
FW-HA/sec/stby#show failover
<paste recorded command output here>
```

### `show failover state`

**Expected Results**

* [ ] Output confirms the current primary/secondary and active/standby states.
* [ ] Both units report healthy baseline failover states.

```text
FW-HA/sec/stby#show failover state
<paste recorded command output here>
```

### `show failover interface`

**Expected Results**

* [ ] Output confirms the dedicated failover/state link is operational.

```text
FW-HA/sec/stby#show failover interface
<paste recorded command output here>
```

### `show monitor-interface`

**Expected Results**

* [ ] Output confirms the `inside` and `outside` data interfaces are monitored.

```text
FW-HA/sec/stby#show monitor-interface
<paste recorded command output here>
```

### `show interface gigabitethernet0/1`

**Expected Results**

* [ ] Output confirms the dedicated failover/state interface is physically operational.

```text
FW-HA/sec/stby#show interface gigabitethernet0/1
<paste recorded command output here>
```

### `show interface ip brief`

**Expected Results**

* [ ] Output confirms the synchronized `outside` and `inside` interface addressing and status.

```text
FW-HA/sec/stby#show interface ip brief
<paste recorded command output here>
```

### `show nameif`

**Expected Results**

* [ ] Output confirms `Gi0/0` is `outside` and `Gi0/2` is `inside`.

```text
FW-HA/sec/stby#show nameif
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/0`

**Expected Results**

* [ ] Output confirms the synchronized `outside` configuration.

```text
FW-HA/sec/stby#show running-config interface gigabitethernet0/0
<paste recorded command output here>
```

### `show running-config interface gigabitethernet0/2`

**Expected Results**

* [ ] Output confirms the synchronized `inside` configuration.

```text
FW-HA/sec/stby#show running-config interface gigabitethernet0/2
<paste recorded command output here>
```

### `show route`

**Expected Results**

* [ ] Output confirms FW2 received the connected and static routes from FW1.

```text
FW-HA/sec/stby#show route
<paste recorded command output here>
```

### `show nat detail`

**Expected Results**

* [ ] Output confirms FW2 received the dynamic PAT rule.

```text
FW-HA/sec/stby#show nat detail
<paste recorded command output here>
```

### `show running-config policy-map`

**Expected Results**

* [ ] Output confirms the synchronized ICMP inspection configuration.

```text
FW-HA/sec/stby#show running-config policy-map
<paste recorded command output here>
```

### `show running-config service-policy`

**Expected Results**

* [ ] Output confirms the global policy is applied.

```text
FW-HA/sec/stby#show running-config service-policy
<paste recorded command output here>
```

---

## D1

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured D1 hostname.

```text
D1#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
D1#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show running-config | include ^ip routing`

**Expected Results**

* [ ] Output confirms Layer 3 routing is enabled.

```text
D1#show running-config | include ^ip routing
<paste recorded command output here>
```

### `show vlan brief`

**Expected Results**

* [ ] Output confirms VLANs 10 and 99 exist and `Gi0/0` is assigned to VLAN 99.

```text
D1#show vlan brief
<paste recorded command output here>
```

### `show interfaces trunk`

**Expected Results**

* [ ] Output confirms `Gi0/1` carries VLANs 10 and 99 and `Gi0/2` carries VLAN 10.

```text
D1#show interfaces trunk
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/1 switchport`

**Expected Results**

* [ ] Output confirms the D2-facing interface is a static trunk.

```text
D1#show interfaces gigabitethernet0/1 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/2 switchport`

**Expected Results**

* [ ] Output confirms the A1-facing interface is a static trunk.

```text
D1#show interfaces gigabitethernet0/2 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/0 switchport`

**Expected Results**

* [ ] Output confirms the FW1-facing interface is an access port in VLAN 99.

```text
D1#show interfaces gigabitethernet0/0 switchport
<paste recorded command output here>
```

### `show interfaces status`

**Expected Results**

* [ ] Output confirms the physical links are connected.

```text
D1#show interfaces status
<paste recorded command output here>
```

### `show ip interface brief | include Vlan`

**Expected Results**

* [ ] Output confirms the VLAN 10 and VLAN 99 SVIs are up with the expected addresses.

```text
D1#show ip interface brief | include Vlan
<paste recorded command output here>
```

### `show running-config interface vlan 10`

**Expected Results**

* [ ] Output confirms VLAN 10 addressing and HSRP configuration.

```text
D1#show running-config interface vlan 10
<paste recorded command output here>
```

### `show running-config interface vlan 99`

**Expected Results**

* [ ] Output confirms VLAN 99 addressing and HSRP configuration.

```text
D1#show running-config interface vlan 99
<paste recorded command output here>
```

### `show standby brief`

**Expected Results**

* [ ] Output confirms D1 is the expected HSRP Active router for VLANs 10 and 99.
* [ ] D1 is `Active` for HSRP groups `10` and `99` with priority `110`.

```text
D1#show standby brief
<paste recorded command output here>
```

### `show standby vlan 10`

**Expected Results**

* [ ] Output displays detailed VLAN 10 HSRP state, timers, priority, and peer information.

```text
D1#show standby vlan 10
<paste recorded command output here>
```

### `show standby vlan 99`

**Expected Results**

* [ ] Output displays detailed VLAN 99 HSRP state, timers, priority, and peer information.

```text
D1#show standby vlan 99
<paste recorded command output here>
```

### `show spanning-tree vlan 10`

**Expected Results**

* [ ] Output confirms D1's STP role and the forwarding state of VLAN 10 interfaces.
* [ ] D1 is the VLAN `10` root bridge and its participating ports are forwarding.

```text
D1#show spanning-tree vlan 10
<paste recorded command output here>
```

### `show ip route connected`

**Expected Results**

* [ ] Output confirms VLANs 10 and 99 appear as directly connected networks.

```text
D1#show ip route connected
<paste recorded command output here>
```

### `show ip route static`

**Expected Results**

* [ ] Output confirms the default route toward the active firewall address.

```text
D1#show ip route static
<paste recorded command output here>
```

### `show ip route 0.0.0.0`

**Expected Results**

* [ ] Output confirms the default route uses `10.255.0.4` as its next hop.
* [ ] The default route uses next hop `10.255.0.4`.

```text
D1#show ip route 0.0.0.0
<paste recorded command output here>
```

### `ping 10.10.10.3`

**Expected Results**

* [ ] Output confirms VLAN 10 connectivity to D2.

```text
D1#ping 10.10.10.3
<paste recorded command output here>
```

### `ping 10.10.10.10`

**Expected Results**

* [ ] Output confirms VLAN 10 connectivity to A1.

```text
D1#ping 10.10.10.10
<paste recorded command output here>
```

### `ping 10.255.0.3`

**Expected Results**

* [ ] Output confirms VLAN 99 connectivity to D2.

```text
D1#ping 10.255.0.3
<paste recorded command output here>
```

### `ping 10.255.0.4`

**Expected Results**

* [ ] Output confirms connectivity to the active firewall `inside` address.

```text
D1#ping 10.255.0.4
<paste recorded command output here>
```

### `ping 203.0.113.1 source 10.10.10.2`

**Expected Results**

* [ ] Output confirms connectivity through the firewall using an address covered by PAT.

```text
D1#ping 203.0.113.1 source 10.10.10.2
<paste recorded command output here>
```

### `ping 192.0.2.100 source 10.10.10.2`

**Expected Results**

* [ ] Output confirms end-to-end connectivity using a source address covered by PAT.

```text
D1#ping 192.0.2.100 source 10.10.10.2
<paste recorded command output here>
```

---

## D2

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured D2 hostname.

```text
D2#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
D2#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show running-config | include ^ip routing`

**Expected Results**

* [ ] Output confirms Layer 3 routing is enabled.

```text
D2#show running-config | include ^ip routing
<paste recorded command output here>
```

### `show vlan brief`

**Expected Results**

* [ ] Output confirms VLANs 10 and 99 exist and `Gi0/0` is assigned to VLAN 99.

```text
D2#show vlan brief
<paste recorded command output here>
```

### `show interfaces trunk`

**Expected Results**

* [ ] Output confirms `Gi0/1` carries VLANs 10 and 99 and `Gi0/2` carries VLAN 10.

```text
D2#show interfaces trunk
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/1 switchport`

**Expected Results**

* [ ] Output confirms the D1-facing interface is a static trunk.

```text
D2#show interfaces gigabitethernet0/1 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/2 switchport`

**Expected Results**

* [ ] Output confirms the A1-facing interface is a static trunk.

```text
D2#show interfaces gigabitethernet0/2 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/0 switchport`

**Expected Results**

* [ ] Output confirms the FW2-facing interface is an access port in VLAN 99.

```text
D2#show interfaces gigabitethernet0/0 switchport
<paste recorded command output here>
```

### `show interfaces status`

**Expected Results**

* [ ] Output confirms the physical links are connected.

```text
D2#show interfaces status
<paste recorded command output here>
```

### `show ip interface brief | include Vlan`

**Expected Results**

* [ ] Output confirms the VLAN 10 and VLAN 99 SVIs are up with the expected addresses.

```text
D2#show ip interface brief | include Vlan
<paste recorded command output here>
```

### `show running-config interface vlan 10`

**Expected Results**

* [ ] Output confirms VLAN 10 addressing and HSRP configuration.

```text
D2#show running-config interface vlan 10
<paste recorded command output here>
```

### `show running-config interface vlan 99`

**Expected Results**

* [ ] Output confirms VLAN 99 addressing and HSRP configuration.

```text
D2#show running-config interface vlan 99
<paste recorded command output here>
```

### `show standby brief`

**Expected Results**

* [ ] Output confirms D2 is the expected HSRP Standby router for VLANs 10 and 99.
* [ ] D2 is `Standby` for HSRP groups `10` and `99` with priority `100`.

```text
D2#show standby brief
<paste recorded command output here>
```

### `show standby vlan 10`

**Expected Results**

* [ ] Output displays detailed VLAN 10 HSRP state, timers, priority, and active-peer information.

```text
D2#show standby vlan 10
<paste recorded command output here>
```

### `show standby vlan 99`

**Expected Results**

* [ ] Output displays detailed VLAN 99 HSRP state, timers, priority, and active-peer information.

```text
D2#show standby vlan 99
<paste recorded command output here>
```

### `show spanning-tree vlan 10`

**Expected Results**

* [ ] D2 is the intended secondary root bridge for VLAN `10`.
* [ ] `Gi0/1` toward D1 is the root port and is forwarding.
* [ ] `Gi0/2` toward A1 is a designated port and is forwarding.

```text
D2#show spanning-tree vlan 10
<paste recorded command output here>
```

### `show ip route connected`

**Expected Results**

* [ ] Output confirms VLANs 10 and 99 appear as directly connected networks.

```text
D2#show ip route connected
<paste recorded command output here>
```

### `show ip route static`

**Expected Results**

* [ ] Output confirms the default route toward the active firewall address.

```text
D2#show ip route static
<paste recorded command output here>
```

### `show ip route 0.0.0.0`

**Expected Results**

* [ ] Output confirms the default route uses `10.255.0.4` as its next hop.
* [ ] The default route uses next hop `10.255.0.4`.

```text
D2#show ip route 0.0.0.0
<paste recorded command output here>
```

### `ping 10.10.10.2`

**Expected Results**

* [ ] Output confirms VLAN 10 connectivity to D1.

```text
D2#ping 10.10.10.2
<paste recorded command output here>
```

### `ping 10.10.10.10`

**Expected Results**

* [ ] Output confirms VLAN 10 connectivity to A1.

```text
D2#ping 10.10.10.10
<paste recorded command output here>
```

### `ping 10.255.0.2`

**Expected Results**

* [ ] Output confirms VLAN 99 connectivity to D1.

```text
D2#ping 10.255.0.2
<paste recorded command output here>
```

### `ping 10.255.0.4`

**Expected Results**

* [ ] Output confirms connectivity to the active firewall `inside` address.

```text
D2#ping 10.255.0.4
<paste recorded command output here>
```

### `ping 192.0.2.100 source 10.10.10.3`

**Expected Results**

* [ ] Output confirms end-to-end connectivity using a source address covered by the PAT rule.

```text
D2#ping 192.0.2.100 source 10.10.10.3
<paste recorded command output here>
```

---

## A1

### `show running-config | include ^hostname`

**Expected Results**

* [ ] Output confirms the configured A1 hostname.

```text
A1#show running-config | include ^hostname
<paste recorded command output here>
```

### `show running-config | include ^no ip domain lookup`

**Expected Results**

* [ ] Output confirms DNS lookup is disabled.

```text
A1#show running-config | include ^no ip domain lookup
<paste recorded command output here>
```

### `show vlan brief`

**Expected Results**

* [ ] Output confirms VLAN 10 exists.

```text
A1#show vlan brief
<paste recorded command output here>
```

### `show interfaces trunk`

**Expected Results**

* [ ] Output confirms `Gi0/0` and `Gi0/1` carry VLAN 10.

```text
A1#show interfaces trunk
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/0 switchport`

**Expected Results**

* [ ] Output confirms the D1-facing interface is a static trunk allowing VLAN 10.

```text
A1#show interfaces gigabitethernet0/0 switchport
<paste recorded command output here>
```

### `show interfaces gigabitethernet0/1 switchport`

**Expected Results**

* [ ] Output confirms the D2-facing interface is a static trunk allowing VLAN 10.

```text
A1#show interfaces gigabitethernet0/1 switchport
<paste recorded command output here>
```

### `show interfaces status`

**Expected Results**

* [ ] Output confirms both distribution uplinks are physically connected.

```text
A1#show interfaces status
<paste recorded command output here>
```

### `show spanning-tree vlan 10`

**Expected Results**

* [ ] Output confirms one redundant uplink is forwarding and the other is blocking or alternate as expected.
* [ ] The D1-facing uplink is forwarding and the D2-facing uplink is alternate or blocking.

```text
A1#show spanning-tree vlan 10
<paste recorded command output here>
```

### `show ip interface brief | include Vlan10`

**Expected Results**

* [ ] Output confirms the VLAN 10 SVI is up with address `10.10.10.10`.

```text
A1#show ip interface brief | include Vlan10
<paste recorded command output here>
```

### `show running-config interface vlan 10`

**Expected Results**

* [ ] Output confirms the VLAN 10 SVI configuration.

```text
A1#show running-config interface vlan 10
<paste recorded command output here>
```

### `show running-config | include ^ip default-gateway`

**Expected Results**

* [ ] Output confirms the default gateway is the HSRP VIP `10.10.10.1`.
* [ ] Output includes `ip default-gateway 10.10.10.1`.

```text
A1#show running-config | include ^ip default-gateway
<paste recorded command output here>
```

### `ping 10.10.10.1`

**Expected Results**

* [ ] Output confirms connectivity to the VLAN 10 HSRP virtual gateway.

```text
A1#ping 10.10.10.1
<paste recorded command output here>
```

### `ping 10.10.10.2`

**Expected Results**

* [ ] Output confirms connectivity to D1's VLAN 10 SVI.

```text
A1#ping 10.10.10.2
<paste recorded command output here>
```

### `ping 10.10.10.3`

**Expected Results**

* [ ] Output confirms connectivity to D2's VLAN 10 SVI.

```text
A1#ping 10.10.10.3
<paste recorded command output here>
```

### `ping 10.255.0.4`

**Expected Results**

* [ ] Output confirms routed connectivity to the active firewall `inside` address.

```text
A1#ping 10.255.0.4
<paste recorded command output here>
```

### `ping 203.0.113.1`

**Expected Results**

* [ ] Output confirms connectivity through the firewall to R1.

```text
A1#ping 203.0.113.1
<paste recorded command output here>
```

### `ping 192.0.2.100`

**Expected Results**

* [ ] Output confirms end-to-end connectivity through HSRP, the active firewall, PAT, R1, and the ISP.
* [ ] The ping succeeds end to end through HSRP, the active firewall, PAT, R1, and ISP.

```text
A1#ping 192.0.2.100
<paste recorded command output here>
```

### `traceroute 192.0.2.100`

**Expected Results**

* [ ] Output displays the Layer 3 path toward the simulated external destination.
* [ ] The trace progresses toward the simulated external destination `192.0.2.100`.

```text
A1#traceroute 192.0.2.100
<paste recorded command output here>
```
