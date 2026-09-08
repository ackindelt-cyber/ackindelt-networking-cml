# Lab Reflection – MPLS L3VPN Service Provider Lab

## 1. Purpose and Context

The purpose of this lab was to build a small service provider network that provides Layer 3 VPN connectivity between two customer sites using MPLS.

The provider core uses IS-IS for internal Layer 3 reachability, LDP for MPLS transport-label distribution, and MP-BGP VPNv4 to exchange customer routes and VPN labels between the provider edge routers. VRFs on PE1 and PE2 maintain customer routing separation while P1 operates only as a transit MPLS router.

This lab was intended to move beyond learning each protocol in isolation and demonstrate how the individual control-plane components work together to provide an end-to-end MPLS L3VPN service.

---

## 2. Design Rationale

The topology was designed with two customer sites connected through a three-router provider core consisting of PE1, P1, and PE2.

Each customer site includes an access switch, a distribution switch, and a customer edge router. This provided enough customer-side infrastructure to validate local Layer 2 and Layer 3 forwarding before introducing the provider network.

The provider design intentionally separates customer routing from provider transport:

- IS-IS provides reachability between provider infrastructure addresses and loopbacks.
- LDP distributes transport labels for provider FECs.
- MPLS provides label-switched forwarding across the provider core.
- VRFs provide separate customer routing and forwarding tables on the provider edge routers.
- MP-BGP VPNv4 exchanges customer VPN routes and associated VPN labels between PE1 and PE2.
- Route Distinguishers provide uniqueness for VPNv4 routes.
- Route Targets control VPN route import and export between CUSTOMER_A VRFs.
- P1 does not participate in MP-BGP and does not maintain customer LAN routes.

PE loopbacks were used as stable IS-IS, LDP, and MP-BGP control-plane endpoints. The PE-to-PE MP-BGP session therefore relies on the IS-IS underlay for reachability while remaining independent of the directly connected provider links.

Static routing was used on the customer side to keep the focus of the lab on the MPLS L3VPN architecture rather than introducing another PE-CE routing protocol.

---

## 3. Methodology and Testing Approach

Validation was performed progressively from the customer edge toward the provider core rather than relying only on an end-to-end ping.

Customer-side verification confirmed:

- Access VLAN membership and switchport operation.
- SVI and routed-interface status.
- Local default gateways and static routes.
- Hop-by-hop reachability between TSA, TSD, TSE, and PE devices.

Provider verification then confirmed each control-plane layer independently:

- IS-IS adjacencies and provider loopback reachability.
- LDP neighbor relationships and local/remote label bindings.
- MPLS forwarding-table entries.
- MP-BGP VPNv4 peering and customer route exchange.
- VPN/service label assignment to VPNv4 routes.
- Installation of remote customer routes into the correct VRF.
- Absence of customer LAN routes from P1's global routing table.

Finally, bidirectional pings between `10.10.10.10` and `10.20.20.10` confirmed end-to-end CUSTOMER_A connectivity across the MPLS L3VPN.

Recorded command output is available in [`verification/verification-commands.md`](verification/verification-commands.md).

---

## 4. Observations and Lessons Learned

This lab required more troubleshooting than most of the earlier module labs, which made the validation process especially useful.

Several configuration issues were discovered during initial testing:

- The IOSvL2 access switches were operating with Layer 3 routing enabled by default. Because of this, `ip default-gateway` was not being used for off-subnet traffic. Explicitly configuring `no ip routing` on TSA1 and TSA2 corrected the intended Layer 2 access-switch behavior.
- Both provider-facing interfaces on P1 were initially administratively shut, preventing basic routed connectivity through the provider core.
- PE2 contained a typo in the IS-IS process name, preventing the expected IS-IS adjacency with P1.
- TSD2's routed interface toward TSE2 had not been fully configured.
- TSE2 contained a typo in the static route for the Site B LAN, which created a valid route to the wrong destination prefix and caused a return-path failure.

Working through these failures reinforced the value of troubleshooting from the simplest forwarding dependency upward.

The most useful troubleshooting pattern was:

1. Confirm local Layer 2 and Layer 3 connectivity.
2. Verify directly connected routed links.
3. Verify the provider IGP.
4. Verify LDP.
5. Verify the MPLS forwarding table.
6. Verify MP-BGP VPNv4.
7. Verify VRF route installation.
8. Return to end-to-end customer forwarding.

This prevented unrelated technologies from becoming suspects before their underlying dependencies had been proven.

The lab also clarified several MPLS L3VPN concepts that were initially difficult to visualize.

The most important distinction was between the two label functions:

- LDP provides the outer transport label used to move traffic toward the remote PE.
- MP-BGP provides the VPN/service label associated with the customer route.

It also became much clearer that the provider core does not need customer routes. P1 forwards traffic using MPLS transport state while the PE routers maintain the customer-specific routing information.

Another important distinction was between the VRF and the VPNv4 table. The VRF is the customer-specific forwarding table used for customer packet routing, while the VPNv4 table is the shared MP-BGP control-plane mechanism used to exchange those customer routes between provider edge routers.

---

## 5. Comparison and Next Steps

This lab differs from earlier routing labs because multiple control-plane technologies must work together before the final service can operate.

In a basic OSPF or IS-IS lab, the dynamic routing protocol directly creates the IP forwarding information required for traffic delivery. In this design, IS-IS provides only the provider underlay. Customer VPN reachability depends additionally on LDP, MPLS forwarding, VRFs, and MP-BGP VPNv4.

The next step is to break the integrated architecture into smaller focused labs so each component can be practiced independently:

- IS-IS Fundamentals
- LDP Fundamentals
- MPLS Forwarding Fundamentals
- VRF / VRF-Lite Fundamentals
- BGP Fundamentals Refresher
- MP-BGP VPNv4 Fundamentals
- MPLS L3VPN Fundamentals
- Route-Based IPsec / VTI Fundamentals

After completing those focused labs, this integrated MPLS L3VPN lab can be revisited with a stronger understanding of each individual component.

Possible future extensions include:

- Multiple customer VRFs.
- Overlapping customer address space.
- Dynamic PE-to-CE routing.
- Controlled protocol and link failures.
- Additional provider routers.
- Route Target import/export policy changes.
- Customer-controlled IPsec encryption across the MPLS L3VPN.

---

## 6. Personal Insights

This lab reinforced that understanding a complex network does not require understanding every component simultaneously.

At the beginning, the combination of IS-IS, LDP, MPLS, VRFs, MP-BGP, Route Distinguishers, Route Targets, and multiple MPLS labels felt like one large technology. Building and troubleshooting the network made it easier to see each component as a separate responsibility with a defined relationship to the others.

The most important takeaway was that complexity became manageable once the network was treated as a series of dependencies rather than a single system.

The troubleshooting process also reinforced the value of proving what is working before investigating what is not. Several failures initially appeared capable of being MPLS or routing-protocol problems but were ultimately simple configuration issues such as a shutdown interface or an incorrect static route.

This lab was my first integrated MPLS L3VPN build. It does not represent deep production experience with the technology, but it provided a much stronger conceptual foundation and a clear roadmap for the focused labs that will follow.

---