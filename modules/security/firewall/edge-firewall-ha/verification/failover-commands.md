# Edge Firewall HA Failover Test Commands

This file contains the failover-test procedure and recorded output for the Edge Firewall HA lab.

The tests below validate manual firewall switchover, automatic failover after monitored-interface and node failures, HSRP convergence, continued end-to-end forwarding, and restoration of the preferred baseline state.

**Perform these tests only after the normal verification document passes. Restore the topology to its expected operating state before beginning each test.**

## CML Long-Ping Control

Before starting a long-running ping on A1:

```text
A1#terminal escape-character 33
```

- Press `!` to stop the ping after the failure and recovery sequence is complete.


---

## Test 1: Manual Firewall Switchover

**Purpose:** Confirm FW2 can assume the Active role, maintain end-to-end forwarding, and return to the original firewall roles.

### Traffic Monitor — `ping 192.0.2.100 repeat 100000`

**Run on:** A1

**Expected Results**

- [x] A1 maintains or resumes end-to-end reachability across the role transitions.
- [x] The final statistics record any transient packet loss during switchover and restoration.

```text
A1#ping 192.0.2.100 repeat 100000
Success rate is 99 percent (11944/11949), round-trip min/avg/max = 4/8/127 ms
```

### Manual Switchover — `no failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 relinquishes the Active role and FW2 assumes the Active role.

```text
FW-HA/pri/act# no failover active

FW-HA/pri/act# 
        Switching to Standby
```

### Post-Switchover Role Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Standby Ready`.
- [x] FW2 reports `Secondary - Active`.

```text
FW-HA/pri/stby# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 20:51:15 UTC Aug 8 2026
        This host: Primary - Standby Ready 
                Active time: 387 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Secondary - Active 
                Active time: 23 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### Restore Preferred Active Unit — `failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 resumes the Active role.

```text
FW-HA/pri/stby# failover active

        Switching to Active
```

### Final Role Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Active`.
- [x] FW2 reports `Secondary - Standby Ready`.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 20:52:09 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 21 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 54 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

---

## Test 2: FW1 Inside-Link Failure

**Purpose:** Confirm failure of FW1's monitored inside path automatically moves the Active role to FW2, maintains end-to-end forwarding after convergence, and allows FW1 to recover to Standby Ready after the failed path is restored.

### Traffic Monitor — `ping 192.0.2.100 repeat 100000`

**Run on:** A1

**Expected Results**

- [x] A1 maintains or resumes end-to-end reachability after automatic failover.
- [x] The final statistics record any transient packet loss during failure and recovery.

```text
A1#ping 192.0.2.100 repeat 100000
Success rate is 99 percent (27429/27439), round-trip min/avg/max = 5/9/721 ms
```

### Induce Inside-Link Failure — D1 `Gi0/0` Shutdown

**Run on:** D1

**Expected Results**

- [x] D1 `Gi0/0` transitions down, causing FW1 to lose its monitored `inside` path.

```text
D1#configure terminal
D1(config)#interface gigabitethernet0/0
D1(config-if)#shutdown
D1(config-if)#end
D1(config-if)#
*Aug  8 22:46:34.237: %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to administratively down
*Aug  8 22:46:35.237: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to down
```

### Automatic Failover Check — `show failover`

**Run on:** FW2

**Expected Results**

- [x] FW2 reports `Secondary - Active`.
- [x] FW1 reports `Primary - Failed`.
- [x] FW1's `inside` interface reports the failed or no-traffic condition associated with the inside-path failure.

```text
FW-HA/sec/act# show failover
Failover On 
Failover unit Secondary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ARTESW8ALK, Mate 9ABP74674HW
Last Failover at: 22:48:34 UTC Aug 8 2026
        This host: Secondary - Active 
                Active time: 242 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Primary - Failed 
                Active time: 453 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): No Traffic (Waiting)
```

### Restore Inside Link — D1 `Gi0/0` No Shutdown

**Run on:** D1

**Expected Results**

- [x] D1 `Gi0/0` returns to an operational state.

```text
D1#configure terminal
D1(config)#interface gigabitethernet0/0
D1(config-if)#no shutdown
D1(config-if)#end
D1(config-if)#
*Aug  8 22:50:30.338: %LINK-3-UPDOWN: Interface GigabitEthernet0/0, changed state to up
*Aug  8 22:50:31.338: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up
```

### FW1 Recovery Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 automatically recovers to `Primary - Standby Ready` after the inside path is restored.
- [x] FW2 remains `Secondary - Active`.
- [x] FW1's monitored `inside` and `outside` interfaces report `Normal (Monitored)`.

```text
FW-HA/pri/stby# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 22:48:34 UTC Aug 8 2026
        This host: Primary - Standby Ready 
                Active time: 453 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Secondary - Active 
                Active time: 372 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### Restore Preferred Active Unit — `failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 resumes the Active role after automatic recovery is complete.

```text
FW-HA/pri/stby# failover active

        Switching to Active
```

### Final Role Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Active`.
- [x] FW2 reports `Secondary - Standby Ready`.
- [x] The monitored `inside` and `outside` interfaces are healthy on both units.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 22:55:03 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 55 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 390 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

---

## Test 3: FW1 Outside-Link Failure

**Purpose:** Confirm failure of FW1's monitored outside path automatically moves the Active role to FW2, maintains end-to-end forwarding after convergence, and allows FW1 to recover to Standby Ready after the failed path is restored.

### Traffic Monitor — `ping 192.0.2.100 repeat 100000`

**Run on:** A1

**Expected Results**

- [x] A1 maintains or resumes end-to-end reachability after automatic failover.
- [x] The final statistics record any transient packet loss during failure and recovery.

```text
A1#ping 192.0.2.100 repeat 100000
Success rate is 99 percent (23818/23832), round-trip min/avg/max = 5/9/214 ms
```

### Induce Outside-Link Failure — OS1 `Gi0/1` Shutdown

**Run on:** OS1

**Expected Results**

- [x] OS1 `Gi0/1` transitions down, causing FW1 to lose its monitored `outside` path.

```text
OS1#configure terminal
OS1(config)#interface gigabitethernet0/1
OS1(config-if)#shutdown
OS1(config-if)#end
OS1(config-if)#
*Aug  8 23:03:16.543: %LINK-5-CHANGED: Interface GigabitEthernet0/1, changed state to administratively down
*Aug  8 23:03:17.543: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to down
```

### Automatic Failover Check — `show failover`

**Run on:** FW2

**Expected Results**

- [x] FW2 reports `Secondary - Active`.
- [x] FW1 reports `Primary - Failed`.
- [x] FW1's `outside` interface reports the failed condition associated with the outside-path failure.

```text
FW-HA/sec/act# show failover
Failover On 
Failover unit Secondary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ARTESW8ALK, Mate 9ABP74674HW
Last Failover at: 23:05:54 UTC Aug 8 2026
        This host: Secondary - Active 
                Active time: 45 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Waiting)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Primary - Failed 
                Active time: 645 (sec)
                  Interface outside (203.0.113.3): Failed (Waiting)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

### Restore Outside Link — OS1 `Gi0/1` No Shutdown

**Run on:** OS1

**Expected Results**

- [x] OS1 `Gi0/1` returns to an operational state.

```text
OS1#configure terminal
OS1(config)#interface gigabitethernet0/1
OS1(config-if)#no shutdown
OS1(config-if)#end
OS1(config-if)#
*Aug  8 23:05:06.566: %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to up
*Aug  8 23:05:07.570: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/1, changed state to up
```

### FW1 Recovery Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 automatically recovers to `Primary - Standby Ready` after the outside path is restored.
- [x] FW2 remains `Secondary - Active`.
- [x] FW1's monitored `inside` and `outside` interfaces report `Normal (Monitored)`.

```text
FW-HA/pri/stby# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 23:05:54 UTC Aug 8 2026
        This host: Primary - Standby Ready 
                Active time: 645 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Secondary - Active 
                Active time: 148 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### Restore Preferred Active Unit — `failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 resumes the Active role after automatic recovery is complete.

```text
FW-HA/pri/stby# failover active

        Switching to Active
```

### Final Role Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Active`.
- [x] FW2 reports `Secondary - Standby Ready`.
- [x] The monitored `inside` and `outside` interfaces are healthy on both units.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 23:08:51 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 21 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 177 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

---

## Test 4: FW1 Node Failure

**Purpose:** Confirm complete loss of the Active firewall automatically moves the Active role to FW2, maintains end-to-end forwarding, and allows FW1 to rejoin as the Standby unit after recovery.

### Traffic Monitor — `ping 192.0.2.100 repeat 100000`

**Run on:** A1

**Expected Results**

- [x] A1 maintains or resumes end-to-end reachability after FW2 assumes the Active role.
- [x] The final statistics record any transient packet loss during firewall failure and recovery.

```text
A1#ping 192.0.2.100 repeat 100000
Success rate is 99 percent (70383/70399), round-trip min/avg/max = 3/9/186 ms
```

### Stop FW1 Node

**Run in:** CML

**Expected Results**

- [x] FW1 stops completely and FW2 detects loss of the Active peer.

```text
CML action: Stop the FW1 node
```

### FW2 Role Check — `show failover`

**Run on:** FW2

**Expected Results**

- [x] FW2 reports `Secondary - Active`.
- [x] FW1 reports `Primary - Failed`.
- [x] FW2's active data interfaces may report `Normal (Waiting)` while FW1 is unavailable.

```text
FW-HA/sec/act# show failover
Failover On 
Failover unit Secondary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ARTESW8ALK, Mate 9ABP74674HW
Last Failover at: 22:22:20 UTC Aug 8 2026
        This host: Secondary - Active 
                Active time: 96 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Waiting)
                  Interface inside (10.255.0.4): Normal (Waiting)
        Other host: Primary - Failed 
                Active time: 528 (sec)
                  Interface outside (203.0.113.3): Unknown (Monitored)
                  Interface inside (10.255.0.5): Unknown (Monitored)
```

### Start FW1 Node

**Run in:** CML

**Expected Results**

- [x] FW1 boots and rejoins the failover pair.

```text
CML action: Start the FW1 node
```

### FW1 Rejoin Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Standby Ready`.
- [x] FW2 remains `Secondary - Active`.
- [x] The monitored `inside` and `outside` interfaces return to `Normal (Monitored)`.

```text
FW-HA/pri/stby# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 22:30:17 UTC Aug 8 2026
        This host: Primary - Standby Ready 
                Active time: 0 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Secondary - Active 
                Active time: 575 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### Restore Preferred Active Unit — `failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 resumes the Active role.

```text
FW-HA/pri/stby# failover active

        Switching to Active
```

### Final Role Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Active`.
- [x] FW2 reports `Secondary - Standby Ready`.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 22:32:24 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 68 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 603 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

---

## Test 5: D1 Node Failure

**Purpose:** Confirm complete loss of D1 moves traffic onto the surviving FW2-D2 path, causes the expected firewall and HSRP convergence, and preserves end-to-end forwarding.

### Traffic Monitor — `ping 192.0.2.100 repeat 100000`

**Run on:** A1

**Expected Results**

- [x] A1 maintains or resumes end-to-end reachability after convergence.
- [x] The final statistics record any transient packet loss during distribution-layer failure and recovery.

```text
A1#ping 192.0.2.100 repeat 100000
Success rate is 99 percent (46937/46999), round-trip min/avg/max = 4/8/158 ms
```

### Stop D1 Node

**Run in:** CML

**Expected Results**

- [x] D1 stops completely and the topology begins convergence onto the surviving FW2-D2 path.

```text
CML action: Stop the D1 node
```

### Firewall Failover Check — `show failover`

**Run on:** FW2

**Expected Results**

- [x] FW2 reports `Secondary - Active` after FW1 loses its inside path.
- [x] FW1 reports `Primary - Failed`.

```text
FW-HA/sec/act# show failover
Failover On 
Failover unit Secondary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ARTESW8ALK, Mate 9ABP74674HW
Last Failover at: 22:35:52 UTC Aug 8 2026
        This host: Secondary - Active 
                Active time: 24 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Waiting)
        Other host: Primary - Failed 
                Active time: 206 (sec)
                  Interface outside (203.0.113.3): Normal (Waiting)
                  Interface inside (10.255.0.5): Failed (Waiting)
```

### D2 HSRP Check — `show standby brief`

**Run on:** D2

**Expected Results**

- [x] D2 reports `Active` for HSRP groups `10` and `99`.

```text
D2#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   100 P Active  local           unknown         10.10.10.1
Vl99        99   100 P Active  local           unknown         10.255.0.1
```

### Start D1 Node

**Run in:** CML

**Expected Results**

- [x] D1 boots and rejoins the topology.

```text
CML action: Start the D1 node
```

### D1 HSRP Restoration Check — `show standby brief`

**Run on:** D1

**Expected Results**

- [x] D1 reclaims the `Active` role for HSRP groups `10` and `99` through preemption.

```text
D1#show standby brief
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State   Active          Standby         Virtual IP
Vl10        10   110 P Active  local           10.10.10.3      10.10.10.1
Vl99        99   110 P Active  local           10.255.0.3      10.255.0.1
```

### FW1 Rejoin Check — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 automatically recovers to `Primary - Standby Ready` after its inside path is restored.
- [x] FW2 remains `Secondary - Active`.
- [x] The monitored `inside` and `outside` interfaces return to `Normal (Monitored)`.

```text
FW-HA/pri/stby# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 22:35:52 UTC Aug 8 2026
        This host: Primary - Standby Ready 
                Active time: 206 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
        Other host: Secondary - Active 
                Active time: 275 (sec)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
```

### Restore Preferred Active Unit — `failover active`

**Run on:** FW1

**Expected Results**

- [x] FW1 resumes the Active role after automatic recovery is complete.

```text
FW-HA/pri/stby# failover active

        Switching to Active
```

---

## Final Restoration and Baseline Confirmation

**Purpose:** Confirm the preferred operating state has been restored after all failover tests.

### Firewall Baseline — `show failover`

**Run on:** FW1

**Expected Results**

- [x] FW1 reports `Primary - Active`.
- [x] FW2 reports `Secondary - Standby Ready`.
- [x] The monitored `inside` and `outside` interfaces report `Normal (Monitored)` on both units.

```text
FW-HA/pri/act# show failover
Failover On 
Failover unit Primary
Failover LAN Interface: FAILOVER GigabitEthernet0/1 (up)
Reconnect timeout 0:00:00
Unit Poll frequency 1 seconds, holdtime 15 seconds
Interface Poll frequency 5 seconds, holdtime 25 seconds
Interface Policy 1
Monitored Interfaces 2 of 311 maximum
MAC Address Move Notification Interval not set
Version: Ours 9.23(1), Mate 9.23(1)
Serial Number: Ours 9ABP74674HW, Mate 9ARTESW8ALK
Last Failover at: 23:08:51 UTC Aug 8 2026
        This host: Primary - Active 
                Active time: 215 (sec)
                slot 0: ASAv hw/sw rev (/9.23(1)) status (Up Sys)
                  Interface outside (203.0.113.2): Normal (Monitored)
                  Interface inside (10.255.0.4): Normal (Monitored)
        Other host: Secondary - Standby Ready 
                Active time: 177 (sec)
                  Interface outside (203.0.113.3): Normal (Monitored)
                  Interface inside (10.255.0.5): Normal (Monitored)
```

### End-to-End Baseline — `ping 192.0.2.100`

**Run on:** A1

**Expected Results**

- [x] A1 successfully reaches `192.0.2.100` after all failure scenarios are complete.

```text
A1#ping 192.0.2.100
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.0.2.100, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 6/6/7 ms
```