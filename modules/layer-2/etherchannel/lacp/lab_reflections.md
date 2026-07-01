# Lab Reflection — LACP EtherChannel

## 1. Purpose and Context

This lab demonstrates how LACP EtherChannel forms logical links from multiple physical switch links.

The goal was to configure LACP between each switch pair, verify that the member links bundled correctly, and confirm that STP saw the resulting port-channels as logical interfaces.

This matters because EtherChannel is common in real switching environments. It gives the network more usable bandwidth between switches and simplifies how STP sees multiple physical links.

---

## 2. Design Rationale

The topology uses three switches connected in a triangle.

Each switch pair has two physical links bundled into a single LACP port-channel:

* S1 to S2 uses Po1.
* S1 to S3 uses Po2.
* S2 to S3 uses Po3.

That design keeps the physical topology simple while still showing the main value of EtherChannel. Instead of STP evaluating every physical link independently, it evaluates the logical port-channel.

LACP active mode is used on both ends of each bundle. That was intentional because active/active negotiation is easy to verify and avoids ambiguity about which side is initiating LACP.

---

## 3. Methodology and Testing Approach

Validation focused on proving that LACP successfully formed the expected port-channels.

The main verification commands were:

* `show etherchannel summary`
* `show lacp neighbor`
* `show interfaces port-channel x`
* `show interfaces gigabitEthernet0/x etherchannel`
* `show spanning-tree`

These commands validated several things:

* The expected port-channels existed.
* The protocol was LACP.
* Member interfaces showed bundled status.
* LACP neighbors were visible.
* Port-channel interfaces were up.
* Aggregated bandwidth appeared on the port-channel interfaces.
* STP saw port-channels as logical links.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

The biggest lab-environment lesson was around CML persistence.

Even if I save the running config inside the node, I still need to extract the configs from CML before shutting the VM down. Otherwise, the node can come back with the default configuration. I had to redo the config on this lab because of that.

That was annoying, but it also made the value of documentation obvious. Most of the time spent on these labs is not the initial configuration. It is verification, cleanup, and making sure the work can be understood later.

The protocol lesson was that LACP feels cleaner than static EtherChannel. With static EtherChannel, the switches will form the bundle if configured, even when mistakes can create risk. With LACP, there is actual negotiation. You can see whether the neighbor is present, whether the member links agree, and whether the port is really bundled.

That makes LACP easier to trust and easier to troubleshoot.

---

## 5. Comparison and Next Steps

This lab is a direct comparison point against static EtherChannel.

Static EtherChannel proves that links can be manually bundled, but LACP adds negotiation and better operational visibility. The `show lacp neighbor` output is especially useful because it proves that the switch is actually exchanging LACP information with the expected partner.

Useful next steps include:

* Compare LACP against PAgP.
* Test active/passive LACP behavior.
* Test what happens when one side is configured incorrectly.
* Add a future member-link failure test.
* Review best-practice placement of trunk configuration on the port-channel interface versus the physical member interfaces.

---

## 6. Personal Insights

It is nice to work in an era where protocols like LACP exist.

It was not that long ago that link bundling had fewer protections and less useful negotiation. LACP makes the behavior easier to reason about because it either forms correctly or gives you something clear to troubleshoot.

This lab also reinforces why verification matters. It is one thing to configure `channel-group` on the correct interfaces. It is another thing to prove that the links are actually bundled, that LACP neighbors exist, that the port-channel is up, and that STP sees the logical link instead of the individual physical links.
