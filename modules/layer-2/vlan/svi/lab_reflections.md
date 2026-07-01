# Lab Reflection — SVI Inter-VLAN Routing

## 1. Purpose and Context

This lab demonstrates inter-VLAN routing using switched virtual interfaces on a Layer 3 switch.

The goal was to build a small distribution-to-access topology where D1 provides the default gateways for multiple VLANs. Instead of using an external router and subinterfaces like Router-on-a-Stick, the Layer 3 switch routes directly between VLAN interfaces.

This matters because SVI-based routing is closer to how many enterprise campus networks handle inter-VLAN routing. It keeps routing close to the switching layer and avoids sending all inter-VLAN traffic through a single external router interface.

---

## 2. Design Rationale

This topology is more complex than the Router-on-a-Stick lab and is closer to a distribution/access design.

D1 acts as the distribution switch and has Layer 3 routing enabled. Four access switches connect back to D1 using trunk links. Each access switch has one client-facing access port assigned to a different VLAN:

* A1 provides access for VLAN 10.
* A2 provides access for VLAN 20.
* A3 provides access for VLAN 30.
* A4 provides access for VLAN 40.

D1 has one SVI per user VLAN. Those SVIs provide the default gateway addresses for the clients and create connected routes in the routing table.

VLAN 99 is used as the native VLAN on trunk links. That made the trunk configuration more realistic, but it also created a useful troubleshooting lesson when a mismatch caused STP to filter a port.

---

## 3. Methodology and Testing Approach

Validation focused on whether the mechanisms required for SVI routing existed and whether traffic routed correctly.

The lab needed to prove several things:

* VLANs existed on the required switches.
* Trunks were carrying VLANs 10, 20, 30, 40, and 99.
* Native VLAN 99 matched across trunks.
* SVIs existed and were up/up on D1.
* `ip routing` was enabled on D1.
* Connected routes existed for each SVI network.
* Clients could reach their default gateways.
* Clients could reach hosts in other VLANs.
* Traceroute showed traffic using the correct SVI gateway as the first hop.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

Functionally, this lab is similar to Router-on-a-Stick, but the routing lives in a better place.

With Router-on-a-Stick, the router provides the inter-VLAN routing through subinterfaces. With SVIs, the Layer 3 switch owns the gateways directly. The logic is similar, but the design feels cleaner for an enterprise switching environment.

The most useful mistake was the native VLAN mismatch. Changing the native VLAN to 99 made sense, but it also meant every trunk had to match. When one side did not match, STP filtered the port. That was a good cause-and-effect reminder: trunk consistency matters, and STP will react when it sees something unsafe.

---

## 5. Comparison and Next Steps

The next version could build on this design by adding services and resiliency features, such as:

* DHCP relay
* DNS forwarding
* HSRP or another first-hop redundancy protocol
* ACLs between VLANs
* EtherChannel uplinks
* Better STP root bridge placement
* A management VLAN
* Centralized verification and troubleshooting runbooks

This lab is also a good comparison point against Router-on-a-Stick. Both approaches route between VLANs, but SVI-based routing is the more scalable and realistic campus design.

---

## 6. Personal Insights

This was the first lab where I used a runbook as part of the build process instead of just creating one afterward.

That made a noticeable difference. I normally try to build the first version of a technology from scratch so I understand it, then write a runbook afterward if the process is worth repeating. For this lab, adapting the Router-on-a-Stick runbook into an SVI runbook made the work cleaner from the start.

The lesson is pretty straightforward: preparation wins.

Being good matters, but being well prepared makes the work repeatable. This lab reinforced that documentation is not just cleanup after the fact. Good documentation can directly improve the quality of the build.
