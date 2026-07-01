# Lab Reflection — Static EtherChannel

## 1. Purpose and Context

This lab demonstrates how to configure and verify static EtherChannel using `mode on` between Layer 2 switches.

Static EtherChannel allows multiple physical links to operate as one logical Port-Channel. This can increase available bandwidth and simplify STP behavior by allowing STP to treat the bundle as a single logical link.

The lab also demonstrates why static EtherChannel is less forgiving than negotiated EtherChannel options such as LACP. Because static EtherChannel does not use a negotiation protocol, mismatches or partial configuration states can create instability before the switch clearly identifies the issue.

---

## 2. Design Rationale

The topology uses three Layer 2 switches connected in a triangle, with static EtherChannel bundles between each switch pair.

This design was chosen to validate two behaviors at the same time:

* Static EtherChannel formation between switch pairs
* STP behavior across logical Port-Channel links

The triangular topology intentionally creates redundant Layer 2 paths. This allows STP to select a forwarding path and block the redundant path while still treating each EtherChannel bundle as a single logical interface.

The design is useful for observing EtherChannel and STP interaction, but it also adds risk during configuration because static EtherChannel does not negotiate or protect against partial bundle formation the way LACP does.

---

## 3. Methodology and Testing Approach

Validation focused on confirming that each Port-Channel formed correctly, that member interfaces bundled as expected, and that STP recognized the Port-Channels as logical links.

The primary verification commands were:

* `show etherchannel summary`
* `show interfaces port-channel x`
* `show spanning-tree`
* `show interfaces gigabitEthernet0/x etherchannel`

Successful validation required the Port-Channels to show as Layer 2 and in use, member links to show bundled state, and STP to list the Port-Channels rather than treating each physical link independently.

Recorded verification output is available in [`verification/verification_commands.md`](verification/verification_commands.md).

---

## 4. Observations and Lessons Learned

Static EtherChannel is fragile compared to negotiated EtherChannel.

I expected static EtherChannel to require careful configuration, but this lab made the timing issue more obvious. Because the topology included a Layer 2 loop, partial Port-Channel formation created instability before all switch links were fully bundled.

During validation, S2 interfaces associated with Po3 entered an error-disabled state during partial formation. The issue appeared related to the interaction between static EtherChannel configuration timing and STP loop prevention behavior.

To recover, I removed the affected Port-Channel/member configuration, reset the affected interfaces with `shutdown` and `no shutdown`, preconfigured the Port-Channel interface, and then re-added the physical member interfaces to the bundle. After that, Po3 formed correctly and STP blocked the redundant path as expected.

The key lesson is that static EtherChannel should be configured carefully and consistently on both sides before member links are allowed to actively forward. In a real environment, LACP would generally be preferred because it provides negotiation and helps prevent mismatched or partially formed bundles from forwarding unexpectedly.

---

## 5. Comparison and Next Steps

This lab provides a useful baseline for comparing static EtherChannel against negotiated EtherChannel methods.

Next comparison points:

* LACP EtherChannel
* PAgP EtherChannel

The LACP version should be more operationally realistic because it includes negotiation and better protection against mismatched bundle configuration.

The PAgP version is useful mainly for understanding Cisco-proprietary EtherChannel behavior and how it differs from both static EtherChannel and LACP.

---

## 6. Personal Insights

I made this lab more complicated than it needed to be by combining static EtherChannel with a triangular STP topology.

That made the lab more interesting, but it also introduced instability that would not have appeared in a simpler two-switch static EtherChannel lab. In this case, the added complexity was useful because it exposed a real operational concern: static EtherChannel does not tolerate partial or inconsistent configuration well.

I am keeping the topology because it produced a useful lesson, but the better operational takeaway is clear: for production-aligned designs, LACP is the safer and more practical EtherChannel method.
