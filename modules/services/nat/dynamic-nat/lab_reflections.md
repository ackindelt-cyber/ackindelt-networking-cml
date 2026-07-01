# Lab Reflection — Dynamic NAT

## 1. Purpose and Context

This lab demonstrates Dynamic Network Address Translation on a Cisco router.

The goal was to translate traffic from an inside private network to addresses from a public NAT pool. R1 translates inside local addresses from `192.168.10.0/24` to public addresses from the `203.0.113.66` through `203.0.113.70` NAT pool.

This matters because NAT is commonly used at network edges to allow private inside networks to communicate with outside networks. Dynamic NAT is not as common as PAT in many modern environments, but it is useful for understanding NAT boundaries, inside local versus inside global addressing, and the translation process.

---

## 2. Design Rationale

The topology uses a simple inside-to-outside design.

R1 acts as the customer edge router and performs NAT. Its inside interface connects to the private LAN, and its outside interface connects to R2. R2 simulates an ISP router. C2 represents an outside host.

The NAT pool uses public addresses from `203.0.113.66` through `203.0.113.70`. That provides five possible public addresses for dynamic one-to-one translations. An ACL identifies which inside addresses are eligible for translation.

The ISP-side router routes the public NAT block toward R1. This is important because return traffic is sent to the translated public address, not directly to the private inside address.

---

## 3. Methodology and Testing Approach

Verification focused on proving that NAT was configured correctly and that traffic was actually translated.

The router verification confirmed:

* NAT inside and outside interfaces were configured.
* The Dynamic NAT pool existed.
* ACL `1` was bound to the NAT pool.
* NAT statistics showed the correct pool and address allocation.
* NAT translations appeared after traffic was generated.
* R1 had a default route toward R2.

Traffic testing confirmed:

* C1 could ping the outside host at `203.0.114.100`.
* R1 created a translation from inside local `192.168.10.10` to inside global `203.0.113.66`.
* NAT hit counters incremented during translated traffic.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

The main lesson from this lab is that Dynamic NAT is inside-initiated.

An outside host cannot reliably initiate traffic to a dynamically translated inside host unless an active translation already exists. The translation is created when inside traffic leaves the NAT boundary. Return traffic can then use the existing translation to get back to the inside host.

This helped reinforce the meaning of inside local and inside global addresses. The inside local address was `192.168.10.10`, and the inside global address was `203.0.113.66`. From the outside network’s perspective, the inside host is represented by the translated public address.

This lab also clarified why the upstream route to the public NAT block matters. R2 needs to know how to reach the translated address space. It does not need to know the private inside LAN for NAT return traffic to work.

---

## 5. Comparison and Next Steps

This lab fits naturally between static NAT and PAT.

Static NAT provides a fixed one-to-one mapping. Dynamic NAT provides temporary one-to-one mappings from a pool. PAT allows many inside hosts to share one public address by using ports.

Useful next steps include:

* Build a PAT lab.
* Build a static NAT lab if not already completed.
* Create a NAT troubleshooting runbook.
* Compare static NAT, Dynamic NAT, and PAT behavior.
* Add multiple inside clients to demonstrate pool allocation.
* Test NAT pool exhaustion.
* Add packet captures before and after translation.
* Document inside local, inside global, outside local, and outside global address meanings.

PAT is the next logical lab because it is the NAT form most commonly used for many-to-one edge translation.

---

## 6. Personal Insights

This lab was straightforward, but it still helped clarify NAT behavior.

Nothing major went wrong, which made it easier to focus on the translation itself instead of chasing a misconfiguration. The useful part was seeing the inside local address become an inside global address in the translation table.

The biggest takeaway was that Dynamic NAT is not the same thing as PAT. It uses a pool, but it still consumes public addresses for active translations. That makes it easier to understand conceptually, but less efficient than overload/PAT in most real environments.

This was a good foundation before moving into PAT, where the translation table gets more interesting because ports become part of the scaling mechanism.
