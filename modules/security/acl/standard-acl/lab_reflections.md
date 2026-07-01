# Lab Reflection — Standard ACL

## 1. Purpose and Context

This lab demonstrates how to configure and verify a standard IPv4 Access Control List.

The goal was to use a standard ACL to match traffic based on source IP address. In this lab, ACL `10` permits traffic sourced from the inside LAN network `192.168.10.0/24`. The ACL is applied outbound on R1’s WAN interface toward the outside network.

This matters because standard ACLs are one of the simplest forms of traffic classification and filtering on Cisco routers. They are limited compared to extended ACLs, but that limitation is useful to understand.

---

## 2. Design Rationale

The design uses the same basic inside/WAN/outside structure as the extended ACL lab.

R1 acts as the customer edge router. A1 provides inside access connectivity for C1 and C3. R2 acts as the upstream router, and C2 represents the outside host.

The ACL is applied outbound on R1’s WAN interface. This allows traffic sourced from the inside LAN to be matched as it leaves the customer edge router. Since this is a standard ACL, it only looks at the source address. It does not evaluate protocol, destination IP, or destination port.

The ACL permits the entire inside LAN subnet instead of individual hosts. That keeps the lab focused on the standard ACL source-matching behavior.

---

## 3. Methodology and Testing Approach

Verification focused on proving that the ACL was present, applied correctly, and matching traffic.

The router verification confirmed:

* Standard ACL `10` existed.
* The permit statement matched `192.168.10.0/24`.
* The ACL match counter incremented after test traffic.
* The ACL was applied outbound on R1’s WAN interface.
* R1 had a default route toward R2.

Traffic testing confirmed:

* C1 could reach the outside host at `203.0.113.100`.
* C3 could reach the outside host at `203.0.113.100`.
* The ACL match counter increased after inside-client traffic.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

The main lesson from this lab is that standard ACLs are simple but blunt.

They can match source addresses, but they cannot distinguish between different destination hosts, protocols, or ports. That makes them useful for broad source-based classification, but less useful when the goal is precise traffic filtering.

This lab also reinforces why ACL placement matters. With a standard ACL, placing it too close to the source can block more than intended because the ACL has no destination context. Applying it outbound on the WAN interface made sense for this lab because the goal was to match inside-source traffic as it left R1.

The verification is also intentionally limited. The captured evidence proves permitted inside-source traffic and ACL counter behavior. It does not prove a denied source test. That is an important distinction because good documentation should not claim more than the lab actually validates.

---

## 5. Comparison and Next Steps

This lab pairs directly with the extended ACL lab.

The standard ACL lab shows basic source-based matching. The extended ACL lab adds destination, protocol, and port matching, which provides more precise policy control.

Useful next steps include:

* Add a denied-source test from a non-permitted network.
* Compare standard ACL placement with extended ACL placement.
* Convert the numbered ACL to a named standard ACL.
* Add sequence-number edits to modify ACL behavior.
* Add an extended ACL version of the same traffic policy.
* Combine ACLs with NAT or PAT.
* Use ACLs for route filtering or NAT classification in future labs.

A future improvement would be adding a second source network so the implicit deny behavior can be validated directly.

---

## 6. Personal Insights

This lab felt more limited than the extended ACL lab, but that was useful in its own way.

A standard ACL is simple: it matches source IP and nothing else. That makes it easy to configure, but also easy to misuse if you forget how little context it has. It cannot care where the traffic is going or what kind of traffic it is. It only knows where the traffic came from.

That made the lab feel less exciting than the extended ACL version, but it helped reinforce why standard ACL placement matters so much. If the tool only sees source address, then placement becomes a major part of the design.

This was also a good reminder that simple does not always mean flexible. Standard ACLs have their place, but extended ACLs feel much more practical when the goal is real traffic control.
