# <Protocol / Feature> Packet Capture Analysis

This file contains packet-level analysis for the <Lab Name> lab.

The capture demonstrates <brief statement describing the overall protocol behavior>.

---

## Capture Information

**Capture Point:** <Device/interface> ↔ <Device/interface>

**Traffic Generation:** <Action used to generate or expose the traffic>

**PCAP:** `<filename.pcap>`

---

## Expected Flow

`<Stage 1> → <Stage 2> → <Stage 3> → <Stage 4>`

---

## 1. <Protocol Stage>

**Purpose:** <What this stage accomplishes in the protocol process.>

### Packet <number> — <source> → <destination> — <message type>

```text
<important decoded fields only>
```

### Interpretation

* `<field>` — <what the field proves or means>.
* `<field>` — <what the field proves or means>.
* `<field>` — <what the field proves or means>.

**Result:** <One sentence explaining what has now occurred in the protocol flow.>

---

## 2. <Protocol Stage>

**Purpose:** <What this stage accomplishes.>

### Packet <number> — <source> → <destination> — <message type>

```text
<important decoded fields>
```

### Packet <number> — <source> → <destination> — <message type>

```text
<important decoded fields>
```

### Interpretation

* `<field>` — <explanation>.
* `<field>` — <explanation>.

**Result:** <What these packets collectively prove.>

---

## 3. <Protocol Stage>

**Purpose:** <What this stage accomplishes.>

### Packet <number> — <source> → <destination> — <message type>

```text
<important decoded fields>
```

### Interpretation

#### <Message Type 1>

* `<field>` — <explanation>.
* `<field>` — <explanation>.

#### <Message Type 2>

* `<field>` — <explanation>.
* `<field>` — <explanation>.

**Result:** <What this stage proves.>

---

## CLI Correlation

The packet capture can be correlated with the device control-plane or forwarding state.

```text
<relevant show command output>
```

* `<captured value>` corresponds to `<CLI value or behavior>`.
* <Brief explanation of why the relationship matters.>

---

## What This Demonstrates

<Short paragraph summarizing the complete protocol behavior demonstrated by the capture and why the evidence matters.>