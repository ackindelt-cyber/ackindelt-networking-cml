# Build Order

## Status

**Status:** Draft  
**Last Updated:** 2026-08-30  
**Applies To:** Talos Solutions Enterprise Campus v1  
**Document Type:** Build Plan  

---

## Purpose

This document defines the planned implementation order for the Talos Solutions Enterprise Campus v1 lab.

The purpose of this document is to keep the build controlled, phased, and verifiable. Each phase should be completed and validated before moving to the next phase.

---

## Scope

### In Scope

- CML topology build order
- Device and link creation sequence
- Interface validation
- Base device preparation
- Layer 2 foundation
- VLAN and trunk implementation
- Port-channel implementation
- Core SVI and HSRP implementation
- STP/HSRP alignment
- Firewall HA implementation
- Static routing implementation
- DHCP and relay implementation
- PAT implementation
- Baseline verification
- Failure testing
- Final artifact capture

### Out of Scope

- Final device configurations
- Final verification output
- Troubleshooting procedures
- Automation
- Multi-site expansion
- Production change-control process

---

## Summary

The Talos Solutions Enterprise Campus v1 lab should be built in phases.

The build should start with the physical CML topology, not VLANs or addressing. Once the topology is physically built and interface counts are confirmed, the lab can move into base configuration, Layer 2 foundation, Layer 3 gateways, firewall HA, routing, DHCP, PAT, verification, and failure testing.

High-level build sequence:

```text
Topology -> Base Prep -> Layer 2 -> Layer 3 -> Firewall HA -> Routing -> DHCP -> PAT -> Verification -> Failure Testing
```

The main rule:

```text
Do not move to the next phase until the current phase has been validated.
```

---

## Build Principles

- Build the topology first.
- Validate interface support before assigning final interface numbers.
- Keep each phase small enough to troubleshoot.
- Do not configure everything at once.
- Verify after each major feature.
- Capture final configs only after the build is stable.
- Document any design changes before implementing them.
- Keep v1 focused and finishable.

---

## Phase 0 - Pre-Build Review

### Goal

Confirm that the planned design documents are ready enough to start building the lab in CML.

### Tasks

- Review `03-topology-and-device-roles.md`.
- Review `05-vlan-plan.md`.
- Review `06-addressing-plan.md`.
- Review `07-interface-map.md`.
- Review `08-port-channel-plan.md`.
- Review `09-routing-plan.md`.
- Review `10-firewall-plan.md`.
- Review `11-dhcp-plan.md`.
- Review `12-management-plan.md`.
- Review `16-required-runbooks.md`.

### Validation Gate

Before moving forward:

- Planned devices are known.
- Planned links are known.
- VLAN model is drafted.
- Addressing model is drafted.
- Firewall HA design is accepted.
- Major open questions are documented.

---

## Phase 1 - Build CML Topology

### Goal

Create the physical topology in Cisco CML.

### Devices to Add

| Device | CML Node Type | Role |
|---|---|---|
| ISP1 | IOSv | Simulated ISP |
| EDGE1 | IOSv | Customer edge router |
| OS1 | IOSvL2 | Outside transit switch |
| FW1 | ASAv | Firewall HA pair member |
| FW2 | ASAv | Firewall HA pair member |
| CORE1 | IOSvL2 | Collapsed core/distribution |
| CORE2 | IOSvL2 | Collapsed core/distribution |
| ASW1 | IOSvL2 | Access switch |
| ASW2 | IOSvL2 | Access switch |
| ASW3 | IOSvL2 | Access switch |
| INFRA1 | IOSv | DHCP/infrastructure server |
| C1-C4 | Alpine/Desktop | Client endpoints |
| PRN1 | Alpine/Desktop | Printer/IoT endpoint |

### Links to Add

- ISP1 to EDGE1
- EDGE1 to OS1
- OS1 to FW1
- OS1 to FW2
- FW1 to FW2
- FW1 to CORE1
- FW2 to CORE2
- CORE1 to CORE2 x2
- ASW1 to CORE1 x2
- ASW1 to CORE2 x2
- ASW2 to CORE1 x2
- ASW2 to CORE2 x2
- ASW3 to CORE1 x2
- ASW3 to CORE2 x2
- ASW2 to INFRA1
- ASW1 to C1
- ASW1 to C2
- ASW2 to C3
- ASW3 to C4
- ASW3 to PRN1

### Validation Gate

Before moving forward:

- All planned nodes exist.
- All required links exist.
- Devices boot successfully.
- CML supports the required interface count.
- Interface numbering is visible and documented.
- The topology layout is readable.
- A CML screenshot is captured.

### Output Artifacts

- `topology/cml-topology-screenshot.png`
- `cml/talos-enterprise-campus-v1.yaml`
- Updated `docs/07-interface-map.md`
- Updated `cml/cml-notes.md`

---

## Phase 2 - Base Device Preparation

### Goal

Apply basic device identity and lab-readiness configuration.

### Tasks

Apply base configuration to:

- ISP1
- EDGE1
- OS1
- FW1
- FW2
- CORE1
- CORE2
- ASW1
- ASW2
- ASW3
- INFRA1

Expected base items:

- Hostnames
- Disable DNS lookup where applicable
- Basic console behavior
- Basic logging/timestamps where appropriate
- Interface descriptions where known
- Save baseline configuration

### Validation Gate

Before moving forward:

- Hostnames are correct.
- Devices are reachable through console.
- Interfaces are identifiable.
- No accidental configuration exists from previous labs.
- Base configs are saved.

### Output Artifacts

- Initial config snapshots if desired
- Updated `configs/` later after final state is stable

---

## Phase 3 - Layer 2 Foundation

### Goal

Build the switching foundation before Layer 3 configuration.

### Tasks

On OS1, CORE1, CORE2, ASW1, ASW2, and ASW3:

- Create planned VLANs.
- Configure VLAN names.
- Configure trunk native VLAN.
- Configure parking VLAN.
- Configure unused ports as shutdown and parked where applicable.
- Disable DTP where supported.
- Configure access ports for endpoint VLANs.
- Configure OS1 outside transit switching behavior.

### VLANs

| VLAN | Name |
|---:|---|
| 10 | USERS |
| 20 | ADMIN |
| 30 | SERVERS_INFRA |
| 40 | PRINTERS_IOT |
| 50 | MGMT |
| 900 | FW_TRANSIT |
| 901 | OUTSIDE_TRANSIT |
| 998 | NATIVE |
| 999 | PARKING |

### Validation Gate

Before moving forward:

- VLANs exist where required.
- VLAN 901 remains local to OS1.
- VLAN 900 exists on CORE1 and CORE2.
- Access VLANs exist on access switches.
- VLAN 1 is not intentionally used for access, management, or native VLAN.
- Unused ports are handled consistently.

---

## Phase 4 - Port-Channels and Trunks

### Goal

Build the LACP trunk foundation between cores and access switches.

### Tasks

Configure:

- CORE1 to CORE2 inter-core port-channel
- ASW1 to CORE1 port-channel
- ASW1 to CORE2 port-channel
- ASW2 to CORE1 port-channel
- ASW2 to CORE2 port-channel
- ASW3 to CORE1 port-channel
- ASW3 to CORE2 port-channel

### Planned Port-Channels

| Port-Channel | Relationship |
|---|---|
| Po1 | CORE1 to CORE2 |
| Po11 | CORE1 to ASW1 |
| Po12 | CORE2 to ASW1 |
| Po21 | CORE1 to ASW2 |
| Po22 | CORE2 to ASW2 |
| Po31 | CORE1 to ASW3 |
| Po32 | CORE2 to ASW3 |

### Validation Gate

Before moving forward:

- All port-channels form correctly.
- All member links bundle correctly.
- Trunks are up.
- Native VLAN is correct.
- Allowed VLANs are correct.
- No access switch port-channel is split across both cores.

### Expected Commands

```text
show etherchannel summary
show interfaces trunk
show lacp neighbor
show running-config interface port-channel <number>
```

---

## Phase 5 - Core Layer 3, SVIs, and HSRP

### Goal

Configure campus gateway services on CORE1 and CORE2.

### Tasks

On CORE1 and CORE2:

- Enable Layer 3 routing.
- Configure SVIs for routed VLANs.
- Configure HSRP for each routed VLAN.
- Configure VLAN 900 firewall transit SVI.
- Configure VLAN 50 management gateway.
- Configure HSRP priorities and preemption as planned.
- Confirm clients will use HSRP VIPs as default gateways.

### Routed VLANs

| VLAN | Name | HSRP VIP |
|---:|---|---|
| 10 | USERS | `10.10.10.1` |
| 20 | ADMIN | `10.10.20.1` |
| 30 | SERVERS_INFRA | `10.10.30.1` |
| 40 | PRINTERS_IOT | `10.10.40.1` |
| 50 | MGMT | `10.10.50.1` |
| 900 | FW_TRANSIT | `10.10.90.1` |

### Validation Gate

Before moving forward:

- SVIs are up/up where expected.
- HSRP is active/standby as planned.
- HSRP VIPs are reachable.
- CORE1 and CORE2 can reach each other on routed VLANs.
- Access switches can reach the VLAN 50 HSRP VIP after management SVIs are configured.

### Expected Commands

```text
show ip interface brief
show standby brief
show ip route
ping <hsrp-vip>
```

---

## Phase 6 - STP and HSRP Alignment

### Goal

Align Layer 2 root placement with Layer 3 gateway placement.

### Tasks

- Configure STP root primary/secondary per VLAN.
- Align preferred STP root with HSRP active gateway.
- Verify STP path selection on access switches.
- Confirm VLANs prefer the expected core.

### Planned Alignment

| VLAN | Preferred Core | STP Role | HSRP Role |
|---:|---|---|---|
| 10 | CORE1 | Root primary | Active |
| 20 | CORE2 | Root primary | Active |
| 30 | CORE1 | Root primary | Active |
| 40 | CORE2 | Root primary | Active |
| 50 | CORE1 | Root primary | Active |
| 900 | CORE1 | Root primary | Active |

### Validation Gate

Before moving forward:

- STP root placement matches the plan.
- HSRP active placement matches the plan.
- Access-layer forwarding behavior is predictable.
- No unexpected STP blocking occurs on required forwarding paths.

### Expected Commands

```text
show spanning-tree vlan <vlan-id>
show standby brief
show interfaces trunk
```

---

## Phase 7 - Firewall HA Foundation

### Goal

Configure and validate FW1/FW2 as an ASAv active/standby pair.

### Tasks

On FW1 and FW2:

- Configure failover roles.
- Configure failover/state link.
- Configure outside interface addressing.
- Configure inside interface addressing.
- Configure interface monitoring if used.
- Verify active/standby status.
- Confirm active and standby IP ownership.

### Firewall Networks

| Segment | Network |
|---|---|
| Outside | `198.51.100.0/29` |
| Inside | `10.10.90.0/24` |
| Failover/state | `10.255.255.0/30` |

### Validation Gate

Before moving forward:

- FW1/FW2 failover relationship forms.
- One firewall is active.
- One firewall is standby.
- Active outside IP is owned by active firewall.
- Active inside IP is owned by active firewall.
- Standby IPs are visible as expected.
- No firewall failover instability exists.

### Expected Commands

```text
show failover
show interface ip brief
show route
show running-config failover
```

---

## Phase 8 - Static Routing

### Goal

Configure static routing across ISP1, EDGE1, FW1/FW2, CORE1/CORE2, and INFRA1.

### Tasks

Configure:

- ISP1 route to firewall outside transit network.
- EDGE1 default route to ISP1.
- Firewall default route to EDGE1.
- Firewall route to campus summary through VLAN 900 HSRP VIP.
- CORE1 and CORE2 default routes to firewall active inside IP.
- INFRA1 default route to VLAN 30 HSRP VIP.

### Validation Gate

Before moving forward:

- ISP1 can reach EDGE1.
- EDGE1 can reach ISP1 and firewall outside subnet.
- Firewall can reach EDGE1.
- Firewall can reach VLAN 900 HSRP VIP.
- CORE1/CORE2 can reach firewall inside active IP.
- INFRA1 can reach VLAN 30 gateway.
- Routes appear as expected.

### Expected Commands

```text
show ip route
show route
ping
traceroute
```

---

## Phase 9 - DHCP and Relay

### Goal

Configure centralized DHCP service on INFRA1 and DHCP relay on CORE1/CORE2.

### Tasks

On INFRA1:

- Enable DHCP service.
- Configure excluded ranges.
- Configure DHCP pools.
- Configure default-router values as HSRP VIPs.
- Configure default route toward VLAN 30 HSRP VIP.

On CORE1 and CORE2:

- Configure DHCP relay on required client VLAN SVIs.

### DHCP VLANs

| VLAN | Name | DHCP Status |
|---:|---|---|
| 10 | USERS | DHCP |
| 20 | ADMIN | DHCP |
| 30 | SERVERS_INFRA | Static |
| 40 | PRINTERS_IOT | DHCP |
| 50 | MGMT | Static |

### Validation Gate

Before moving forward:

- Clients receive DHCP addresses.
- Clients receive correct subnet masks.
- Clients receive HSRP VIPs as default gateways.
- INFRA1 shows DHCP bindings.
- Clients can ping their default gateways.
- Clients can ping INFRA1 where expected.

### Expected Commands

```text
show ip dhcp binding
show ip dhcp pool
show running-config | section dhcp
show running-config interface vlan <vlan-id>
```

---

## Phase 10 - Firewall PAT and Basic Policy

### Goal

Configure outbound PAT and basic firewall behavior.

### Tasks

On FW1/FW2:

- Configure inside/outside interface roles.
- Configure PAT for internal campus VLANs that require outbound access.
- Configure basic inside-to-outside policy if required.
- Confirm outside-to-inside unsolicited traffic is denied by default.
- Validate traffic from internal clients to simulated external destination.

### Validation Gate

Before moving forward:

- Internal clients can reach the simulated external destination.
- PAT translations appear on the active firewall.
- Return traffic works.
- EDGE1 sees traffic sourced from the firewall outside active IP.
- ISP1 does not require routes to internal campus subnets.
- Outside-to-inside behavior matches the intended policy.

### Expected Commands

```text
show nat
show xlate
show conn
show route
packet-tracer
```

---

## Phase 11 - Baseline Verification

### Goal

Prove the completed v1 network works under normal conditions.

### Validation Areas

- Device status
- Interface status
- VLANs
- Trunks
- Port-channels
- STP
- HSRP
- Routing
- DHCP
- Firewall HA
- PAT
- Management reachability
- End-to-end client reachability

### Output Artifact

```text
verification/01-baseline-verification.md
```

### Validation Gate

Before moving forward:

- All baseline tests pass or are documented.
- Any failed test has a known cause.
- Any workaround is documented.
- Configs are saved.

---

## Phase 12 - Failure Testing

### Goal

Prove the redundancy design behaves as expected.

### Planned Failure Tests

- Single LACP member failure
- Full access uplink port-channel failure
- CORE1 failure
- CORE2 failure
- HSRP failover
- STP path change
- Firewall failover from FW1 to FW2
- Firewall failback behavior if tested
- Client reachability before and after failures
- DHCP behavior after failover
- PAT behavior after firewall failover

### Output Artifact

```text
verification/07-failure-testing.md
```

### Validation Gate

Before moving forward:

- Expected failover behavior is documented.
- Actual failover behavior is documented.
- Any unexpected behavior is explained.
- Any design limitation is added to `14-known-limitations.md`.

---

## Phase 13 - Final Artifact Capture

### Goal

Capture the final project artifacts after the network is stable.

### Tasks

- Save final running configurations.
- Export CML topology.
- Capture CML screenshot.
- Capture logical topology diagram.
- Update interface map with final interface numbers.
- Update port-channel plan with final member interfaces.
- Update addressing plan if changes occurred.
- Update known limitations.
- Update future roadmap.
- Review README.

### Output Artifacts

```text
configs/
cml/talos-enterprise-campus-v1.yaml
topology/cml-topology-screenshot.png
topology/logical-topology.png
verification/
docs/
```

### Validation Gate

The v1 build is complete when:

- Final configs are stored.
- CML topology export is stored.
- Diagrams are stored.
- Baseline verification is complete.
- Failure testing is complete.
- Known limitations are documented.
- README accurately reflects the finished lab.

---

## Build Order Summary

| Phase | Name | Primary Outcome |
|---:|---|---|
| 0 | Pre-Build Review | Docs ready enough to build |
| 1 | Build CML Topology | Nodes and links exist |
| 2 | Base Device Preparation | Devices ready for configuration |
| 3 | Layer 2 Foundation | VLANs and basic switching prepared |
| 4 | Port-Channels and Trunks | LACP/trunks operational |
| 5 | Core Layer 3, SVIs, and HSRP | Campus gateways operational |
| 6 | STP and HSRP Alignment | Predictable L2/L3 path alignment |
| 7 | Firewall HA Foundation | FW1/FW2 active/standby operational |
| 8 | Static Routing | End-to-end routing path built |
| 9 | DHCP and Relay | Clients receive correct leases |
| 10 | Firewall PAT and Basic Policy | Internal clients reach outside simulation |
| 11 | Baseline Verification | Normal-state operation proven |
| 12 | Failure Testing | Redundancy behavior proven |
| 13 | Final Artifact Capture | Portfolio artifacts finalized |

---

## Design Notes

- The build should not start with full device configs.
- The physical CML topology must be validated first.
- Firewall HA should be validated before relying on routing and PAT behavior.
- Layer 2 should be stable before Layer 3 is layered on top.
- DHCP should be tested after routing is stable.
- PAT should be tested after routing and DHCP are stable.
- Failure testing should happen only after baseline verification passes.
- Any major design change should be reflected in the relevant design doc before final configs are saved.

---

## Validation or Success Criteria

The build order is successful when:

- The lab can be built in a repeatable sequence.
- Each phase has a clear purpose.
- Each phase has a validation gate.
- Troubleshooting is isolated by phase.
- Final configs match the documented design.
- Verification proves the design works.
- Failure testing proves redundancy behavior.
- The final project can be explained clearly to a technical reviewer.

---

## Open Questions

- None currently identified.

---

## Related Documents

- `docs/01-initial-planning.md`
- `docs/03-topology-and-device-roles.md`
- `docs/04-design-decisions.md`
- `docs/05-vlan-plan.md`
- `docs/06-addressing-plan.md`
- `docs/07-interface-map.md`
- `docs/08-port-channel-plan.md`
- `docs/09-routing-plan.md`
- `docs/10-firewall-plan.md`
- `docs/11-dhcp-plan.md`
- `docs/12-management-plan.md`
- `docs/14-known-limitations.md`
- `docs/15-future-roadmap.md`
- `verification/01-baseline-verification.md`
- `verification/07-failure-testing.md`

---

## Change Log

| Date | Change |
|---|---|
| 2026-08-30 | Initial draft |