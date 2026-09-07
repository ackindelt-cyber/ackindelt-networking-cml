# L2-IS-IS Failure Testing Output

This file contains recorded failure-testing output for the L2-IS-IS lab.

The tests below validate IS-IS behavior during link failure, missing interface participation, routing-level mismatch, missing prefix advertisement, and missing NET configuration. Each test records the observed failure state, verifies the expected control-plane and forwarding behavior, restores the intended configuration, and confirms recovery.

---

## R1

### Downed Interface

This test simulates failure of the direct R1-R2 transit link while preserving the alternate path through R3.

#### Create Failure State

```text
R1#configure terminal
R1(config)#interface GigabitEthernet0/0
R1(config-if)#shutdown
R1(config-if)#end

<paste recorded command output here>
```

#### Expected Results
* [ ] `GigabitEthernet0/0` is administratively down.
* [ ] The direct R1-R2 Level 2 adjacency is removed.
* [ ] The R1-R3 Level 2 adjacency remains `UP`.
* [ ] R1 recalculates a route to `10.255.0.2/32` through R3.
* [ ] R2's loopback remains reachable after IS-IS reconvergence.
* [ ] Traceroute confirms traffic uses the alternate R1-R3-R2 path.

#### Verification

##### `show ip interface brief`

```text
R1#show ip interface brief

<paste recorded command output here>
```

##### `show isis neighbors`

```text
R1#show isis neighbors

<paste recorded command output here>
```

##### `show ip route 10.255.0.2`

```text
R1#show ip route 10.255.0.2

<paste recorded command output here>
```

##### `ping 10.255.0.2 source Loopback0`

```text
R1#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```

##### `traceroute 10.255.0.2 source Loopback0`

```text
R1#traceroute 10.255.0.2 source Loopback0

<paste recorded command output here>
```

#### Restore Intended State

```text
R1#configure terminal
R1(config)#interface GigabitEthernet0/0
R1(config-if)#no shutdown
R1(config-if)#end
R1#write memory

<paste recorded command output here>
```

---

### Missing IS-IS Interface

This test removes IS-IS participation from the R1-R2 transit interface while leaving the Layer 3 interface operational.

#### Create Failure State

```text
R1#configure terminal
R1(config)#interface GigabitEthernet0/0
R1(config-if)#no ip router isis
R1(config-if)#end

<paste recorded command output here>
```

#### Expected Results

* [ ] `GigabitEthernet0/0` remains `up/up`.

* [ ] IS-IS is no longer participating on `GigabitEthernet0/0`.

* [ ] The R1-R2 Level 2 adjacency is removed.

* [ ] The R1-R3 Level 2 adjacency remains `UP`.

* [ ] R1 recalculates the route to `10.255.0.2/32` through R3.

* [ ] R2's loopback remains reachable through the alternate path.

#### Verification

##### `show ip interface brief`

```text
R1#show ip interface brief

<paste recorded command output here>
```

##### `show clns interface GigabitEthernet0/0`

```text
R1#show clns interface GigabitEthernet0/0

<paste recorded command output here>
```

##### `show isis neighbors`

```text
R1#show isis neighbors

<paste recorded command output here>
```

##### `show ip route 10.255.0.2`

```text
R1#show ip route 10.255.0.2

<paste recorded command output here>
```

##### `ping 10.255.0.2 source Loopback0`

```text
R1#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```

##### `traceroute 10.255.0.2 source Loopback0`

```text
R1#traceroute 10.255.0.2 source Loopback0

<paste recorded command output here>
```

#### Restore Intended State

```text
R1#configure terminal
R1(config)#interface GigabitEthernet0/0
R1(config-if)#ip router isis
R1(config-if)#end
R1#write memory

<paste recorded command output here>
```

---

### IS-IS Level Mismatch

This test changes R1 from Level 2-only to Level 1-only operation while R2 and R3 remain Level 2-only.

#### Create Failure State

```text
R1#configure terminal
R1(config)#router isis
R1(config-router)#is-type level-1
R1(config-router)#end

<paste recorded command output here>
```

#### Expected Results

* [ ] R1 reports Level 1-only IS-IS operation.

* [ ] The R1-R2 Level 2 adjacency is removed.

* [ ] The R1-R3 Level 2 adjacency is removed.

* [ ] Remote Level 2 IS-IS routes are no longer installed on R1.

* [ ] R2's loopback `10.255.0.2/32` is no longer reachable through IS-IS.

* [ ] R3's loopback `10.255.0.3/32` is no longer reachable through IS-IS.

#### Verification

##### `show clns protocol`

```text
R1#show clns protocol

<paste recorded command output here>
```

##### `show isis neighbors`

```text
R1#show isis neighbors

<paste recorded command output here>
```

##### `show ip route isis`

```text
R1#show ip route isis

<paste recorded command output here>
```

##### `ping 10.255.0.2 source Loopback0`

```text
R1#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```

##### `ping 10.255.0.3 source Loopback0`

```text
R1#ping 10.255.0.3 source Loopback0

<paste recorded command output here>
```

#### Restore Intended State

```text
R1#configure terminal
R1(config)#router isis
R1(config-router)#is-type level-2-only
R1(config-router)#end
R1#write memory

<paste recorded command output here>
```

---

### Missing Loopback Prefix

This test removes Loopback0 from IS-IS while leaving the interface itself operational and preserving all transit adjacencies.

#### Create Failure State

```text
R1#configure terminal
R1(config)#interface Loopback0
R1(config-if)#no ip router isis
R1(config-if)#end

<paste recorded command output here>
```

#### Expected Results

* [ ] R1's Level 2 adjacencies with R2 and R3 remain `UP`.

* [ ] R1's Loopback0 remains operational.

* [ ] R1's LSP no longer advertises `10.255.0.1/32`.

* [ ] R2 no longer installs `10.255.0.1/32` as an IS-IS route.

* [ ] R2 can no longer reach R1's loopback through IS-IS.

#### Verification

##### `show isis neighbors`

```text
R1#show isis neighbors

<paste recorded command output here>
```

##### `show isis database detail`

```text
R1#show isis database detail

<paste recorded command output here>
```

##### `show ip route 10.255.0.1`

Run this check on R2.

```text
R2#show ip route 10.255.0.1

<paste recorded command output here>
```

##### `ping 10.255.0.1 source Loopback0`

Run this check on R2.

```text
R2#ping 10.255.0.1 source Loopback0

<paste recorded command output here>
```

#### Restore Intended State

```text
R1#configure terminal
R1(config)#interface Loopback0
R1(config-if)#ip router isis
R1(config-if)#end
R1#write memory

<paste recorded command output here>
```

---

### Missing IS-IS NET

This test removes R1's configured NET, removing the IS-IS identity required for normal participation in the Level 2 domain.

#### Create Failure State

```text
R1#configure terminal
R1(config)#router isis
R1(config-router)#no net 49.0001.0000.0000.0001.00
R1(config-router)#end

<paste recorded command output here>
```

#### Expected Results

* [ ] R1 no longer has the intended NET `49.0001.0000.0000.0001.00`.

* [ ] R1 can no longer maintain its expected Level 2 adjacencies.

* [ ] The missing NET affects normal Level 2 LSDB participation.

* [ ] Remote IS-IS routes are no longer installed on R1.

* [ ] R2's loopback `10.255.0.2/32` is no longer reachable through IS-IS.

* [ ] R3's loopback `10.255.0.3/32` is no longer reachable through IS-IS.

#### Verification

##### `show clns protocol`

```text
R1#show clns protocol

<paste recorded command output here>
```

##### `show isis neighbors`

```text
R1#show isis neighbors

<paste recorded command output here>
```

##### `show isis database`

```text
R1#show isis database

<paste recorded command output here>
```

##### `show ip route isis`

```text
R1#show ip route isis

<paste recorded command output here>
```

##### `ping 10.255.0.2 source Loopback0`

```text
R1#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```

##### `ping 10.255.0.3 source Loopback0`

```text
R1#ping 10.255.0.3 source Loopback0

<paste recorded command output here>
```

#### Restore Intended State

```text
R1#configure terminal
R1(config)#router isis
R1(config-router)#net 49.0001.0000.0000.0001.00
R1(config-router)#end
R1#write memory

<paste recorded command output here>
```