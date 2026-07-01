# Lab Reflection — VLAN Trunking

## 1. Purpose and Context

This lab demonstrates VLAN trunking between two Layer 2 switches.

The goal was to carry VLAN 10 and VLAN 20 across an interswitch trunk so same-VLAN hosts could communicate even when they were connected to different switches. C1 and C3 are both in VLAN 10, while C2 and C4 are both in VLAN 20.

This matters because VLANs usually need to span more than one switch in a real access layer. Trunking is what allows that to happen without dedicating a separate physical link to each VLAN.

---

## 2. Design Rationale

The topology is intentionally simple:

* Two switches
* Two VLANs
* One trunk link
* Two clients in each VLAN

That keeps the focus on trunk behavior instead of adding routing, ACLs, or additional switching features too early.

Each switch has one VLAN 10 access port and one VLAN 20 access port. The trunk between S1 and S2 carries VLANs 10 and 20. No Layer 3 routing is configured, so hosts in different VLANs should remain isolated.

This makes the expected behavior clear:

* VLAN 10 should work across the trunk.
* VLAN 20 should work across the trunk.
* VLAN 10 should not communicate with VLAN 20.

---

## 3. Methodology and Testing Approach

Validation focused on proving both trunk function and VLAN isolation.

The switch verification confirmed:

* VLAN 10 and VLAN 20 existed on both switches.
* Access ports were assigned to the correct VLANs.
* Gi0/1 operated as a trunk on both switches.
* VLANs 10 and 20 were allowed across the trunk.
* MAC addresses were learned locally and across the trunk.
* STP showed the VLANs active and forwarding.

Client testing confirmed:

* C1 could reach C3 across the trunk in VLAN 10.
* C2 could reach C4 across the trunk in VLAN 20.
* VLAN 10 clients could not reach VLAN 20 clients.
* VLAN 20 clients could not reach VLAN 10 clients.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

The big lesson from this lab was simple: read the CLI output.

Before the VLAN and trunk configuration was finished, connectivity looked confusing. The issue was that the trunk port had not actually been set to trunk mode. The reason was that 802.1Q encapsulation had not been set yet, so the trunk command was not accepted the way I expected.

That is the kind of mistake that is easy to miss if I type commands and move on without reading the response from the device.

The actual technology is not complicated. Create the VLANs, assign the access ports, configure the trunk, allow the VLANs, and verify the result. But one missed line can break the whole lab.

This also reinforced that diagrams should come before configuration. Having the topology clearly laid out makes it easier to catch interface mistakes and makes the documentation cleaner.

---

## 5. Comparison and Next Steps

This lab builds directly on the VLAN Creation lab.

The VLAN Creation lab showed local VLAN segmentation on one switch. This lab extends those same VLANs across two switches using a trunk. The next natural step is inter-VLAN routing, where Router-on-a-Stick or SVIs are added so selected traffic can move between VLANs.

Useful next steps include:

* Router-on-a-Stick inter-VLAN routing
* SVI-based inter-VLAN routing
* ACLs to control traffic between VLANs
* STP behavior across trunk links
* Native VLAN mismatch testing
* DHCP to reduce manual client addressing

---

## 6. Personal Insights

This lab was not vastly different from the VLAN Creation lab, but it still produced a useful mistake.

That is one of the things I like about networking. It is generally consistent and deterministic if I slow down and read what the device is telling me. When things break, it is often because I missed a detail.

The value of the mistake is that it goes into the “do not do that again” folder. For trunking, that means checking encapsulation, checking the trunk state, checking allowed VLANs, and not assuming the command worked just because I typed it.
