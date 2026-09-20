# Runbook — ASAv Active/Standby Failover

## Overview

This runbook provides a structured process for configuring and validating Cisco ASAv active/standby failover between two firewalls.

Active/standby failover provides firewall redundancy by allowing one ASAv to operate as the active unit while a second ASAv maintains synchronized configuration and connection state as the standby unit. If the active firewall becomes unavailable, the standby firewall can assume the active role and continue forwarding traffic.

This runbook focuses on configuring the failover relationship, dedicated failover and state links, production-interface standby addressing, configuration synchronization, and controlled failover validation between two Cisco ASAv firewalls.

Routed firewall interfaces, static routing, and PAT are configured separately before failover is introduced.

---

## Scope and Assumptions

This runbook assumes:

- Two Cisco ASAv firewalls are available for an active/standby pair.
- The required ASAv baselines have already been applied to both firewalls.
- The ASAv Routed Firewall and PAT runbook has already been applied and validated on the intended primary firewall.
- The failover link and state-link interfaces have been identified.
- Active and standby IP addressing has been defined for production firewall interfaces.
- Failover-link and state-link addressing has been defined.
- Both ASAv units are running compatible software versions and platform settings.
- The intended primary firewall contains the authoritative configuration for initial synchronization.
- Adjacent Layer 2 and Layer 3 devices are configured to support both firewall units.
- Firewall policy, routing, and NAT design are handled separately from failover.

This runbook does not cover multi-context failover, clustering, active/active failover, VPN high availability, dynamic routing failover behavior, or upstream/downstream redundancy outside the firewall pair.

---

## Reference Design

This runbook uses two Cisco ASAv firewalls configured as an active/standby failover pair.

FW1 is configured as the primary unit and is expected to operate as active during normal operation. FW2 is configured as the secondary unit and is expected to remain standby ready unless a failover event occurs.

Primary and secondary identify the configured failover units. Active and standby identify their current operational states.

### Devices

| Device | Role                    | Purpose                                             |
|--------|-------------------------|-----------------------------------------------------|
| FW1    | Primary ASAv firewall   | Normal active unit and initial configuration source |
| FW2    | Secondary ASAv firewall | Normal standby unit and failover peer               |

### Failover Link Summary

| Device | Interface | Purpose       | Failover IP    |
|--------|-----------|---------------|----------------|
| FW1    | Gi0/2     | Failover link | 192.0.2.1/30   |
| FW2    | Gi0/2     | Failover link | 192.0.2.2/30   |

### State Link Summary

| Device | Interface | Purpose       | State IP       |
|--------|-----------|---------------|----------------|
| FW1    | Gi0/3     | Stateful link | 192.0.2.5/30   |
| FW2    | Gi0/3     | Stateful link | 192.0.2.6/30   |

### Production Interface Addressing

| Interface | Active Address | Standby Address |
|-----------|----------------|-----------------|
| outside   | 10.0.0.2/29    | 10.0.0.3/29     |
| inside    | 10.0.1.1/29    | 10.0.1.2/29     |

### Failover Roles

| Device | Failover Role | Expected Normal State |
|--------|---------------|-----------------------|
| FW1    | Primary       | Active                |
| FW2    | Secondary     | Standby Ready         |

---

## Prerequisites and Pre-Checks

Before configuring ASAv active/standby failover, confirm that both firewalls and the surrounding network are ready for high-availability configuration.

### Prerequisites

- [ ] Both ASAv firewalls are powered on and accessible for configuration.
- [ ] The required ASAv baselines have already been applied to both firewalls.
- [ ] The ASAv Routed Firewall and PAT runbook has already been applied and validated on the intended primary firewall.
- [ ] The failover and state-link interfaces have been identified.
- [ ] Failover-link and state-link IPv4 addressing has been defined.
- [ ] Active and standby IPv4 addresses have been defined for production interfaces.
- [ ] Both ASAv units are running compatible software versions and platform settings.
- [ ] Adjacent switching and routing infrastructure supports connectivity for both firewall units.
- [ ] No conflicting failover configuration is already present.

### Baseline Verification

Run these commands on both ASAv firewalls before applying failover configuration.

```bash
show running-config hostname
show interface ip brief
show nameif
show failover
show running-config failover
```

### Expected Baseline Results

- [ ] Each firewall has the expected hostname.
- [ ] The intended primary firewall's inside and outside interfaces match the validated routed firewall configuration.
- [ ] The failover and state-link interfaces are present and available for failover configuration.
- [ ] The intended primary firewall has the expected routed-interface IPv4 addressing.
- [ ] No unexpected failover relationship is currently active.
- [ ] No conflicting failover commands are present.
- [ ] Both firewalls are ready for active/standby pairing.

---

## Configuration Procedure

Use this procedure to configure an ASAv active/standby failover pair with dedicated failover and state links.

### Configuration Notes

- Confirm which firewall will operate as the primary unit before enabling failover.
- The intended primary firewall should contain the validated routed firewall, routing, NAT, and policy configuration before synchronization begins.
- The secondary unit requires only the failover bootstrap configuration needed to establish communication with the primary unit.
- Once the failover relationship forms, the primary configuration synchronizes to the secondary unit.
- Configure a standby IPv4 address for each production data interface participating in failover.
- Dedicated interfaces are used for the failover LAN link and state link in this reference design.
- Physical data interfaces are monitored for failover health by default.
- Do not independently maintain shared production configuration on both units after failover is established.
- Replace example interface numbers and IPv4 addresses with those defined for the target deployment.

---

### Step 1 — Configure Production Interface Standby Addresses

Run on FW1.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

interface GigabitEthernet0/0 # Target the outside interface
ip address 10.0.0.2 255.255.255.248 standby 10.0.0.3 # Configure active and standby outside addresses

interface GigabitEthernet0/1 # Target the inside interface
ip address 10.0.1.1 255.255.255.248 standby 10.0.1.2 # Configure active and standby inside addresses

end # Return to privileged EXEC mode
```

---

### Step 2 — Configure FW1 as the Primary Failover Unit

Run on FW1.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

failover lan unit primary # Define FW1 as the primary failover unit

failover lan interface FAILOVER GigabitEthernet0/2 # Assign Gi0/2 as the failover LAN link
failover interface ip FAILOVER 192.0.2.1 255.255.255.252 standby 192.0.2.2 # Configure failover-link addressing

interface GigabitEthernet0/2 # Target the failover LAN interface
no shutdown # Administratively enable the failover interface

failover link STATE GigabitEthernet0/3 # Assign Gi0/3 as the state synchronization link
failover interface ip STATE 192.0.2.5 255.255.255.252 standby 192.0.2.6 # Configure state-link addressing

interface GigabitEthernet0/3 # Target the state-link interface
no shutdown # Administratively enable the state-link interface

failover # Enable failover

end # Return to privileged EXEC mode
write memory # Save the primary firewall configuration
```

---

### Step 3 — Configure FW2 as the Secondary Failover Unit

Run on FW2.

```bash
enable # Enter privileged EXEC mode
configure terminal # Enter global configuration mode

failover lan unit secondary # Define FW2 as the secondary failover unit

failover lan interface FAILOVER GigabitEthernet0/2 # Assign Gi0/2 as the failover LAN link
failover interface ip FAILOVER 192.0.2.1 255.255.255.252 standby 192.0.2.2 # Configure failover-link addressing

interface GigabitEthernet0/2 # Target the failover LAN interface
no shutdown # Administratively enable the failover interface

failover link STATE GigabitEthernet0/3 # Assign Gi0/3 as the state synchronization link
failover interface ip STATE 192.0.2.5 255.255.255.252 standby 192.0.2.6 # Configure state-link addressing

interface GigabitEthernet0/3 # Target the state-link interface
no shutdown # Administratively enable the state-link interface

failover # Enable failover

end # Return to privileged EXEC mode

# Do not write memory until initial configuration synchronization has been verified.
```

---

## Post-Configuration Validation

Use this section to confirm that the ASAv failover pair has formed correctly, configuration synchronization has completed, and both units are in the expected active/standby state.

### Step 1 — Verify Failover Pair State

Run on both ASAv firewalls.

```bash
show failover
```

Expected results:

- [ ] Failover is enabled.
- [ ] FW1 identifies itself as the primary unit.
- [ ] FW2 identifies itself as the secondary unit.
- [ ] FW1 is active during normal operation.
- [ ] FW2 is standby ready.
- [ ] Both units report the peer as reachable.
- [ ] No failed or unresolved failover state is present.

---

### Step 2 — Verify Failover and State Links

Run on both ASAv firewalls.

```bash
show failover
show interface ip brief
```

Expected results:

- [ ] The failover LAN link is operational.
- [ ] The state synchronization link is operational.
- [ ] The failover-link addresses match the reference design.
- [ ] The state-link addresses match the reference design.
- [ ] No failover or state-link interface is administratively down.
- [ ] No unexpected interface failure is reported.

---

### Step 3 — Verify Production Interface Addressing

Run on both ASAv firewalls.

```bash
show failover
show interface ip brief
```

Expected results:

- [ ] The active unit uses outside address `10.0.0.2`.
- [ ] The standby unit uses outside address `10.0.0.3`.
- [ ] The active unit uses inside address `10.0.1.1`.
- [ ] The standby unit uses inside address `10.0.1.2`.
- [ ] Production interfaces report normal status.

---

### Step 4 — Verify Configuration Synchronization

Run on both ASAv firewalls.

```bash
show failover
show running-config failover
show running-config
```

Expected results:

- [ ] Both units report a healthy failover relationship.
- [ ] FW2 has received the synchronized production configuration.
- [ ] Failover interface assignments and addressing match on both units.
- [ ] Shared production configuration is synchronized between the units.
- [ ] No configuration synchronization failure is reported.
- [ ] The pair is ready for controlled failover testing.

---

### Step 5 — Save the Synchronized Secondary Configuration

Run on FW2 after initial configuration synchronization has completed successfully.

```bash
write memory # Save the synchronized configuration to startup configuration
```

Expected results:

- [ ] FW2 saves the synchronized configuration successfully.
- [ ] The failover pair remains in the expected active/standby state.

---

## Failover Validation

Use this section to confirm that the standby firewall can assume the active role and that the failover pair can be returned to the intended normal operating state.

Perform controlled failover testing only in a lab or approved maintenance window.

### Step 1 — Confirm Initial Failover State

Run on both ASAv firewalls.

```bash
show failover
```

Expected results:

- [ ] FW1 is the primary unit and is active.
- [ ] FW2 is the secondary unit and is standby ready.
- [ ] Both units report normal communication with the failover peer.
- [ ] The failover and state links are operational.
- [ ] No monitored interface failure is present.

---

### Step 2 — Force FW2 Active

Run on FW2.

```bash
failover active
```

Expected results:

- [ ] FW2 transitions from standby to active.
- [ ] FW1 transitions from active to standby.
- [ ] The failover pair remains operational during the role transition.
- [ ] No unexpected failed state is introduced.

---

### Step 3 — Verify Failover State

Run on both ASAv firewalls.

```bash
show failover
```

Expected results:

- [ ] FW2 is active.
- [ ] FW1 is standby ready.
- [ ] Production interfaces report normal status.
- [ ] Failover and state-link communication remain operational.
- [ ] The pair remains synchronized after the role transition.

---

### Step 4 — Restore FW1 as Active

Run on FW1.

```bash
failover active
```

Expected results:

- [ ] FW1 returns to the active role.
- [ ] FW2 returns to standby-ready state.
- [ ] Failover and state links remain operational.
- [ ] No unexpected interface or synchronization failure occurs.

---

### Step 5 — Verify Final Failover State

Run on both ASAv firewalls.

```bash
show failover
```

Expected results:

- [ ] FW1 is the primary active unit.
- [ ] FW2 is the secondary standby-ready unit.
- [ ] Both units report a healthy failover relationship.
- [ ] The final state matches the intended reference design.

---

## If Validation Fails

If post-configuration or failover validation does not produce the expected results:

- Identify which validation step failed.
- Confirm that both ASAv units are configured with the correct primary and secondary failover roles.
- Confirm that the failover LAN and state-link interfaces are operational.
- Confirm that failover-link and state-link addressing match on both units.
- Confirm that active and standby addresses are correctly defined for production interfaces.
- Confirm that both units are running compatible software versions and platform settings.
- Review `show failover` output for failed interfaces, communication failures, or synchronization errors.
- Confirm that FW1 contains the intended production configuration and that FW2 has synchronized successfully.
- Correct the affected configuration.
- Re-run the failed validation step.
- If the issue cannot be resolved, return the firewall pair to its previous known-good standalone or failover state before continuing.

---

## Backout Considerations

Changing or removing active/standby failover configuration can interrupt firewall redundancy and may affect production traffic if performed while the pair is carrying active sessions.

Before removing or changing the configuration, confirm:

- The current active and standby states are documented.
- The primary and secondary unit assignments are documented.
- The failover LAN and state-link configuration is documented.
- Active and standby production-interface addressing is documented.
- The current production configuration on the active firewall is backed up or otherwise recoverable.
- The intended standalone or previous failover state is understood.
- Any traffic or services dependent on firewall redundancy are identified.
- The firewall pair can be returned to its previous known-good state if required.
- Adjacent devices can continue forwarding traffic if the pair is temporarily reduced to a single operational firewall.

For lab use, restore the last known working failover configuration before continuing additional validation.

---

## Document Metadata

| Field            | Value                                      |
|------------------|--------------------------------------------|
| Author           | Aaron Kindelt                              |
| Category         | Configuration / High Availability          |
| Technology       | Cisco ASAv Active/Standby Failover         |
| Applies To       | Cisco ASAv firewall pairs                  |
| Primary Use Case | Stateful active/standby firewall failover  |
| Version          | 1.0                                        |
| Last Updated     | 2026-09-20                                 |