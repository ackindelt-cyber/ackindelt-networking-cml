# Lab Reflection — Extended ACL

## 1. Purpose and Context

This lab demonstrates how to configure and verify an extended IPv4 Access Control List.

The goal was to filter traffic from an inside LAN toward an outside server using protocol, source network, destination host, and destination port. The ACL permits TCP traffic from the inside LAN to `203.0.113.100` on port `8080`. All other IP traffic is denied and logged.

This matters because extended ACLs are a basic but important traffic-control tool. They allow filtering decisions to be made with more context than a standard ACL, which only matches source addresses.

---

## 2. Design Rationale

The topology separates the network into inside, WAN, and outside segments.

R1 acts as the customer edge router. Its inside interface connects to the LAN, and its outside interface connects to R2. R2 represents the upstream or ISP-side router. S1 represents an outside service host.

The ACL is applied inbound on R1’s inside interface. This placement filters traffic close to the source before unwanted traffic crosses the WAN link. The permitted traffic is intentionally narrow: inside clients may reach only the outside server on TCP port `8080`.

A netcat listener was used on S1 because it provides a simple way to prove that TCP/8080 traffic is permitted and that payload data reaches the destination.

---

## 3. Methodology and Testing Approach

Verification focused on proving both permitted and denied behavior.

The router verification confirmed:

* The named extended ACL existed.
* The TCP/8080 permit statement was present.
* The explicit deny statement was present.
* ACL match counters incremented.
* The ACL was applied inbound on R1’s inside interface.
* R1 had a default route toward R2.

The allowed-traffic test confirmed:

* C1 could send TCP traffic to S1 on port `8080`.
* S1 received the expected netcat payload.
* The permit counter incremented.

The denied-traffic test confirmed:

* ICMP from C2 to S1 failed.
* R1 logged the denied ICMP packet.
* The deny counter incremented.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

The biggest lesson from this lab was that ACL name accuracy matters.

During the lab, I applied a misspelled ACL name to the interface. That created a silent failure where traffic was not filtered the way I expected. The fix was to check the interface with `show ip interface` and confirm the exact inbound ACL name.

This was a good reminder that ACL troubleshooting is not just about the ACL entries. You also have to confirm where the ACL is applied, what direction it is applied in, and whether the name matches exactly.

The deny logging also surfaced another important point: an ACL can block more traffic than expected. DHCP traffic would also be blocked by this ACL if DHCP were in scope and crossing this interface in the filtered direction. This lab does not use DHCP, but the observation matters. Before applying an ACL, you need to understand what traffic actually moves through that interface.

---

## 5. Comparison and Next Steps

This lab builds naturally toward NAT and DHCP.

The next logical steps include:

* Build a DHCP lab.
* Build a static NAT lab.
* Build a dynamic NAT lab.
* Build a PAT lab.
* Compare standard ACL placement with extended ACL placement.
* Add named ACL edits and sequence-number changes.
* Add more specific deny statements before the final deny.
* Test ACL behavior with additional protocols and ports.

A future version of this lab could also include packet captures showing permitted TCP/8080 traffic and denied ICMP traffic.

---

## 6. Personal Insights

This was the first lab where I hit a misconfiguration and found the issue without relying on a pre-written troubleshooting process or outside resources.

That matters. I knew where to look, what command to use, and what the output should show. The problem was not complicated, but solving it from memory showed progress.

The larger takeaway is that I am getting better at thinking like a network troubleshooter. Instead of guessing, I checked the applied interface, checked the ACL, checked the counters, and matched the observed behavior against the intended policy.
