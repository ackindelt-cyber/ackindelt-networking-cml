# Basic HSRP Verification Output

This file contains recorded verification output for the Basic HSRP lab.

The checks below confirm interface state, HSRP active and standby roles, virtual IP address, virtual MAC address, default timers, preemption, client connectivity to the virtual gateway, failover to R2, and recovery back to R1.

---

## R1

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up.
* [x] `GigabitEthernet0/0` has IP address `192.168.10.2`.

```text
R1>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         192.168.10.2    YES NVRAM  up                    up      
GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down   
```

### `show standby`

**Expected Results**

* [x] HSRP state is `Active`.
* [x] HSRP virtual IP address is `192.168.10.1`.
* [x] HSRP virtual MAC address is `0000.0c07.ac0a`.
* [x] Hello timer is 3 seconds.
* [x] Hold timer is 10 seconds.
* [x] Preemption is enabled.
* [x] Standby router is `192.168.10.3`.
* [x] Standby router priority is `100`.
* [x] Local priority is `110`.
* [x] Group name is `HSRP-GROUP10`.

```text
R1>show standby
GigabitEthernet0/0 - Group 10
  State is Active
    2 state changes, last state change 00:09:00
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac0a
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.432 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.10.3, priority 100 (expires in 9.840 sec)
  Priority 110 (configured 110)
  Group name is "HSRP-GROUP10" (cfgd)
```

---

## R2

### `show ip interface brief`

**Expected Results**

* [x] `GigabitEthernet0/0` is up/up.
* [x] `GigabitEthernet0/0` has IP address `192.168.10.3`.

```text
R2>show ip interface brief
Interface                  IP-Address      OK? Method Status                Protocol
GigabitEthernet0/0         192.168.10.3    YES NVRAM  up                    up      
GigabitEthernet0/1         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/2         unassigned      YES NVRAM  administratively down down    
GigabitEthernet0/3         unassigned      YES NVRAM  administratively down down   
```

### `show standby`

**Expected Results**

* [x] HSRP state is `Standby`.
* [x] HSRP virtual IP address is `192.168.10.1`.
* [x] HSRP virtual MAC address is `0000.0c07.ac0a`.
* [x] Hello timer is 3 seconds.
* [x] Hold timer is 10 seconds.
* [x] Preemption is enabled.
* [x] Active router is `192.168.10.2`.
* [x] Active router priority is `110`.
* [x] Local priority is `100`.
* [x] Group name is `HSRP-GROUP10`.

```text
R2>show standby
GigabitEthernet0/0 - Group 10
  State is Standby
    1 state change, last state change 00:13:05
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac0a
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.264 secs
  Preemption enabled
  Active router is 192.168.10.2, priority 110 (expires in 10.272 sec)
  Standby router is local
  Priority 100 (default 100)
  Group name is "HSRP-GROUP10" (cfgd)
```

---

## S1

### `show mac address-table | include 0000.0c07.ac0a`

**Expected Results**

* [x] HSRP virtual MAC address `0000.0c07.ac0a` is present in the MAC address table.
* [x] The virtual MAC address is learned on the active-router path.

```text
S1#show mac address-table | include 0000.0c07.ac0a
   1    0000.0c07.ac0a    DYNAMIC     Gi0/0
```

---

## C1

### `ping -c 5 192.168.10.1`

**Expected Results**

* [x] C1 receives ICMP replies from the HSRP virtual gateway.
* [x] Initial client connectivity to the VIP is successful.

```text
C1:~# ping -c 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1): 56 data bytes
64 bytes from 192.168.10.1: seq=0 ttl=255 time=1.820 ms
64 bytes from 192.168.10.1: seq=1 ttl=255 time=1.776 ms
64 bytes from 192.168.10.1: seq=2 ttl=255 time=1.965 ms
64 bytes from 192.168.10.1: seq=3 ttl=255 time=1.923 ms
64 bytes from 192.168.10.1: seq=4 ttl=255 time=1.879 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.776/1.872/1.965 ms
```

---

## Simulated R1 Failure

### `show standby` on R2 after R1 failure

**Expected Results**

* [x] R2 becomes active for HSRP group 10.
* [x] R2 owns the virtual IP address during the failure.
* [x] Standby router is unknown because R1 is unavailable.

```text
R2>show standby
GigabitEthernet0/0 - Group 10
  State is Active
    2 state changes, last state change 00:00:34
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac0a
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 1.984 secs
  Preemption enabled
  Active router is local
  Standby router is unknown
  Priority 100 (default 100)
  Group name is "HSRP-GROUP10" (cfgd)
```

### `ping -c 5 192.168.10.1` from C1 after R1 failure

**Expected Results**

* [x] C1 receives ICMP replies from the HSRP virtual gateway after failover.
* [x] Client connectivity to the VIP remains successful.

```text
C1:~# ping -c 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1): 56 data bytes
64 bytes from 192.168.10.1: seq=0 ttl=255 time=1.820 ms
64 bytes from 192.168.10.1: seq=1 ttl=255 time=1.776 ms
64 bytes from 192.168.10.1: seq=2 ttl=255 time=1.965 ms
64 bytes from 192.168.10.1: seq=3 ttl=255 time=1.923 ms
64 bytes from 192.168.10.1: seq=4 ttl=255 time=1.879 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.776/1.872/1.965 ms
```

---

## R1 Recovery

### `show standby` on R1 after recovery

**Expected Results**

* [x] R1 returns to active state.
* [x] R1 resumes the active role because it has higher priority and preemption enabled.
* [x] R2 returns to standby state.

```text
R1#show standby
GigabitEthernet0/0 - Group 10
  State is Active
    4 state changes, last state change 00:00:27
  Virtual IP address is 192.168.10.1
  Active virtual MAC address is 0000.0c07.ac0a
    Local virtual MAC address is 0000.0c07.ac0a (v1 default)
  Hello time 3 sec, hold time 10 sec
    Next hello sent in 2.448 secs
  Preemption enabled
  Active router is local
  Standby router is 192.168.10.3, priority 100 (expires in 9.616 sec)
  Priority 110 (configured 110)
  Group name is "HSRP-GROUP10" (cfgd)
```

### `ping -c 5 192.168.10.1` from C1 after R1 recovery

**Expected Results**

* [x] C1 receives ICMP replies from the HSRP virtual gateway after R1 recovery.
* [x] Client connectivity to the VIP remains successful.

```text
C1:~# ping -c 5 192.168.10.1
PING 192.168.10.1 (192.168.10.1): 56 data bytes
64 bytes from 192.168.10.1: seq=0 ttl=255 time=1.820 ms
64 bytes from 192.168.10.1: seq=1 ttl=255 time=1.776 ms
64 bytes from 192.168.10.1: seq=2 ttl=255 time=1.965 ms
64 bytes from 192.168.10.1: seq=3 ttl=255 time=1.923 ms
64 bytes from 192.168.10.1: seq=4 ttl=255 time=1.879 ms

--- 192.168.10.1 ping statistics ---
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 1.776/1.872/1.965 ms
```
