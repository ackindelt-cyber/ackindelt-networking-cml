# PCAP Analysis
 
## Capture Overview
 
| Field              | Value                                                |
|--------------------|------------------------------------------------------|
| Capture File       | `<capture-name>.pcap`                                |
| Capture Point      | `<device/interface/link>`                            |
| Protocol / Traffic | `<OSPF / BGP / ARP / ICMP / DHCP / etc.>`            |
| Purpose            | `<What behavior this capture is intended to verify>` |

---

## Traffic Flow
 
> Remove this section when packet sequencing is not important to the analysis.
 
```text
<DEVICE-A>                         <DEVICE-B>
    |                                  |
    | ---- <Message / Packet> -------> |
    |                                  |
    | <--- <Message / Packet> -------- |
    |                                  |
    | ---- <Message / Packet> -------> |
    |                                  |
```
 
`<Briefly explain the exchange and what the sequence demonstrates.>`

---
 
## Wireshark Filter
 
```text
<display filter>
```
 
 ---

## Packet Analysis
 
### Packet `<number>` — `<Packet / Message Type>`
 
**Source:** `<IP / MAC>`  
**Destination:** `<IP / MAC>`  
**Protocol:** `<protocol>`
 
| Field     | Observed Value | Significance         |
|-----------|----------------|----------------------|
| `<field>` | `<value>`      | `<Why this matters>` |
| `<field>` | `<value>`      | `<Why this matters>` |
| `<field>` | `<value>`      | `<Why this matters>` |
 
**Interpretation**
 
`<Explain what this packet demonstrates. Focus on why the packet exists, what the important fields mean, and how they relate to the expected protocol behavior.>`
 
### Packet `<number>` — `<Packet / Message Type>`
 
**Source:** `<IP / MAC>`  
**Destination:** `<IP / MAC>`  
**Protocol:** `<protocol>`
 
| Field     | Observed Value | Significance         |
|-----------|----------------|----------------------|
| `<field>` | `<value>`      | `<Why this matters>` |
| `<field>` | `<value>`      | `<Why this matters>` |
 
**Interpretation**
 
`<Explain how this packet relates to the previous packet and what stage of the exchange or protocol process it represents.>`
 
 ---
 
## Conclusion
 
The capture confirms that `<specific protocol behavior or traffic flow>` occurred as expected.
 
Key evidence:
 
- `<Important verified behavior>`
- `<Important field, state, flag, address, or protocol value>`
- `<Important response, transition, or forwarding result>`
 
The packet-level evidence is consistent with the CLI verification recorded in the lab.
 