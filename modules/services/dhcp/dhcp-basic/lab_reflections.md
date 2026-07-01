# Lab Reflection — Basic DHCP

## 1. Purpose and Context

This lab demonstrates basic DHCP service on a Cisco router.

The goal was to have R1 automatically assign IP addressing information to clients in two different VLANs. Instead of manually configuring client IP addresses, each client receives an address from the correct subnet through DHCP.

This matters because DHCP is foundational in real networks. Manually addressing every endpoint does not scale, creates avoidable mistakes, and slows down testing. DHCP makes client onboarding repeatable and much easier to validate.

---

## 2. Design Rationale

The topology uses router-on-a-stick because it allows one router interface to support multiple VLAN gateways.

VLAN 10 uses the `192.168.10.0/24` subnet, and VLAN 20 uses the `192.168.20.0/24` subnet. R1 provides the default gateway for both VLANs using subinterfaces. It also provides DHCP pools for both VLANs.

The first twenty addresses in each subnet are excluded from DHCP. This reserves space for gateway addresses and future static infrastructure addresses. DHCP clients receive addresses from the remaining available range.

Two clients were included so DHCP could be tested from both VLANs. Inter-VLAN ping testing was used to confirm that the clients not only received addresses, but could also communicate through the router using those DHCP-assigned addresses.

---

## 3. Methodology and Testing Approach

Verification focused on proving DHCP assignment and routed connectivity.

The router verification confirmed:

* DHCP bindings existed for both clients.
* Each client received an address from the expected subnet.
* DHCP pool usage showed one leased address per pool.
* The excluded address ranges were present.
* The expected DHCP pools were configured.

The client verification confirmed:

* C1 received a dynamic address from `192.168.10.0/24`.
* C2 received a dynamic address from `192.168.20.0/24`.
* C1 could ping C2 using DHCP-assigned addresses.
* C2 could ping C1 using DHCP-assigned addresses.

Packet-level inspection was also used during troubleshooting to confirm that the DHCP Offer included the expected gateway information.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

The most important lesson from this lab was that DHCP troubleshooting is not always router-side.

The router configuration looked correct. The DHCP bindings existed, the pools were active, and the clients received addresses from the correct subnets. At first, that made the failure confusing because the clients could ping their gateways but could not communicate outside their own subnet.

The issue turned out to be client-side behavior in the Alpine CML nodes. Packet capture showed that the DHCP Offer included the correct default gateway, but the Alpine client did not install the gateway as expected. Switching to Ubuntu Server clients resolved the issue.

That was a useful reminder: once the router-side DHCP configuration looks correct, the client still has to accept and install the offered information properly.

---

## 5. Comparison and Next Steps

This lab should become the foundation for future client-based labs.

Manually configuring client addressing is tedious and error-prone. DHCP makes the lab environment cleaner and more realistic.

Useful next steps include:

* Create a DHCP troubleshooting runbook.
* Use DHCP in future multi-client labs.
* Add DHCP relay in a separate lab.
* Add DHCP exclusion and lease-timing variations.
* Capture and document the DHCP Discover, Offer, Request, and Acknowledgment process.
* Compare Alpine and Ubuntu client behavior in CML.
* Use DHCP before building NAT and PAT labs.

The most immediate next step is to create a reusable DHCP runbook so client addressing stops becoming manual setup work in every lab.

---

## 6. Personal Insights

This lab was frustrating in a useful way.

Seeing the clients receive DHCP addresses was exciting because it felt like the main problem was solved. Then the first routed connectivity test failed, which made the result feel worse than a clean failure. It looked almost right, but not quite.

Checking the router configuration was not enough, so the next step was packet capture. Seeing the DHCP Offer at the packet level and confirming that the gateway was actually included made the issue much clearer.

That was a good moment. It showed that troubleshooting does not stop at show commands. Sometimes the answer is in the packet, and being able to prove that is a real skill.
