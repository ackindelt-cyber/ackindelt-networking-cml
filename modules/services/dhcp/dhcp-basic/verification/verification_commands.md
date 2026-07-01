# Basic DHCP Verification Output

This file contains recorded verification output for the Basic DHCP lab.

The checks below confirm DHCP bindings, DHCP pool status, DHCP configuration presence, dynamic client addressing, and inter-VLAN connectivity between DHCP clients.

---

## R1

### `show ip dhcp binding`

**Expected Results**

* [x] DHCP binding exists for VLAN 10 client.
* [x] DHCP binding exists for VLAN 20 client.
* [x] Leased addresses are from the expected subnets.
* [x] Lease type is automatic.

```text
R1#show ip dhcp binding
Bindings from all pools not associated with VRF:
IP address          Client-ID/              Lease expiration        Type
                    Hardware address/
                    User name
192.168.10.22       ff32.39f9.b500.0200.    Dec 23 2025 02:56 AM    Automatic
                    00ab.11ef.ce86.319a.
                    273b.9b
192.168.20.22       ff32.39f9.b500.0200.    Dec 23 2025 02:59 AM    Automatic
                    00ab.116d.3d3d.5584.
                    9608.b4
```

### `show ip dhcp pool`

**Expected Results**

* [x] DHCP pool `VLAN10` exists.
* [x] DHCP pool `VLAN20` exists.
* [x] Each pool has one leased address.
* [x] Address ranges align with the configured subnets.
* [x] No pending DHCP events are present.

```text
R1#show ip dhcp pool

Pool VLAN10 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.10.23        192.168.10.1     - 192.168.10.254    1

Pool VLAN20 :
 Utilization mark (high/low)    : 100 / 0
 Subnet size (first/next)       : 0 / 0 
 Total addresses                : 254
 Leased addresses               : 1
 Pending event                  : none
 1 subnet is currently in the pool :
 Current index        IP address range                    Leased addresses
 192.168.20.23        192.168.20.1     - 192.168.20.254    1
```

**Note:** The pool output shows the full subnet range. The excluded addresses are confirmed separately in the running configuration and are not available for assignment.

### `show running-config | include dhcp`

**Expected Results**

* [x] VLAN 10 excluded range is present.
* [x] VLAN 20 excluded range is present.
* [x] DHCP pool `VLAN10` exists.
* [x] DHCP pool `VLAN20` exists.
* [x] No unintended DHCP pools are shown.

```text
R1#show running-config | include dhcp
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp excluded-address 192.168.20.1 192.168.20.20
ip dhcp pool VLAN10
ip dhcp pool VLAN20
```

---

## C1

### `ip a`

**Expected Results**

* [x] Client receives address from `192.168.10.0/24`.
* [x] Address is assigned to interface `ens2`.
* [x] Address is marked dynamic.

```text
cisco@C1:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:4c:37:bb brd ff:ff:ff:ff:ff:ff
    altname enp0s2
    inet 192.168.10.22/24 metric 100 brd 192.168.10.255 scope global dynamic ens2
       valid_lft 86220sec preferred_lft 86220sec
    inet6 fe80::5054:ff:fe4c:37bb/64 scope link 
       valid_lft forever preferred_lft forever
```

### `ping -w 5 192.168.20.22`

**Expected Results**

* [x] C1 can reach C2.
* [x] Inter-VLAN connectivity succeeds using DHCP-assigned addresses.

```text
cisco@C1:~$ ping -w 5 192.168.20.22
PING 192.168.20.22 (192.168.20.22) 56(84) bytes of data.
64 bytes from 192.168.20.22: icmp_seq=1 ttl=63 time=2.20 ms
64 bytes from 192.168.20.22: icmp_seq=2 ttl=63 time=2.57 ms
64 bytes from 192.168.20.22: icmp_seq=3 ttl=63 time=2.44 ms
64 bytes from 192.168.20.22: icmp_seq=4 ttl=63 time=2.38 ms
64 bytes from 192.168.20.22: icmp_seq=5 ttl=63 time=2.23 ms
```

---

## C2

### `ip a`

**Expected Results**

* [x] Client receives address from `192.168.20.0/24`.
* [x] Address is assigned to interface `ens2`.
* [x] Address is marked dynamic.

```text
cisco@C2:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: ens2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:54:e3:35 brd ff:ff:ff:ff:ff:ff
    altname enp0s2
    inet 192.168.20.22/24 metric 100 brd 192.168.20.255 scope global dynamic ens2
       valid_lft 86354sec preferred_lft 86354sec
    inet6 fe80::5054:ff:fe54:e335/64 scope link 
       valid_lft forever preferred_lft forever
```

### `ping -w 5 192.168.10.22`

**Expected Results**

* [x] C2 can reach C1.
* [x] Inter-VLAN connectivity succeeds using DHCP-assigned addresses.

```text
cisco@C2:~$ ping -w 5 192.168.10.22
PING 192.168.10.22 (192.168.10.22) 56(84) bytes of data.
64 bytes from 192.168.10.22: icmp_seq=1 ttl=63 time=2.64 ms
64 bytes from 192.168.10.22: icmp_seq=2 ttl=63 time=2.59 ms
64 bytes from 192.168.10.22: icmp_seq=3 ttl=63 time=2.88 ms
64 bytes from 192.168.10.22: icmp_seq=4 ttl=63 time=2.53 ms
64 bytes from 192.168.10.22: icmp_seq=5 ttl=63 time=2.48 ms

--- 192.168.10.22 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4003ms
rtt min/avg/max/mdev = 2.478/2.623/2.878/0.139 ms
```
