# Lab Reflection — Rapid PVST+

## 1. Purpose and Context

This lab demonstrates how Rapid PVST+ prevents Layer 2 loops and provides faster convergence than traditional STP.

Rapid PVST+ is Cisco’s per-VLAN implementation of RSTP. It allows each VLAN to have its own spanning-tree instance while using faster convergence behavior than legacy STP.

The lab focuses on root bridge election, port roles, port states, manual bridge priority changes, and convergence after a simulated link failure.

This matters in real networks because redundant Layer 2 paths are common, but uncontrolled redundancy can create loops. Rapid PVST+ allows redundant links to exist while keeping the active topology loop-free.

---

## 2. Design Rationale

The topology uses three Layer 2 switches connected in a triangle.

This design intentionally creates redundant Layer 2 paths so Rapid PVST+ can be observed making forwarding and blocking decisions. The triangular topology makes it easy to see which switch becomes the root bridge, which ports become root or designated ports, and which redundant path is placed into a non-forwarding state.

VLAN 10 is used as the test VLAN so the spanning-tree behavior can be validated against a specific VLAN instance.

The lab also includes a manual root bridge change. S1 is configured as the initial root bridge, and S2 is later configured with a lower bridge priority to force a new root bridge election. This makes the lab more useful than simply observing the default election result.

---

## 3. Methodology and Testing Approach

Validation focused on confirming Rapid PVST+ behavior before and after topology changes.

The primary verification command was:

* `show spanning-tree vlan 10`

This command was used to confirm:

* The active spanning-tree protocol
* The elected root bridge
* Local bridge priority and bridge ID
* Root path cost
* Root, designated, and alternate port roles
* Forwarding and blocking states
* Reconvergence after a simulated link failure

The lab was validated in three phases:

* Initial Rapid PVST+ state with S1 as the root bridge
* Root bridge change after lowering S2 bridge priority
* Link failure convergence after shutting down a forwarding path

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

Rapid PVST+ behavior is much easier to understand when the root bridge is intentionally controlled.

Allowing STP to elect a root bridge by default can work, but it makes the topology less predictable. Manually setting bridge priority makes the expected forwarding path clearer and makes verification easier.

The root bridge change was also useful because it showed that spanning-tree behavior is not static. When S2 became the root bridge, the port roles and root paths changed across the topology. This helped reinforce how non-root switches select their root ports based on the best path back to the root bridge.

The simulated link failure showed the value of having a redundant path available. After the failure, Rapid PVST+ reconverged and the previously non-forwarding path became part of the active topology where needed.

The most important lesson from this lab is that STP should not be treated as background magic. The root bridge, port roles, path costs, and blocked links should be intentionally understood and verified.

---

## 5. Comparison and Next Steps

This lab provides a useful baseline for comparing traditional STP behavior against Rapid PVST+.

Useful next comparison points include:

* Traditional STP convergence behavior
* Rapid PVST+ behavior with multiple VLANs
* Root primary and root secondary configuration
* EtherChannel interaction with STP
* PortFast and BPDU Guard behavior on access ports

A future version of this lab could add multiple VLANs with different root bridges to demonstrate per-VLAN spanning-tree behavior more clearly.

---

## 6. Personal Insights

This lab reinforced that Layer 2 redundancy only helps when the control plane is understood.

It is easy to look at a triangle of switches and think only in terms of physical links, but the forwarding topology is determined by spanning tree. The useful skill is not just configuring `spanning-tree mode rapid-pvst`; it is being able to read the output and explain why each port is forwarding or blocking.

The lab also showed why root bridge placement should be intentional. In a real environment, leaving root bridge election to default MAC address behavior can create confusing or undesirable traffic paths.

For this portfolio, this lab is a good Layer 2 foundation because it demonstrates loop prevention, controlled root bridge election, and convergence behavior in a simple but realistic redundant topology.
