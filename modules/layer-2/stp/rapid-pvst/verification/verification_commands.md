# Rapid PVST+ Verification Output

This file contains recorded verification output for the Rapid PVST+ lab.

The checks below confirm initial root bridge election, Rapid PVST+ port roles and states, a manual root bridge priority change, and convergence after a simulated link failure.

---

## Step 1 — Initial Rapid PVST+ State

### S1

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] S1 is elected as the root bridge for VLAN 10.
* [x] Root ID and Bridge ID match, confirming S1 is the root.
* [x] Bridge priority reflects the configured value: `4096 + VLAN ID 10 = 4106`.
* [x] Gi0/0 and Gi0/1 are Designated and Forwarding.
* [x] No ports are in an Alternate or Blocking state on the root bridge.

```text
S1#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    4106
             Address     5254.0041.03a6
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106   (priority 4096 sys-id-ext 10)
             Address     5254.0041.03a6
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### S2

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root ID matches S1: priority `4106`, address `5254.0041.03a6`.
* [x] S2 is not the root bridge.
* [x] S2 Bridge ID shows default priority plus VLAN ID: `32768 + 10 = 32778`.
* [x] Gi0/0 is the Root port and Forwarding.
* [x] Gi0/1 is Designated and Forwarding for the S2-S3 segment.
* [x] Root path cost is `4`, reflecting a direct GigabitEthernet path to the root bridge.

```text
S2#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    4106
             Address     5254.0041.03a6
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.001f.1b83
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### S3

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root ID matches S1: priority `4106`, address `5254.0041.03a6`.
* [x] S3 is not the root bridge.
* [x] S3 Bridge ID shows default priority plus VLAN ID: `32768 + 10 = 32778`.
* [x] Gi0/0 is the Root port and Forwarding.
* [x] Gi0/1 is Alternate and Blocking to prevent a Layer 2 loop.

```text
S3#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    4106
             Address     5254.0041.03a6
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.00e9.ca57
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Altn BLK 4         128.2    P2p 
```

---

## Step 2 — Root Bridge Change

### S1

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root ID matches S2: priority `10`, address `5254.001f.1b83`.
* [x] S1 is no longer the root bridge.
* [x] S1 Bridge ID remains `4106`.
* [x] Gi0/0 is the Root port and Forwarding.
* [x] Gi0/1 is Designated and Forwarding for the S1-S3 segment.
* [x] Root path cost is `4`, reflecting a direct GigabitEthernet path to S2.

```text
S1#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             Cost        4
             Port        1 (GigabitEthernet0/0)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106   (priority 4096 sys-id-ext 10)
             Address     5254.0041.03a6
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Root FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### S2

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] S2 is elected as the root bridge for VLAN 10.
* [x] Root ID and Bridge ID match, confirming S2 is the root.
* [x] Bridge priority reflects the configured value: `0 + VLAN ID 10 = 10`.
* [x] Gi0/0 and Gi0/1 are Designated and Forwarding.
* [x] No ports are in an Alternate or Blocking state on the root bridge.

```text
S2#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    10     (priority 0 sys-id-ext 10)
             Address     5254.001f.1b83
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### S3

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root ID matches S2: priority `10`, address `5254.001f.1b83`.
* [x] S3 is not the root bridge.
* [x] S3 Bridge ID shows default priority plus VLAN ID: `32768 + 10 = 32778`.
* [x] Gi0/1 is the Root port and Forwarding.
* [x] Gi0/0 is Alternate and Blocking to prevent a loop on the S1-S3 link.

```text
S3#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.00e9.ca57
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Altn BLK 4         128.1    P2p 
Gi0/1               Root FWD 4         128.2    P2p 
```

---

## Step 3 — Link Failure Convergence

### S1

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root bridge remains S2: priority `10`, address `5254.001f.1b83`.
* [x] S1 is not the root bridge.
* [x] S1 Bridge ID remains `4106`.
* [x] Gi0/1 is the Root port and Forwarding after reconvergence.
* [x] Root path cost is `8`, reflecting the indirect path through S3.
* [x] No ports are shown in an Alternate or Blocking state on S1.

```text
S1#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             Cost        8
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    4106   (priority 4096 sys-id-ext 10)
             Address     5254.0041.03a6
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/1               Root FWD 4         128.2    P2p 
```

### S2

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] S2 remains the root bridge for VLAN 10.
* [x] Root ID and Bridge ID match, confirming S2 is still the root.
* [x] Gi0/0 and Gi0/1 remain Designated and Forwarding.
* [x] No topology instability is shown after the simulated link failure.

```text
S2#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             This bridge is the root
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    10     (priority 0 sys-id-ext 10)
             Address     5254.001f.1b83
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Desg FWD 4         128.2    P2p 
```

### S3

#### `show spanning-tree vlan 10`

**Expected Results**

* [x] VLAN 10 spanning tree is enabled and operating in RSTP mode.
* [x] Root bridge remains S2: priority `10`, address `5254.001f.1b83`.
* [x] S3 is not the root bridge.
* [x] S3 Bridge ID shows default priority plus VLAN ID: `32768 + 10 = 32778`.
* [x] Gi0/1 remains the Root port and Forwarding.
* [x] Gi0/0 transitions from Alternate to Designated and Forwarding after reconvergence.

```text
S3#show spanning-tree vlan 10

VLAN0010
  Spanning tree enabled protocol rstp
  Root ID    Priority    10
             Address     5254.001f.1b83
             Cost        4
             Port        2 (GigabitEthernet0/1)
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec

  Bridge ID  Priority    32778  (priority 32768 sys-id-ext 10)
             Address     5254.00e9.ca57
             Hello Time   2 sec  Max Age 20 sec  Forward Delay 15 sec
             Aging Time  300 sec

Interface           Role Sts Cost      Prio.Nbr Type
------------------- ---- --- --------- -------- --------------------------------
Gi0/0               Desg FWD 4         128.1    P2p 
Gi0/1               Root FWD 4         128.2    P2p 
```
