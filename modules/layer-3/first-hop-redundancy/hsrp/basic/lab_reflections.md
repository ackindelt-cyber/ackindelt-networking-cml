# Lab Reflection — Basic HSRP

## 1. Purpose and Context

This lab demonstrates first-hop gateway redundancy using HSRP.

The goal was to configure two routers on the same LAN segment so they could share a virtual default gateway. R1 is the preferred active router, and R2 is the standby router. If R1 fails, R2 takes over the virtual gateway address. When R1 recovers, preemption allows it to return to the active role.

This matters because client devices should not lose gateway access just because one physical router fails. HSRP provides a simple redundancy mechanism for that default-gateway function.

---

## 2. Design Rationale

The design is intentionally simple:

* Two routers
* One Layer 2 switch
* One client
* One HSRP group
* One virtual gateway

That keeps the focus on HSRP behavior instead of mixing in routing protocols, multiple VLANs, tracking, or upstream failover.

R1 uses priority `110`, making it the preferred active router. R2 uses priority `100`, making it the standby router under normal conditions. Both routers share the virtual IP address `192.168.10.1`, which acts as the client default gateway.

This topology also makes failover easy to simulate. Shutting down R1’s LAN interface removes the active router from the HSRP group, allowing R2 to become active. Bringing R1 back online validates recovery and preemption.

---

## 3. Methodology and Testing Approach

Validation focused on proving the normal state, failure state, and recovery state.

The initial verification confirmed:

* R1 and R2 LAN interfaces were up/up.
* R1 was active.
* R2 was standby.
* The virtual IP address was `192.168.10.1`.
* The virtual MAC address was `0000.0c07.ac0a`.
* Preemption was enabled.
* The client could reach the virtual gateway.

The failure test confirmed:

* R2 became active after R1’s LAN interface was shut down.
* The client could still reach the virtual gateway after failover.

The recovery test confirmed:

* R1 returned to the active role after its LAN interface came back up.
* The client could still reach the virtual gateway after recovery.

Packet captures were also collected for selected ICMP and HSRP states.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

This lab was straightforward to configure, but the packet captures made the behavior more interesting.

While capturing traffic, I saw HSRP messages from both routers. At first, that looked like it might be instability or flapping. After looking closer, it made sense: the routers were communicating their HSRP state to each other. Seeing both active and standby state information in the capture was expected behavior, not a problem.

The lab also reinforced the relationship between priority and preemption. Priority decides which router should be preferred. Preemption decides whether that preferred router should reclaim the active role after it returns.

The cleanest part of HSRP is that the client does not need to know which physical router is active. It only needs the virtual gateway. The routers handle the active and standby logic behind the scenes.

---

## 5. Comparison and Next Steps

HSRP is one first-hop redundancy option. It is Cisco proprietary, while VRRP provides a standards-based alternative. GLBP is another Cisco option that can provide gateway redundancy with load balancing behavior.

Useful next steps include:

* Add interface tracking so router priority changes when an upstream path fails.
* Test faster HSRP timers.
* Compare HSRP with VRRP.
* Compare HSRP with GLBP.
* Add multiple VLANs with one HSRP group per VLAN.
* Combine HSRP with SVI-based inter-VLAN routing.
* Add routing beyond the local LAN to test more realistic failure scenarios.

For basic default-gateway redundancy, HSRP is simple and effective. In a larger design, it should be used where gateway availability matters, not blindly applied everywhere.

---

## 6. Personal Insights

This lab felt elegant because the mechanism is simple and useful.

A client points to one gateway address. Behind that address, two routers decide who is active, who is standby, and who should take over during failure. That is a clean solution to a real operational problem.

It is easy to see why first-hop redundancy matters. Without something like HSRP, a single gateway failure can turn into a user-facing outage. With it, the failure can be absorbed by the network design instead of becoming an immediate firefight for the admin.

