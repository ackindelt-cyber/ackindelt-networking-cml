# L2-IS-IS Verification Output

This file contains recorded verification output for the L2-IS-IS lab.

The checks below confirm the configured IS-IS process and NET identity, Level 2-only operation, interface participation, point-to-point circuit behavior, neighbor adjacencies, link-state database synchronization, SPF path calculation, IPv4 route installation, and end-to-end loopback reachability.

---

## R1

### `show clns protocol`

**Expected Results**
* [ ] The IS-IS process is active on R1.
* [ ] The configured NET is `49.0001.0000.0000.0001.00`.
* [ ] R1 is operating as a Level 2-only IS-IS router.

```text
R1#show clns protocol

<paste recorded command output here>
```

### `show clns interface`

**Expected Results**
* [ ] `GigabitEthernet0/0` is participating in IS-IS on the R1-R2 transit link.
* [ ] `GigabitEthernet0/1` is participating in IS-IS on the R1-R3 transit link.
* [ ] The transit interfaces use point-to-point IS-IS circuit behavior.

```text
R1#show clns interface

<paste recorded command output here>
```

### `show isis neighbors`

**Expected Results**
* [ ] R1 has a Level 2 adjacency with R2.
* [ ] R1 has a Level 2 adjacency with R3.
* [ ] Both expected adjacencies are in the `UP` state.

```text
R1#show isis neighbors

<paste recorded command output here>
```

### `show isis database`

**Expected Results**
* [ ] The Level 2 link-state database contains an LSP originated by R1.
* [ ] The Level 2 link-state database contains LSPs originated by R2 and R3.
* [ ] R1 has learned the expected three-router Level 2 topology.

```text
R1#show isis database

<paste recorded command output here>
```

### `show isis topology`

**Expected Results**
* [ ] R1 has calculated Level 2 paths to R2 and R3.
* [ ] Both directly connected IS-IS neighbors are reachable through the expected transit interfaces.
* [ ] The calculated topology is consistent with the three-router triangle design.

```text
R1#show isis topology

<paste recorded command output here>
```

### `show ip route isis`

**Expected Results**
* [ ] R1 has IS-IS-learned routes for remote prefixes advertised by R2.
* [ ] R1 has IS-IS-learned routes for remote prefixes advertised by R3.
* [ ] Remote loopbacks `10.255.0.2/32` and `10.255.0.3/32` are installed as Level 2 IS-IS routes.

```text
R1#show ip route isis

<paste recorded command output here>
```

### `ping 10.255.0.2 source Loopback0`

**Expected Results**
* [ ] R1 can reach R2's loopback at `10.255.0.2`.
* [ ] The ping is sourced from R1's Loopback0 address `10.255.0.1`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R1#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```

### `ping 10.255.0.3 source Loopback0`

**Expected Results**
* [ ] R1 can reach R3's loopback at `10.255.0.3`.
* [ ] The ping is sourced from R1's Loopback0 address `10.255.0.1`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R1#ping 10.255.0.3 source Loopback0

<paste recorded command output here>
```

---

## R2

### `show clns protocol`

**Expected Results**
* [ ] The IS-IS process is active on R2.
* [ ] The configured NET is `49.0001.0000.0000.0002.00`.
* [ ] R2 is operating as a Level 2-only IS-IS router.

```text
R2#show clns protocol

<paste recorded command output here>
```

### `show clns interface`

**Expected Results**
* [ ] `GigabitEthernet0/0` is participating in IS-IS on the R2-R1 transit link.
* [ ] `GigabitEthernet0/1` is participating in IS-IS on the R2-R3 transit link.
* [ ] The transit interfaces use point-to-point IS-IS circuit behavior.

```text
R2#show clns interface

<paste recorded command output here>
```

### `show isis neighbors`

**Expected Results**
* [ ] R2 has a Level 2 adjacency with R1.
* [ ] R2 has a Level 2 adjacency with R3.
* [ ] Both expected adjacencies are in the `UP` state.

```text
R2#show isis neighbors

<paste recorded command output here>
```

### `show isis database`

**Expected Results**
* [ ] The Level 2 link-state database contains an LSP originated by R2.
* [ ] The Level 2 link-state database contains LSPs originated by R1 and R3.
* [ ] R2 has learned the expected three-router Level 2 topology.

```text
R2#show isis database

<paste recorded command output here>
```

### `show isis topology`

**Expected Results**
* [ ] R2 has calculated Level 2 paths to R1 and R3.
* [ ] Both directly connected IS-IS neighbors are reachable through the expected transit interfaces.
* [ ] The calculated topology is consistent with the three-router triangle design.

```text
R2#show isis topology

<paste recorded command output here>
```

### `show ip route isis`

**Expected Results**
* [ ] R2 has IS-IS-learned routes for remote prefixes advertised by R1.
* [ ] R2 has IS-IS-learned routes for remote prefixes advertised by R3.
* [ ] Remote loopbacks `10.255.0.1/32` and `10.255.0.3/32` are installed as Level 2 IS-IS routes.

```text
R2#show ip route isis

<paste recorded command output here>
```

### `ping 10.255.0.1 source Loopback0`

**Expected Results**
* [ ] R2 can reach R1's loopback at `10.255.0.1`.
* [ ] The ping is sourced from R2's Loopback0 address `10.255.0.2`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R2#ping 10.255.0.1 source Loopback0

<paste recorded command output here>
```

### `ping 10.255.0.3 source Loopback0`

**Expected Results**
* [ ] R2 can reach R3's loopback at `10.255.0.3`.
* [ ] The ping is sourced from R2's Loopback0 address `10.255.0.2`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R2#ping 10.255.0.3 source Loopback0

<paste recorded command output here>
```

---

## R3

### `show clns protocol`

**Expected Results**
* [ ] The IS-IS process is active on R3.
* [ ] The configured NET is `49.0001.0000.0000.0003.00`.
* [ ] R3 is operating as a Level 2-only IS-IS router.

```text
R3#show clns protocol

<paste recorded command output here>
```

### `show clns interface`

**Expected Results**
* [ ] `GigabitEthernet0/0` is participating in IS-IS on the R3-R1 transit link.
* [ ] `GigabitEthernet0/1` is participating in IS-IS on the R3-R2 transit link.
* [ ] The transit interfaces use point-to-point IS-IS circuit behavior.

```text
R3#show clns interface

<paste recorded command output here>
```

### `show isis neighbors`

**Expected Results**
* [ ] R3 has a Level 2 adjacency with R1.
* [ ] R3 has a Level 2 adjacency with R2.
* [ ] Both expected adjacencies are in the `UP` state.

```text
R3#show isis neighbors

<paste recorded command output here>
```

### `show isis database`

**Expected Results**
* [ ] The Level 2 link-state database contains an LSP originated by R3.
* [ ] The Level 2 link-state database contains LSPs originated by R1 and R2.
* [ ] R3 has learned the expected three-router Level 2 topology.

```text
R3#show isis database

<paste recorded command output here>
```

### `show isis topology`

**Expected Results**
* [ ] R3 has calculated Level 2 paths to R1 and R2.
* [ ] Both directly connected IS-IS neighbors are reachable through the expected transit interfaces.
* [ ] The calculated topology is consistent with the three-router triangle design.

```text
R3#show isis topology

<paste recorded command output here>
```

### `show ip route isis`

**Expected Results**
* [ ] R3 has IS-IS-learned routes for remote prefixes advertised by R1.
* [ ] R3 has IS-IS-learned routes for remote prefixes advertised by R2.
* [ ] Remote loopbacks `10.255.0.1/32` and `10.255.0.2/32` are installed as Level 2 IS-IS routes.

```text
R3#show ip route isis

<paste recorded command output here>
```

### `ping 10.255.0.1 source Loopback0`

**Expected Results**
* [ ] R3 can reach R1's loopback at `10.255.0.1`.
* [ ] The ping is sourced from R3's Loopback0 address `10.255.0.3`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R3#ping 10.255.0.1 source Loopback0

<paste recorded command output here>
```

### `ping 10.255.0.2 source Loopback0`

**Expected Results**
* [ ] R3 can reach R2's loopback at `10.255.0.2`.
* [ ] The ping is sourced from R3's Loopback0 address `10.255.0.3`.
* [ ] The ping completes successfully with no expected packet loss.

```text
R3#ping 10.255.0.2 source Loopback0

<paste recorded command output here>
```