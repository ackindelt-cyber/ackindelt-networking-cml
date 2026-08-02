# Edge Firewall HA Failover Test Output

This file contains recorded failover-test output for the Edge Firewall HA lab.

The tests below confirm manual firewall switchover, automatic failover after monitored-interface and node failures, HSRP and Spanning Tree convergence, continued end-to-end forwarding, and restoration of the preferred baseline state.

**Perform these tests only after the normal verification document passes. Restore the topology to its expected operating state before beginning each test.**

---

## Test 1: Manual Firewall Switchover

**Purpose:** Confirm FW2 can assume the Active role, maintain traffic flow, and return to the original firewall roles.

### Traffic Monitor — `ping 192.0.2.100 repeat 10000`

**Expected Results**

* [ ] A1 continuously reaches `192.0.2.100` before, during, and after the role transitions.
* [ ] The final ping statistics record any packet loss during switchover and restoration.

```text
A1#ping 192.0.2.100 repeat 10000
<paste recorded command output here>
```

### Manual Switchover — `no failover active`

**Expected Results**

* [ ] FW1 relinquishes the Active role.
* [ ] FW2 assumes the Active role.

```text
FW-HA/pri/act#no failover active
<paste recorded command output here>
```

### Post-Switchover Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Standby`.
* [ ] FW2 reports `Secondary/Active`.

```text
FW-HA/pri/stby#show failover
<paste recorded command output here>
```

### Restore Preferred Active Unit — `failover active`

**Expected Results**

* [ ] FW1 resumes the Active role.
* [ ] FW2 returns to the Standby role.

```text
FW-HA/pri/stby#failover active
<paste recorded command output here>
```

### Final Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

---

## Test 2: FW1 Inside-Link Failure

**Purpose:** Confirm failure of FW1’s monitored inside path automatically moves the Active role to FW2.

### Traffic Monitor — `ping 192.0.2.100 repeat 10000`

**Expected Results**

* [ ] A1 continuously tests the full forwarding path during failure and recovery.
* [ ] The final statistics record packet loss and recovery behavior.

```text
A1#ping 192.0.2.100 repeat 10000
<paste recorded command output here>
```

### Induce Inside-Link Failure — D1 `Gi0/0` Shutdown

**Expected Results**

* [ ] D1 `Gi0/0` transitions down.
* [ ] FW1 loses its monitored `inside` path without disabling FW2’s corresponding interface.

```text
D1#configure terminal
interface gigabitethernet0/0
shutdown
end
<paste recorded command output here>
```

### Automatic Failover Check — `show failover`

**Expected Results**

* [ ] FW2 reports `Secondary/Active`.
* [ ] FW1 reports an inside-interface failure and transitions out of the Active role.

```text
<firewall prompt>#show failover
<paste recorded command output here>
```

### Failure-State Check — `show failover state`

**Expected Results**

* [ ] The state table records the automatic transition and the failed-unit condition.

```text
<firewall prompt>#show failover state
<paste recorded command output here>
```

### Restore Inside Link — D1 `Gi0/0` No Shutdown

**Expected Results**

* [ ] D1 `Gi0/0` returns to an operational state.
* [ ] FW1 recovers and progresses to `Standby Ready`.

```text
D1#configure terminal
interface gigabitethernet0/0
no shutdown
end
<paste recorded command output here>
```

### FW1 Recovery Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Standby Ready` before role restoration.
* [ ] FW2 remains `Secondary/Active`.

```text
FW-HA/pri/stby#show failover
<paste recorded command output here>
```

### Restore Preferred Active Unit — `failover active`

**Expected Results**

* [ ] FW1 resumes the Active role after its inside path is healthy.

```text
FW-HA/pri/stby#failover active
<paste recorded command output here>
```

### Final Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

---

## Test 3: FW1 Outside-Link Failure

**Purpose:** Confirm failure of FW1’s monitored outside path automatically moves the Active role to FW2.

### Traffic Monitor — `ping 192.0.2.100 repeat 10000`

**Expected Results**

* [ ] A1 continuously tests end-to-end connectivity during failure and recovery.
* [ ] The final statistics record packet loss and recovery behavior.

```text
A1#ping 192.0.2.100 repeat 10000
<paste recorded command output here>
```

### Induce Outside-Link Failure — OS1 `Gi0/1` Shutdown

**Expected Results**

* [ ] OS1 `Gi0/1` transitions down.
* [ ] FW1 loses its monitored `outside` path while FW2 retains outside connectivity.

```text
OS1#configure terminal
interface gigabitethernet0/1
shutdown
end
<paste recorded command output here>
```

### Automatic Failover Check — `show failover`

**Expected Results**

* [ ] FW2 reports `Secondary/Active`.
* [ ] FW1 reports the failed outside interface.

```text
<firewall prompt>#show failover
<paste recorded command output here>
```

### Failure-State Check — `show failover state`

**Expected Results**

* [ ] The state table confirms the automatic role transition.

```text
<firewall prompt>#show failover state
<paste recorded command output here>
```

### Restore Outside Link — OS1 `Gi0/1` No Shutdown

**Expected Results**

* [ ] OS1 `Gi0/1` returns to an operational state.
* [ ] FW1 recovers and progresses to `Standby Ready`.

```text
OS1#configure terminal
interface gigabitethernet0/1
no shutdown
end
<paste recorded command output here>
```

### FW1 Recovery Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Standby Ready` before role restoration.
* [ ] FW2 remains `Secondary/Active`.

```text
FW-HA/pri/stby#show failover
<paste recorded command output here>
```

### Restore Preferred Active Unit — `failover active`

**Expected Results**

* [ ] FW1 resumes the Active role after its outside path is healthy.

```text
FW-HA/pri/stby#failover active
<paste recorded command output here>
```

### Final Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

---

## Test 4: FW1 Node Failure

**Purpose:** Confirm complete loss of the Active firewall automatically moves the Active role to FW2.

### Traffic Monitor — `ping 192.0.2.100 repeat 10000`

**Expected Results**

* [ ] A1 continuously tests end-to-end connectivity while FW1 stops and restarts.
* [ ] The final statistics record packet loss and recovery behavior.

```text
A1#ping 192.0.2.100 repeat 10000
<paste recorded command output here>
```

### Stop FW1 Node in CML

**Expected Results**

* [ ] FW1 stops completely.
* [ ] FW2 detects loss of the Active peer and assumes the Active role.

```text
CML action: Stop the FW1 node
<record the CML action time and any observations here>
```

### FW2 Role Check — `show failover`

**Expected Results**

* [ ] FW2 reports `Secondary/Active`.
* [ ] The Primary unit is reported failed or unavailable.

```text
FW-HA/sec/act#show failover
<paste recorded command output here>
```

### FW2 State Check — `show failover state`

**Expected Results**

* [ ] FW2 reports a completed transition to the Active state.

```text
FW-HA/sec/act#show failover state
<paste recorded command output here>
```

### Start FW1 Node in CML

**Expected Results**

* [ ] FW1 boots and rejoins the failover pair.
* [ ] FW2 remains Active while FW1 recovers.

```text
CML action: Start the FW1 node
<record the CML action time and any observations here>
```

### FW1 Rejoin Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Standby Ready` after synchronization.
* [ ] FW2 remains `Secondary/Active`.

```text
FW-HA/pri/stby#show failover
<paste recorded command output here>
```

### Restore Preferred Active Unit — `failover active`

**Expected Results**

* [ ] FW1 resumes the Active role.

```text
FW-HA/pri/stby#failover active
<paste recorded command output here>
```

### Final Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

---

## Test 5: D1 Node Failure

**Purpose:** Confirm complete loss of D1 moves traffic onto the surviving FW2–D2 path.

### Traffic Monitor — `ping 192.0.2.100 repeat 10000`

**Expected Results**

* [ ] A1 continuously tests end-to-end connectivity while the preferred distribution switch fails and recovers.
* [ ] The final statistics record packet loss and recovery behavior.

```text
A1#ping 192.0.2.100 repeat 10000
<paste recorded command output here>
```

### Stop D1 Node in CML

**Expected Results**

* [ ] D1 stops completely.
* [ ] FW1 loses its inside path, D2 becomes the surviving distribution switch, and the topology begins convergence.

```text
CML action: Stop the D1 node
<record the CML action time and any observations here>
```

### Firewall Failover Check — `show failover`

**Expected Results**

* [ ] FW2 reports `Secondary/Active` after FW1 loses its inside path.
* [ ] FW1 transitions out of the Active role.

```text
FW-HA/sec/act#show failover
<paste recorded command output here>
```

### Firewall State Check — `show failover state`

**Expected Results**

* [ ] The failover state table confirms the automatic firewall transition.

```text
FW-HA/sec/act#show failover state
<paste recorded command output here>
```

### D2 HSRP Check — `show standby brief`

**Expected Results**

* [ ] D2 reports `Active` for HSRP groups `10` and `99`.

```text
D2#show standby brief
<paste recorded command output here>
```

### A1 Spanning Tree Check — `show spanning-tree vlan 10`

**Expected Results**

* [ ] The D2-facing uplink is forwarding for VLAN `10`.

```text
A1#show spanning-tree vlan 10
<paste recorded command output here>
```

### Start D1 Node in CML

**Expected Results**

* [ ] D1 boots and rejoins the topology.
* [ ] HSRP and Spanning Tree begin returning to their preferred baseline roles.

```text
CML action: Start the D1 node
<record the CML action time and any observations here>
```

### D1 HSRP Restoration Check — `show standby brief`

**Expected Results**

* [ ] D1 reclaims the `Active` role for HSRP groups `10` and `99` through preemption.

```text
D1#show standby brief
<paste recorded command output here>
```

### D1 Spanning Tree Restoration Check — `show spanning-tree vlan 10`

**Expected Results**

* [ ] D1 resumes the preferred VLAN `10` root role.

```text
D1#show spanning-tree vlan 10
<paste recorded command output here>
```

### FW1 Rejoin Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Standby Ready` after its inside link recovers.
* [ ] FW2 remains `Secondary/Active`.

```text
FW-HA/pri/stby#show failover
<paste recorded command output here>
```

### Restore Preferred Active Unit — `failover active`

**Expected Results**

* [ ] FW1 resumes the Active role.

```text
FW-HA/pri/stby#failover active
<paste recorded command output here>
```

### Final Firewall Role Check — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

### Final A1 Spanning Tree Check — `show spanning-tree vlan 10`

**Expected Results**

* [ ] The original forwarding and alternate uplink roles are restored.

```text
A1#show spanning-tree vlan 10
<paste recorded command output here>
```

---

## Final Restoration and Baseline Confirmation

**Purpose:** Confirm the original operating state has been restored after all failover tests.

### Firewall Baseline — `show failover`

**Expected Results**

* [ ] FW1 reports `Primary/Active`.
* [ ] FW2 reports `Secondary/Standby Ready`.

```text
FW-HA/pri/act#show failover
<paste recorded command output here>
```

### HSRP Baseline — `show standby brief`

**Expected Results**

* [ ] D1 reports `Active` and D2 reports `Standby` for VLANs `10` and `99`.

```text
D1#show standby brief
<paste recorded command output here>
```

### Spanning Tree Baseline — `show spanning-tree vlan 10`

**Expected Results**

* [ ] The original VLAN `10` forwarding and alternate uplink roles are restored.

```text
A1#show spanning-tree vlan 10
<paste recorded command output here>
```

### End-to-End Baseline — `ping 192.0.2.100`

**Expected Results**

* [ ] A1 successfully reaches `192.0.2.100` after all failure scenarios are complete.

```text
A1#ping 192.0.2.100
<paste recorded command output here>
```
