# Lab Reflection — Static Two Router

## 1. Purpose and Context

This lab demonstrates basic manual routing with static routes.

The goal was to connect two routers with a point-to-point link and use static default routes so each router could forward traffic toward the remote LAN. Unlike OSPF or another dynamic routing protocol, static routing requires the administrator to manually define where traffic should go.

This matters because static routing is one of the simplest ways to understand router forwarding behavior. Even when dynamic routing protocols are used in larger networks, static routes are still common for edge paths, default routes, lab environments, and small networks.

---

## 2. Design Rationale

The topology was intentionally kept small:

* Two routers
* One point-to-point transit link
* One LAN per router
* Default static routes on both routers
* Optional clients for endpoint testing

The design was based on the same general topology as the two-router OSPF lab. That made it easier to compare manual routing with dynamic routing while keeping the addressing and topology simple.

A `/30` subnet was used for the point-to-point link because only two usable router addresses were needed. `/24` LANs were used for readability and simplicity.

Default routes were used instead of more specific static routes. In a two-router topology, a default route on each router is enough to send non-local traffic toward the other router.

---

## 3. Methodology and Testing Approach

Verification focused on proving that forwarding worked without a dynamic routing protocol.

The router verification confirmed:

* R1 and R2 interfaces were up/up.
* Each router had the expected connected routes.
* Each router had a default static route pointing to the other router.
* No dynamic routing protocol was configured.
* ARP entries were present for local and next-hop addresses.
* CDP confirmed the direct router-to-router connection.

Connectivity testing confirmed that R1 could reach the R2 LAN gateway using the R1 LAN gateway as the source. Traceroute confirmed the next hop was R2 over the point-to-point link.

Recorded verification output is available in `verification/verification_commands.md`.

---

## 4. Observations and Lessons Learned

Static routing is straightforward in very small topologies.

With only two routers, it is easy to see what is happening. Each router knows its connected networks automatically, and the static default route tells it where to send anything else.

The downside also becomes obvious quickly. Static routing does not scale well when the number of routers and networks increases. Every remote path has to be planned and configured manually, and return routing matters just as much as forward routing.

This lab also reinforced the difference between connected and static routes. A connected route has an administrative distance of 0 and is preferred over a static route with administrative distance 1. If a static route overlaps with a connected route, the connected route wins and the static route may not appear in the routing table the way expected.

That is a useful troubleshooting lesson: always check both the running configuration and the routing table. A route being configured does not automatically mean it is installed or selected.

---

## 5. Comparison and Next Steps

This lab pairs well with the two-router OSPF lab.

The static route version is simpler to configure at small scale, but OSPF becomes more useful as the topology grows. Static routing is manual and predictable. Dynamic routing is more flexible and better suited for larger or changing networks.

Useful next steps include:

* Build a multi-router static routing lab.
* Compare specific static routes with default static routes.
* Add a floating static route.
* Test route preference using administrative distance.
* Compare static routing failure behavior with OSPF reconvergence.
* Add client-to-client verification output if endpoint testing is part of the lab scope.

A larger static routing lab is possible, but it is lower priority because the operational value drops quickly compared to dynamic routing.

---

## 6. Personal Insights

This was the second lab I built, and it made me think I probably should have built static routing before OSPF.

After working through the configuration, though, I understood why OSPF felt more interesting. Static routing is useful, but it does not scratch the same logic itch. It feels manual and rigid compared to a routing protocol that can learn and adapt.

That does not make static routing unimportant. It absolutely has a place. Default routes, edge routing, small networks, lab testing, and controlled backup paths all depend on the same basic idea.

The bigger takeaway is that manual routing makes the forwarding path easy to see. It is not elegant, but it is clear. That makes it a good foundation before moving into dynamic routing protocols.
