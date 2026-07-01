# Lab Reflection — VLAN Creation

## 1. Purpose and Context

This lab demonstrates basic VLAN creation and access-port assignment on a Layer 2 switch.

The goal was to separate four clients into two VLANs and verify that hosts in the same VLAN could communicate while hosts in different VLANs stayed isolated. VLAN 10 contains C1 and C2. VLAN 20 contains C3 and C4.

This matters because VLANs are one of the basic building blocks of network segmentation. They do not solve every security problem by themselves, but they are a starting point for separating traffic into different broadcast domains.

---

## 2. Design Rationale

The design is intentionally simple:

* One switch
* Four clients
* Two VLANs
* Two clients per VLAN

That keeps the focus on the core behavior of VLANs instead of adding routing, trunks, ACLs, or other technologies too early.

The switch uses access ports only. No trunking or inter-VLAN routing is configured in this lab. That makes the expected result clear: same-VLAN pings should work, and different-VLAN pings should fail.

PortFast and BPDU Guard are enabled on the client-facing ports because these are endpoint access ports. That adds a realistic access-layer protection without changing the main purpose of the lab.

---

## 3. Methodology and Testing Approach

Validation focused on proving both connectivity and isolation.

The switch verification confirmed:

* VLAN 10 and VLAN 20 existed.
* Client-facing ports were assigned to the correct VLANs.
* Access ports had PortFast and BPDU Guard enabled.
* MAC addresses were learned in the expected VLANs.
* STP showed the VLANs active and forwarding.

Client testing confirmed:

* C1 could reach C2 inside VLAN 10.
* C3 could reach C4 inside VLAN 20.
* C1 could not reach VLAN 20 hosts.
* C3 could not reach VLAN 10 hosts.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

This lab is not complicated, but that is the point.

At the basic level, VLAN configuration is straightforward: create the VLAN, name it, assign the access port, and verify the result.

The failed pings between VLANs are just as important as the successful pings inside the same VLAN. Without a router, SVI, or other Layer 3 gateway, VLAN 10 and VLAN 20 should not communicate. That failure is the expected behavior.

This lab is also a useful reminder that “segmentation” has layers. VLANs separate Layer 2 broadcast domains. Later labs add routing and policy controls to decide when those separated networks should or should not communicate.

---

## 5. Comparison and Next Steps

This lab naturally leads into VLAN trunking and inter-VLAN routing.

Useful next steps include:

* Add a second switch and trunk VLANs between switches.
* Add Router-on-a-Stick for inter-VLAN routing.
* Add SVI-based routing on a Layer 3 switch.
* Add ACLs to control traffic between VLANs.
* Compare access-port behavior with trunk-port behavior.
* Expand the design into a more realistic access/distribution topology.

---

## 6. Personal Insights

VLANs make sense to me because they are simple but powerful.

A lot of network security comes back to a few repeated ideas: harden systems, segment traffic, baseline the environment, monitor, scan, patch, and repeat. VLANs are not the whole answer, but they are a big part of that segmentation story.

This lab is basic, but it is foundational. Before building routing, ACLs, HSRP, DHCP relay, or larger enterprise designs, the access-layer segmentation needs to be correct.
