# Lab Reflection – Edge Firewall HA

## 1. Purpose and Context

This lab demonstrates a small enterprise edge design using a Cisco ASAv Active/Standby failover pair, redundant distribution switches, HSRP, Spanning Tree, static routing, and PAT.

The goal was not only to configure firewall high availability, but to validate how the surrounding Layer 2 and Layer 3 infrastructure reacts when individual links, firewall nodes, or distribution switches fail. This matters in a real network because redundancy is only useful if the complete forwarding path converges correctly during a failure.

---

## 2. Design Rationale

The topology was designed around two ASAv firewalls operating as an Active/Standby pair, with FW1 as the preferred Primary/Active unit and FW2 as the Secondary/Standby unit.

The outside firewall interfaces share a common Layer 2 segment through OS1 so the Active-role outside IP address can move between physical firewalls during failover. The inside interfaces connect separately to D1 and D2, with VLAN 99 providing the shared inside transit network.

D1 and D2 use HSRP for VLANs 10 and 99. D1 is configured with the higher HSRP priority and preemption so it normally owns the virtual gateway addresses and automatically reclaims the Active HSRP role after recovery.

The access layer uses redundant uplinks to D1 and D2. Spanning Tree prevents the redundant Layer 2 topology from forming a forwarding loop while preserving an alternate path if the preferred distribution switch becomes unavailable.

Static routing was used to keep the routing behavior explicit and easy to validate. The firewall points upstream toward R1 and internally toward the HSRP virtual address on VLAN 99. PAT translates the internal VLAN 10 network to the Active firewall's outside interface address.

---

## 3. Methodology and Testing Approach

Validation was divided into normal-state verification and controlled failure testing.

The normal verification process confirmed interface addressing, VLAN and trunk operation, HSRP roles, Spanning Tree state, routing, NAT, firewall policy, failover status, and end-to-end connectivity before any failure scenarios were introduced.

Failover testing then used a continuous ICMP stream from A1 to the ISP loopback at `192.0.2.100` while failures were deliberately introduced.

The following scenarios were tested:

- Manual firewall switchover from FW1 to FW2
- Failure of FW1's monitored inside path
- Failure of FW1's monitored outside path
- Complete loss of the FW1 node
- Complete loss of D1

Each test verified the resulting firewall or HSRP state, restored the failed component, confirmed automatic recovery, and then manually returned the firewalls to the preferred FW1 Active / FW2 Standby state.

Recorded command output was retained in `verification/failover-commands.md` as evidence of each test.

---

## 4. Observations and Lessons Learned

The most important lesson from this lab was that high availability involves more than assigning redundant addresses or configuring a standby device. The behavior of the entire forwarding path has to be understood and tested.

ASA failover status was initially one of the more confusing parts of the lab. Interface states such as `Normal (Monitored)`, `Normal (Waiting)`, `No Traffic (Waiting)`, and `Failed (Waiting)` do not directly describe whether an interface can forward production traffic. For example, when FW1 was completely powered off, FW2's active inside and outside interfaces could report `Normal (Waiting)` while still successfully forwarding end-to-end traffic. The `Waiting` state referred to peer-interface monitoring rather than data-plane failure.

Another important distinction was between the ASA's Primary/Secondary identity and its Active/Standby service role. After FW1 recovered from a failure, it automatically returned as `Primary - Standby Ready` rather than immediately reclaiming the Active role. Returning to the preferred FW1 Active state required a deliberate `failover active` command. This differs from HSRP, where D1 was configured to automatically reclaim the Active role through preemption.

Failure detection also required patience. Monitored-interface failover does not occur immediately, and checking `show failover` too quickly could capture temporary `Waiting` or `No Traffic` states before the failover process had completed. Testing therefore had to account for convergence time rather than assuming every state transition would be instantaneous.

The D1 node-failure test was particularly useful because it exercised several redundancy mechanisms at the same time. Losing D1 caused D2 to assume the HSRP Active roles, forced firewall traffic onto the FW2-D2 path, and required the Layer 2 topology to use the surviving access path while end-to-end connectivity recovered.

The lab also reinforced the importance of testing failure recovery rather than only testing failure itself. A redundant design is incomplete if a failed component cannot safely rejoin the topology and return to its intended standby or preferred role.

---

## 5. Comparison and Next Steps

This lab provides a strong foundation for more advanced enterprise redundancy and automation work.

Future versions could replace portions of the static routing design with a dynamic routing protocol and examine how routing convergence interacts with firewall and first-hop redundancy. Additional upstream redundancy could also remove OS1 and R1 as remaining single points of failure.

The completed manual implementation also provides a baseline for automation. The configuration and verification process could be recreated with Ansible, while pyATS could validate expected interface states, firewall roles, HSRP roles, routing information, and end-to-end reachability before and after controlled failures.

Other future additions could include centralized syslog, DNS services, monitoring, security telemetry, or expansion into a multi-campus topology.

---

## 6. Personal Insights

This lab changed how I think about redundancy.

Before building and testing the complete topology, it was easy to think of redundancy as individual technologies such as HSRP, Spanning Tree, or firewall failover. In practice, those mechanisms are interdependent. A successful failure response requires the firewall, Layer 2 topology, first-hop redundancy, routing, and upstream connectivity to converge into a usable forwarding path together.

The failover testing was considerably more difficult than the initial configuration because the challenge was no longer just determining whether the configuration was correct. It required understanding transient states, convergence timers, recovery behavior, and the difference between control-plane status and actual data-plane forwarding.

That made the testing portion one of the most valuable parts of the lab.