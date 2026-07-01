# Lab Guide — <Lab Name>

## Overview

Briefly describe what this lab demonstrates.

Explain the technology or behavior being validated and why it matters in a network.

Lab Status: <Validated | In Progress | Planned | Archived>

End-to-End Verification: <Successful | Partial | Not Tested>

---

## Objectives

* [ ] Configure <technology or feature>.
* [ ] Verify <expected behavior>.
* [ ] Validate <supporting behavior, failover behavior, connectivity, or control-plane behavior>.

---

## Topology

Briefly describe the topology used in this lab.

![Topology Diagram](topology/<diagram-file-name>)

---

## Link Tables

### Physical Links

| Local Device | Local Interface | Peer Device | Peer Interface | Description |
| ------------ | --------------- | ----------- | -------------- | ----------- |
| <Device>     | <Interface>     | <Device>    | <Interface>    | <Purpose>   |

### Logical Links

| Logical Interface / Relationship                  | Devices   | Member Interfaces / Networks | Purpose   |
| ------------------------------------------------- | --------- | ---------------------------- | --------- |
| <Port-Channel / VLAN / Route / Peer Relationship> | <Devices> | <Interfaces or Networks>     | <Purpose> |

---

## Configuration Steps

> **Note:** The CLI examples below are annotated for readability. Clean device configurations are available in [`configs/`](configs/).

> **Design note:** Add any important design, protocol, or lab-specific warning here. Keep this specific to the lab.

**<Device 1>**

```bash
# <Device 1> Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname <Device 1> # Sets hostname.

<configuration commands here>

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**<Device 2>**

```bash
# <Device 2> Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname <Device 2> # Sets hostname.

<configuration commands here>

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

**<Device 3>**

```bash
# <Device 3> Configuration Block
enable # Enters privileged EXEC mode.
configure terminal # Enters global configuration mode.
hostname <Device 3> # Sets hostname.

<configuration commands here>

end # Returns to privileged EXEC mode.
write memory # Saves the running configuration.
```

---

## Verification

See [`verification/verification_commands.md`](verification/verification_commands.md) for recorded command output.

**<Device 1>**

```bash
# <Device 1> Verification Block
show <command> # Confirm <expected behavior>.
show <command> # Confirm <expected behavior>.
```

**<Device 2>**

```bash
# <Device 2> Verification Block
show <command> # Confirm <expected behavior>.
show <command> # Confirm <expected behavior>.
```

**<Device 3>**

```bash
# <Device 3> Verification Block
show <command> # Confirm <expected behavior>.
show <command> # Confirm <expected behavior>.
```

---

## Troubleshooting

> **Note:** These are quick-reference checks for this lab. They are not intended to be an exhaustive troubleshooting guide. After any change, re-run the verification steps to confirm the expected behavior.

```bash
# <Issue or symptom>.
show <command> # Confirm <specific state or condition>.
show <command> # Confirm <specific state or condition>.

# <Issue or symptom>.
show <command> # Confirm <specific state or condition>.
show <command> # Confirm <specific state or condition>.
```

---

## Artifacts

| Type           | Location                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| Configurations | [`configs/`](configs/)                                                           |
| Diagram        | [`topology/<diagram-file-name>`](topology/<diagram-file-name>)                   |
| Topology File  | [`topology/topology.yaml`](topology/topology.yaml)                               |
| Verification   | [`verification/verification_commands.md`](verification/verification_commands.md) |

---

## Document Metadata

| Field        | Value         |
| ------------ | ------------- |
| Lab Version  | 1.0           |
| Last Updated | <YYYY-MM-DD>  |
| Author       | Aaron Kindelt |
