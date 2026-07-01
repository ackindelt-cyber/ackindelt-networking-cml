# Lab Reflection — Basic STP

## 1. Purpose and Context

This lab demonstrates how Spanning Tree Protocol prevents Layer 2 loops in a redundant switching topology.

The topology intentionally includes redundant Layer 2 paths. STP keeps the topology loop-free by electing a root bridge, selecting root and designated ports, and placing one redundant path into a blocking state.

This matters in real networks because redundant links are common, but unmanaged Layer 2 redundancy can cause broadcast storms and unstable switching behavior. STP is not exciting, but it is still a foundational safety mechanism in switched networks.

---

## 2. Design Rationale

The lab uses three switches connected in a triangle.

That design is simple, but it creates the exact condition STP is meant to handle: multiple Layer 2 paths between switches. The small topology makes the logic easier to follow because every STP decision can be tied back to a specific physical link.

VLAN 10 is used as the test VLAN so the STP state can be validated against a specific spanning-tree instance.

S1 is configured as the initial root bridge by setting a lower bridge priority. Later, S2 is configured with an even lower priority to force a root bridge change. This makes the lab more useful because it shows both the initial election process and how the topology changes when the root bridge changes.

---

## 3. Methodology and Testing Approach

Validation focused on reading the actual STP state from the switches rather than assuming the topology behaved correctly.

The primary verification command was:

* `show spanning-tree vlan 10`

This command was used to confirm:

* The active STP protocol for VLAN 10
* The elected root bridge
* Local bridge priority and bridge ID
* Root path cost
* Root, designated, and alternate port roles
* Forwarding and blocking states
* Reconvergence after a simulated link failure

The lab was validated in three phases:

* Initial STP state with S1 as the root bridge
* Root bridge change after lowering S2 bridge priority
* Link failure convergence after shutting down a forwarding path

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

The biggest non-STP lesson from this lab was that VLAN persistence in CML was not as straightforward as expected.

VLAN creation did not behave the way I expected after export, which means VLAN-based labs need extra attention when documenting and validating configs. That is annoying, but it is also useful to know because a portfolio lab needs to be reproducible, not just functional once.

The STP-specific lesson is that root bridge placement matters. When S1 was the root bridge, the active and blocked paths followed one pattern. After S2 became the root bridge, the port roles changed. That helped make the STP logic more visible.

This lab also reinforced that STP is a necessary safety mechanism, but good design should avoid casually depending on it. STP should be there to protect the network, not to compensate for unclear Layer 2 design.

---

## 5. Comparison and Next Steps

This lab is a good baseline before comparing STP to Rapid PVST+.

Useful next steps include:

* Comparing classic STP convergence to Rapid PVST+ convergence
* Testing root primary and root secondary behavior
* Adding multiple VLANs with different root bridge placement
* Observing STP behavior when EtherChannel is introduced
* Adding access-port protections such as PortFast and BPDU Guard

The natural follow-up is Rapid PVST+, because it keeps the same basic loop-prevention purpose but improves convergence behavior.

---

## 6. Personal Insights

I like networking because I like logic. Networks are, at their core, logic applied in a very direct way. As the design gets more abstract, it can get more complicated, but these module labs are meant to strip each technology down to its basic behavior.

Most technologies become easier to appreciate when the logic is visible. STP is a little different. It serves a vital purpose, but it is not especially elegant to look at.

That said, this lab was useful because it forced me to slow down and read what the switch was actually doing. The important part was not just configuring STP priority. The important part was being able to explain why a specific switch became the root bridge, why a specific port blocked, and how the topology changed after a failure.