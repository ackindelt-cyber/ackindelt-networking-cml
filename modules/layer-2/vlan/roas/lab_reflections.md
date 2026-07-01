# Lab Reflection — Router-on-a-Stick

## 1. Purpose and Context

This lab demonstrates Router-on-a-Stick inter-VLAN routing.

The goal was to use one router interface, multiple subinterfaces, and 802.1Q encapsulation to route between VLAN 10 and VLAN 20. The switches provide the Layer 2 VLAN and trunking behavior, while R1 provides the Layer 3 default gateway for each VLAN.

This matters because VLANs separate broadcast domains by default. If hosts in different VLANs need to communicate, there has to be a Layer 3 device routing between those networks. Router-on-a-Stick is a simple way to demonstrate that relationship clearly.

---

## 2. Design Rationale

The design uses two switches, one router, and four clients.

VLAN 10 uses the `192.168.10.0/24` network, and VLAN 20 uses the `192.168.20.0/24` network. R1 has one subinterface per VLAN:

* `Gi0/0.10` provides the default gateway for VLAN 10.
* `Gi0/0.20` provides the default gateway for VLAN 20.

S1 connects directly to R1 with a trunk link. S1 also connects to S2 with a trunk link so both VLANs can exist across both switches.

Client placement is split across the two switches so the lab validates more than just local switching. C1 and C3 are both in VLAN 10, while C2 and C4 are both in VLAN 20. That allows the lab to test same-VLAN connectivity, inter-VLAN routing, and trunk behavior between switches.

Access ports use PortFast and BPDU Guard because the connected clients are endpoints. That keeps the access-port behavior realistic without making STP the main focus of the lab.

---

## 3. Methodology and Testing Approach

Validation focused on proving the full path from Layer 2 VLAN membership to Layer 3 routing.

On R1, verification confirmed:

* The physical interface was up.
* Both subinterfaces were up/up.
* Each subinterface had the correct IP address.
* 802.1Q encapsulation matched the intended VLAN.
* Connected routes existed for both VLAN networks.
* ARP entries existed for clients in both VLANs.

On S1 and S2, verification confirmed:

* VLAN 10 and VLAN 20 existed.
* Access ports were assigned to the correct VLANs.
* Trunk ports allowed VLANs 10 and 20.
* STP showed the VLANs active and forwarding.
* Switchport output matched the intended access and trunk roles.

Client testing confirmed that C1 could reach a same-VLAN host and hosts in the other VLAN. Packet captures were also collected for selected ICMP flows.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

This lab reinforces that Router-on-a-Stick is simple conceptually, but only if every layer is correct.

The router subinterfaces can be configured correctly, but hosts still will not communicate if the switch trunk does not allow the VLANs. The VLANs can exist on the switches, but inter-VLAN routing still will not work unless the router subinterfaces have the right encapsulation and gateway addresses.

The verification output made that dependency clear. `show ip route` proved that R1 had connected routes for both VLANs. `show interfaces switchport` proved that the switchports were operating in the expected access or trunk modes. The client pings proved that the full path worked.

This lab also helped reinforce that VLANs do not automatically block Layer 3 communication once routing exists. After Router-on-a-Stick is configured, VLAN 10 and VLAN 20 can communicate through R1. If the design requires restriction between VLANs, ACLs or another policy mechanism need to be added later.

---

## 5. Comparison and Next Steps

Router-on-a-Stick is a good baseline before moving into Layer 3 switching.

A natural next comparison would be to replace R1 with a multilayer switch and use SVIs for inter-VLAN routing. That would show the difference between routing through router subinterfaces and routing directly on a Layer 3 switch.

Useful next steps include:

* Add ACLs to restrict traffic between VLANs.
* Compare Router-on-a-Stick against SVI-based inter-VLAN routing.
* Add a management VLAN.
* Add DHCP relay for client addressing.
* Add more VLANs and verify scaling behavior.
* Capture and review 802.1Q-tagged traffic on trunk links.

---

## 6. Personal Insights

This lab is a good reminder that networking problems are usually solved by checking the path one layer at a time.

For Router-on-a-Stick, the logic is straightforward: the host needs the right IP and gateway, the access port needs the right VLAN, the trunk needs to carry that VLAN, and the router subinterface needs the right encapsulation and address.