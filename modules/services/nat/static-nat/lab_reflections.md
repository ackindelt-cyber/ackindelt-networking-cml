# Lab Reflection — Static NAT

## 1. Purpose and Context

This lab demonstrates Static Network Address Translation on a Cisco router.

The goal was to create a permanent one-to-one mapping between an inside private host and a public address. In this lab, inside host `192.168.10.10` is mapped to public address `203.0.113.66`.

This matters because Static NAT is commonly used when an inside resource needs to be reachable from an outside network using a consistent public address. It also provides a foundation for understanding Dynamic NAT and PAT.

---

## 2. Design Rationale

The topology uses a simple edge NAT design.

R1 acts as the customer edge router and performs NAT. Its inside interface connects to the private LAN, and its outside interface connects to R2. R2 simulates an ISP router, and C2 represents an outside host.

The static NAT mapping gives C1 a fixed inside global address. This allows inside-to-outside traffic to be translated consistently and also allows outside-to-inside testing by targeting the public NAT address.

The public NAT block is separate from both the inside LAN and the outside host network. This separation matters. The public NAT block must be routed toward R1 by the upstream router, but it should not overlap with the inside LAN or outside network.

---

## 3. Methodology and Testing Approach

Verification focused on proving that the static NAT mapping worked in both directions.

The router verification confirmed:

* The static NAT translation existed.
* The inside local address mapped to the expected inside global address.
* NAT inside and outside interfaces were recognized.
* NAT hit counters incremented after traffic was generated.
* R1 had a default route toward R2.

Traffic testing confirmed:

* C1 could ping the outside host at `203.0.114.100`.
* Traffic from C1 was translated from `192.168.10.10` to `203.0.113.66`.
* C2 could ping `203.0.113.66`.
* Traffic destined for `203.0.113.66` was translated back to `192.168.10.10`.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

This was the first lab where a design issue caused problems instead of a simple command misconfiguration.

The original addressing design placed the LAN segment and the public NAT block too close together conceptually, which created confusion during troubleshooting. The lesson is that planning can look thorough and still contain a bad assumption. Addressing design needs to be checked as carefully as command syntax.

Another issue was more basic: the outside-side interface was administratively down. I initially started troubleshooting routing when the better first step would have been checking interface state. That reinforced the value of starting from the bottom of the stack before chasing more complex causes.

The lab also made the static NAT behavior very clear. Unlike Dynamic NAT, the mapping exists whether traffic is currently active or not. That is what allows outside-to-inside testing by targeting the inside global address.

---

## 5. Comparison and Next Steps

This lab is the clean starting point for the NAT series.

Static NAT provides a fixed one-to-one mapping. Dynamic NAT uses temporary one-to-one mappings from a pool. PAT allows many inside hosts to share one public address by tracking transport-layer ports.

Useful next steps include:

* Build a Dynamic NAT lab.
* Build a PAT lab.
* Create a NAT troubleshooting runbook.
* Compare static NAT, Dynamic NAT, and PAT behavior.
* Add packet captures before and after translation.
* Document inside local, inside global, outside local, and outside global address meanings.
* Add service-specific static NAT or port forwarding in a future lab.

The next logical comparison is Dynamic NAT, followed by PAT.

---

## 6. Personal Insights

Static labs seem to be where I find the weirdest self-inflicted problems.

That pattern is annoying, but useful. Static routing and Static NAT are both simple on paper, which makes it tempting to assume the design is obvious. This lab was a reminder that simple does not mean mistake-proof.

The upside is that the mistakes were productive. The addressing design issue forced me to think more clearly about the public NAT block, and the administratively down interface reminded me not to skip basic checks.
