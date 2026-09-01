# BGP Fundamentals Verification Output

This file contains recorded verification output for the BGP Fundamentals lab.

The checks below confirm BGP neighbor establishment, prefix propagation between autonomous systems, BGP best-path selection, routing-table installation, and bidirectional reachability between the Talos Solutions and external service test prefixes.

---

## TSE1

### `show ip bgp summary`

**Expected Results**

* [ ] Local BGP AS is `65001`.

* [ ] Neighbor `10.0.10.2` is associated with remote AS `65010`.

* [ ] Neighbor `10.0.50.2` is associated with remote AS `65001`.

* [ ] Both BGP neighbor sessions are established and `State/PfxRcd` displays received prefix counts.

```text
TSE1#show ip bgp summary

<paste recorded command output here>
```

### `show ip bgp`

**Expected Results**

* [ ] The locally originated Talos Solutions prefix `192.0.2.0/24` is present in the BGP table.

* [ ] The external service prefix `203.0.113.0/24` is present in the BGP table.

* [ ] Multiple valid paths to `203.0.113.0/24` are visible through the eBGP and iBGP relationships.

* [ ] One path to `203.0.113.0/24` is identified as the selected best path.

```text
TSE1#show ip bgp

<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] A route to `203.0.113.0/24` is present in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed next hop corresponds to the BGP-selected best path.

```text
TSE1#show ip route 203.0.113.0

<paste recorded command output here>
```

### `ping 203.0.113.1 source loopback0`

**Expected Results**

* [ ] ICMP traffic is sourced from TSE1 Loopback0 address `192.0.2.1`.

* [ ] The destination `203.0.113.1` responds successfully.

* [ ] Successful replies confirm bidirectional reachability between the Talos Solutions and external service BGP prefixes.

```text
TSE1#ping 203.0.113.1 source loopback0

<paste recorded command output here>
```

---

## TSE2

### `show ip bgp summary`

**Expected Results**

* [ ] Local BGP AS is `65001`.

* [ ] Neighbor `10.0.20.2` is associated with remote AS `65020`.

* [ ] Neighbor `10.0.50.1` is associated with remote AS `65001`.

* [ ] Both BGP neighbor sessions are established and `State/PfxRcd` displays received prefix counts.

```text
TSE2#show ip bgp summary

<paste recorded command output here>
```

### `show ip bgp`

**Expected Results**

* [ ] The Talos Solutions prefix `192.0.2.0/24` is present in the BGP table.

* [ ] The external service prefix `203.0.113.0/24` is present in the BGP table.

* [ ] Multiple valid paths to `203.0.113.0/24` are visible through the eBGP and iBGP relationships.

* [ ] One path to `203.0.113.0/24` is identified as the selected best path.

```text
TSE2#show ip bgp

<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] A route to `203.0.113.0/24` is present in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed next hop corresponds to the BGP-selected best path.

```text
TSE2#show ip route 203.0.113.0

<paste recorded command output here>
```

### `show ip route 192.0.2.0`

**Expected Results**

* [ ] A route to `192.0.2.0/24` is present in the routing table.

* [ ] The prefix is learned from TSE1 through iBGP.

* [ ] The installed next hop is reachable through the internal TSE1-TSE2 transit network.

```text
TSE2#show ip route 192.0.2.0

<paste recorded command output here>
```

---

## ISPA

### `show ip bgp summary`

**Expected Results**

* [ ] Local BGP AS is `65010`.

* [ ] Neighbor `10.0.10.1` is associated with remote AS `65001`.

* [ ] Neighbor `10.0.30.2` is associated with remote AS `65030`.

* [ ] Both eBGP neighbor sessions are established and `State/PfxRcd` displays received prefix counts.

```text
ISPA#show ip bgp summary

<paste recorded command output here>
```

### `show ip bgp`

**Expected Results**

* [ ] Talos Solutions prefix `192.0.2.0/24` is present in the BGP table.

* [ ] External service prefix `203.0.113.0/24` is present in the BGP table.

* [ ] The displayed AS paths reflect propagation between the participating autonomous systems.

* [ ] A best path is selected for each learned prefix.

```text
ISPA#show ip bgp

<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] A route to `203.0.113.0/24` is installed in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed route forwards toward EXS1.

```text
ISPA#show ip route 203.0.113.0

<paste recorded command output here>
```

### `show ip route 192.0.2.0`

**Expected Results**

* [ ] A route to `192.0.2.0/24` is installed in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed route provides reachability toward the Talos Solutions test prefix.

```text
ISPA#show ip route 192.0.2.0

<paste recorded command output here>
```

---

## ISPB

### `show ip bgp summary`

**Expected Results**

* [ ] Local BGP AS is `65020`.

* [ ] Neighbor `10.0.20.1` is associated with remote AS `65001`.

* [ ] Neighbor `10.0.40.2` is associated with remote AS `65030`.

* [ ] Both eBGP neighbor sessions are established and `State/PfxRcd` displays received prefix counts.

```text
ISPB#show ip bgp summary

<paste recorded command output here>
```

### `show ip bgp`

**Expected Results**

* [ ] Talos Solutions prefix `192.0.2.0/24` is present in the BGP table.

* [ ] External service prefix `203.0.113.0/24` is present in the BGP table.

* [ ] The displayed AS paths reflect propagation between the participating autonomous systems.

* [ ] A best path is selected for each learned prefix.

```text
ISPB#show ip bgp

<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] A route to `203.0.113.0/24` is installed in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed route forwards toward EXS1.

```text
ISPB#show ip route 203.0.113.0

<paste recorded command output here>
```

### `show ip route 192.0.2.0`

**Expected Results**

* [ ] A route to `192.0.2.0/24` is installed in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed route provides reachability toward the Talos Solutions test prefix.

```text
ISPB#show ip route 192.0.2.0

<paste recorded command output here>
```

---

## EXS1

### `show ip bgp summary`

**Expected Results**

* [ ] Local BGP AS is `65030`.

* [ ] Neighbor `10.0.30.1` is associated with remote AS `65010`.

* [ ] Neighbor `10.0.40.1` is associated with remote AS `65020`.

* [ ] Both eBGP neighbor sessions are established and `State/PfxRcd` displays received prefix counts.

```text
EXS1#show ip bgp summary

<paste recorded command output here>
```

### `show ip bgp`

**Expected Results**

* [ ] Locally originated external service prefix `203.0.113.0/24` is present in the BGP table.

* [ ] Talos Solutions prefix `192.0.2.0/24` is learned through BGP.

* [ ] Multiple BGP paths to `192.0.2.0/24` may be visible through ISPA and ISPB.

* [ ] One path to `192.0.2.0/24` is identified as the selected best path.

```text
EXS1#show ip bgp

<paste recorded command output here>
```

### `show ip route 192.0.2.0`

**Expected Results**

* [ ] A route to `192.0.2.0/24` is installed in the routing table.

* [ ] The route is learned through BGP.

* [ ] The installed next hop corresponds to the BGP-selected best path toward Talos Solutions.

```text
EXS1#show ip route 192.0.2.0

<paste recorded command output here>
```

### `show ip route 203.0.113.0`

**Expected Results**

* [ ] Prefix `203.0.113.0/24` is present in the local routing table.

* [ ] The route is directly connected through `Loopback0`.

* [ ] The exact connected prefix satisfies the BGP `network 203.0.113.0 mask 255.255.255.0` origination requirement.

```text
EXS1#show ip route 203.0.113.0

<paste recorded command output here>
```