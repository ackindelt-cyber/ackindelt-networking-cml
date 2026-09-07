# List of tables used in the lab

*Note: These serve as a reference for use in creating other labs.

**Physical Link Table**
| Device  | Interface  | Connected To  | Description                            |
|---------|------------|---------------|----------------------------------------|
| S1      | G0/0       | S2 G0/0       | Physical member link between S1 and S2 |
| S1      | G0/1       | S2 G0/1       | Physical member link between S1 and S2 |
| S1      | G0/2       | S3 G0/0       | Physical member link between S1 and S3 |
| S1      | G0/3       | S3 G0/1       | Physical member link between S1 and S3 |
| S2      | G0/2       | S3 G0/2       | Physical member link between S2 and S3 |
| S2      | G0/3       | S3 G0/3       | Physical member link between S2 and S3 |

---

**Logical Link Table**
| Port-Channel | Devices    | Member Interfaces              | Static EtherChannel                   |
|--------------|------------|--------------------------------|---------------------------------------|
| Po1          | S1 <--> S2 | S1 G0/0-G0/1 <--> S2 G0/0-G0/1 | Static EtherChannel between S1 and S2 | 
| Po2          | S1 <--> S3 | S1 G0/2-G0/3 <--> S3 G0/0-G0/1 | Static EtherChannel between S1 and S3 |
| Po3          | S2 <--> S3 | S2 G0/2-G0/3 <--> S3 G0/2-G0/3 | Static EtherChannel between S2 and S3 |

---

**Device Table**
| Device  | Interface  | IP Address / Mode | Connected To  | Description       |
|---------|------------|-------------------|---------------|-------------------|
| R1      | G0/0       | Trunk             | S1 Gi0/0      | Router-on-a-Stick |
| S1      | Gi0/2      | Access            | C1 eth0       | VLAN 10 S1 to C1  |
| S1      | Gi0/3      | Access            | C2 eth0       | VLAN 20 S1 to C2  |
| S2      | Gi0/1      | Trunk             | S1 Gi0/1      | Trunk S2 to S1    |
| S2      | Gi0/2      | Access            | C3 eth0       | VLAN 10 S2 to C3  |
| S2      | Gi0/3      | Access            | C4 eth0       | VLAN 20 S2 to C4  |
| C1      | eth0       | 192.168.10.2 /24  | S1 Gi0/2      | VLAN 10 C1 to S1  |
| C2      | eth0       | 192.168.20.2 /24  | S1 Gi0/3      | VLAN 20 C2 to S1  |
| C3      | eth0       | 192.168.10.3 /24  | S2 Gi0/2      | VLAN 10 C3 to S2  |
| C4      | eth0       | 192.168.20.3 /24  | S2 Gi0/3      | VLAN 20 C4 to S2  |

---

**VLAN Table**

|VLAN | Name     | Subnet           |
|-----|----------|------------------|
| 10  | USERS_10 | 192.168.10.0 /24 |
| 20  | USERS_20 | 192.168.20.0 /24 |

---

**VLAN Definitions**
| VLAN ID | VLAN Name | Purpose                  |
|---------|-----------|--------------------------|
| 10      | USERS_10  | Non-elevated user VLAN   |
| 20      | USERS_20  | Non-elevated user VLAN   |
| 30      | USERS_30  | Non-elevated user VLAN   |
| 40      | USERS_40  | Non-elevated user VLAN   |
| 99      | Native    | Custom Native VLAN       |

---

**Interface, Role & VLAN Assignment**
| device | Interface | Mode   | VLAN(s)                     |
|--------|-----------|--------|-----------------------------|
| D1     | Gi0/1     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| D1     | Gi0/2     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| D1     | Gi0/3     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| D1     | Gi0/4     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| A1     | Gi0/0     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| A2     | Gi0/0     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| A3     | Gi0/0     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| A4     | Gi0/0     | Trunk  | Allowed: 10, 20, 30, 40, 99 |
| A1     | Gi0/1     | Access | 10                          |
| A2     | Gi0/1     | Access | 20                          |
| A3     | Gi0/1     | Access | 30                          |
| A4     | Gi0/1     | Access | 40                          |

---

**SVI Subnet, Interfaces, and Gateway**
| VLAN ID | Subnet          | SVI Interface | Gateway IP   |
|---------|-----------------|---------------|--------------|
| 10      | 192.168.10.0/24 | vlan10        | 192.168.10.1 |
| 20      | 192.168.20.0/24 | vlan20        | 192.168.20.1 |
| 30      | 192.168.30.0/24 | vlan30        | 192.168.10.1 |
| 40      | 192.168.40.0/24 | vlan40        | 192.168.20.1 |

---

**Addressing Table**
| Device  | Interface  | IP Address/Prefix | Connected To  | Description                         |
|---------|------------|-------------------|---------------|-------------------------------------|
| R1      | G0/0       | 10.0.0.1 /30      | R2 G0/0       |Point-to-point link R1 to R2         |
| R1      | G0/1       | 192.168.0.1 /24   | C1 eth0       |LAN segment                          |
| R2      | G0/0       | 10.0.0.2 /30      | R1 G0/0       |Point-to-point link R2 to R1         |
| R2      | G0/1       | 192.168.10.1 /24  | C2 eth0       |LAN segment                          |
| C1      | eth0       | 192.168.0.2 /24   | R1 G0/1       |Test client on LAN                   |
| C2      | eth0       | 192.168.10.2 /24  | R2 G0/1       |Test client on LAN                   |

---

**Router P2P Links**
| Device  | Interface  | IP Address / Prefix  | Connected To  | Description                         |
|---------|------------|----------------------|---------------|-------------------------------------|
| R1      | G0/0       | 10.0.0.1 /30         | R2 G0/0       | Point to point between R1 and R2    |
| R2      | G0/0       | 10.0.0.2 /30         | R1 G0/0       | Point to point between R2 and R1    |
| R2      | G0/1       | 10.0.0.5 /30         | R3 G0/0       | Point to point between R2 and R3    |
| R3      | G0/0       | 10.0.0.6 /30         | R2 G0/1       | Point to point between R3 and R2    |
| R3      | G0/1       | 10.0.0.9 /30         | R4 G0/0       | Point to point between R3 and R4    |
| R4      | G0/0       | 10.0.0.10 /30        | R3 G0/1       | Point to point between R4 and R3    |
| R4      | G0/1       | 10.0.0.13 /30        | R5 G0/0       | Point to point between R4 and R5    |
| R5      | G0/0       | 10.0.0.14 /30        | R4 G0/1       | Point to point between R5 and R4    |

---

**Router LAN Segments**
| Device | Interface | IP Address / Prefix | Connected To | Description  |
|--------|-----------|---------------------|--------------|--------------|
|R1      |G0/1       |192.168.10.1 /24     |N/A           |R1 LAN Segment|
|R2      |G0/2       |192.168.20.1 /24     |N/A           |R2 LAN Segment|
|R3      |G0/2       |192.168.30.1 /24     |N/A           |R3 LAN Segment|
|R4      |G0/2       |192.168.40.1 /24     |N/A           |R4 LAN Segment|
|R5      |G0/1       |192.168.50.1 /24     |N/A           |R5 LAN Segment|

---

**OSPF Config**

| Device  | Interface  | IP Address / Subnet  | Network Address    | Assigned Area | Router ID | Role       | Notes    |
|---------|------------|----------------------|--------------------|---------------|-----------|------------|----------|
| R1      | G0/0       | 10.0.0.1 /30         | 10.0.0.0 /30       | Area 1        |1.1.1.1    |Internal    |P2P to R2 |
| R2      | G0/0       | 10.0.0.2 /30         | 10.0.0.0 /30       | Area 1        |2.2.2.2    |ABR         |P2P to R1 |
| R2      | G0/1       | 10.0.0.5 /30         | 10.0.0.4 /30       | Area 0        |2.2.2.2    |ABR         |P2P to R3 |
| R3      | G0/0       | 10.0.0.6 /30         | 10.0.0.4 /30       | Area 0        |3.3.3.3    |Backbone    |P2P to R2 |
| R3      | G0/1       | 10.0.0.9 /30         | 10.0.0.8 /30       | Area 0        |3.3.3.3    |Backbone    |P2P to R4 |
| R4      | G0/0       | 10.0.0.10 /30        | 10.0.0.8 /30       | Area 0        |4.4.4.4    |ABR         |P2P to R3 | 
| R4      | G0/1       | 10.0.0.13 /30        | 10.0.0.12 /30      | Area 2        |4.4.4.4    |ABR         |P2P to R5 |
| R5      | G0/0       | 10.0.0.14 /30        | 10.0.0.12 /30      | Area 2        |5.5.5.5    |Internal    |P2P to R4 |
| R1      | G0/1       | 192.168.10.1 /24     | 192.168.10.0 /24   | Area 1        |5.5.5.5    |Internal    |R1 LAN    |
| R2      | G0/2       | 192.168.20.1 /24     | 192.168.20.0 /24   | Area 1        |5.5.5.5    |Internal    |R2 LAN    |
| R3      | G0/2       | 192.168.30.1 /24     | 192.168.30.0 /24   | Area 0        |5.5.5.5    |Internal    |R3 LAN    |
| R4      | G0/2       | 192.168.40.1 /24     | 192.168.40.0 /24   | Area 2        |5.5.5.5    |Internal    |R4 LAN    |
| R5      | G0/1       | 192.168.50.1 /24     | 192.168.50.0 /24   | Area 2        |5.5.5.5    |Internal    |R5 LAN    |

---

**Client Addressing**
| Device | Interface | IP Address / Prefix  | Connected To | Description  |
|--------|-----------|----------------------|--------------|--------------|
|C1      |G0/x       |192.168.10.10 /24     |              |R1 LAN to C1  |
|C2      |G0/x       |192.168.20.10 /24     |              |R2 LAN to C2  |
|C3      |G0/x       |192.168.30.10 /24     |              |R3 LAN to C3  |
|C4      |G0/x       |192.168.40.10 /24     |              |R4 LAN to C4  |
|C5      |G0/x       |192.168.50.10 /24     |              |R5 LAN to C5  |

---

**RoaS Subnet, Subinterface, and Gateway**
| VLAN ID | Subnet          | Subinterface | Gateway IP   |
|---------|-----------------|--------------|--------------|
| 10      | 192.168.10.0/24 | Gi0/0.10     | 192.168.10.1 |
| 20      | 192.168.20.0/24 | Gi0/0.20     | 192.168.20.1 |

---


**Device Table**
| Device  | Interface  | IP Address / Prefix | Connected To  | Description         |
|---------|------------|---------------------|---------------|---------------------|
| R1      | G0/0       | None                | S1 G0/0       | RoaS trunk to S1    |
| S1      | G0/0       | Trunk               | R1 G0/0       | S1 trunk to RoaS R1 |
| S1      | G0/1       | 192.168.10.1 /24    | C1 eth0       | LAN segment         |
| S1      | G0/2       | 192.168.10.1 /24    | C2 eth0       | LAN segment         |
| C1      | eth0       | DHCP                | S1 G0/1       | Test client VLAN 10 |
| C2      | eth0       | DHCP                | S2 G0/2       | Test client VLAN 20 |

---

**DHCP Addressing**
| Subnet / VLAN  | VLAN | Gateway      | DHCP Pool Range                   | Excluded Addresses              |
|----------------|------|--------------|-----------------------------------|---------------------------------|
|192.168.10.0/24 | 10   | 192.168.10.1 | 192.168.10.21/24 - 192.168.10.254 | 192.168.10.1/24 - 192.168.10.20 |
|192.168.20.0/24 | 20   | 192.168.20.1 | 192.168.20.21/24 - 192.168.20.254 | 192.168.20.1/24 - 192.168.20.20 |

---