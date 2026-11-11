# Capstone: Investigating ARP Spoofing and Man-in-the-Middle Attacks

## Goal

Practise investigating suspicious network traffic in Wireshark, identify signs of ARP spoofing, examine FTP file transfers, and understand how an attacker can intercept network communications.

## Overview

This investigation focused on a simulated network security incident involving ARP spoofing and a potential Man-in-the-Middle (MITM) attack.

The analysis involved examining ARP requests and replies, comparing Ethernet and IP addresses, inspecting FTP traffic, and following TCP streams to understand how data was transferred.

## 1. Identifying Suspicious ARP Activity

I started by making the source address visible in the Packet List pane:

- Right-click a field in Packet Details.
- Select **Apply as Column**.
- Use the new column to compare packet source addresses.

I then used the following Wireshark display filters:

| Purpose                                           | Wireshark display filter         |
| ------------------------------------------------- | -------------------------------- |
| Display ARP traffic                               | `arp`                            |
| Display ARP requests                              | `arp.opcode == 1`                |
| Display ARP replies                               | `arp.opcode == 2`                |
| Identify duplicate IP address warnings            | `arp.duplicate-address-detected` |
| Filter traffic involving a particular MAC address | `eth.addr == <MAC-address>`      |

I expanded the Ethernet II and Address Resolution Protocol sections in Packet Details to inspect hardware and protocol addresses.

One important warning sign is the same MAC address being associated with multiple IP addresses in suspicious ARP activity. This can indicate ARP spoofing, particularly when a device is falsely claiming another device's IP address.

**Additional Wireshark feature:** Analyze > Expert Information can help identify warnings such as duplicate IP address detection.

## 2. Understanding the Attack

ARP (Address Resolution Protocol) maps IP addresses to MAC addresses within a local network.

ARP does not provide authentication, so an attacker on the same local network may send forged ARP replies to associate their MAC address with another device's IP address.

This can cause network traffic to pass through the attacker's device.

## 3. Examining FTP File Transfers

I used the following display filter to isolate FTP traffic:

```text
ftp
```

FTP control traffic can reveal commands exchanged between a client and server.

The command is:

```text
RETR <filename>
```

`RETR` is an FTP command used to retrieve a file from the server.

FTP generally uses separate connections for control information and file data:

- **Control connection:** Commonly uses TCP port 21.
- **Data connection:** Carries the transferred file contents.

To examine a particular TCP connection, I used:

```text
tcp.port == <port>
```

I then right-clicked a relevant packet and selected:

**Follow > TCP Stream**

This reconstructs the communication exchanged over the TCP connection and makes it easier to inspect data that is divided across multiple packets.

## 4. Searching Packet Contents

Wireshark allows packet contents to be searched for specific text.

For example, I used:

```text
frame contains "<search-term>"
```

This is useful when investigating traffic containing readable application data.

A packet may contain only part of a transferred file, so examining the complete TCP stream can provide more context than inspecting an individual packet.

## 5. Comparing SSH and FTP Security

I examined SSH traffic using:

```text
tcp.port == 22
```

SSH encrypts its communication, making its application data unreadable through ordinary packet inspection.

I also examined FTP traffic and learned that traditional FTP does not encrypt its contents by default.

This creates a security risk: sensitive information transmitted through an unencrypted protocol may be exposed even when another service, such as SSH, uses encryption.

The important lesson is that using an encrypted service does not protect credentials that are also transmitted through an insecure service.

## Conclusion

- ARP does not authenticate devices, making ARP spoofing possible on local networks.
- Comparing IP and MAC addresses can help identify suspicious ARP activity.
- Wireshark's Expert Information feature can highlight potential network issues.
- FTP uses separate control and data connections.
- Follow TCP Stream helps reconstruct communication distributed across multiple packets.
- SSH encrypts application traffic, whereas traditional FTP does not provide encryption by default.
- Sensitive credentials should never be stored or transmitted in plain text.

## Possible Defenses

- Avoid storing passwords in plain-text files.
- Configure switch port security.
- Disable unused switch ports.
- Monitor networks for suspicious ARP activity and duplicate IP address warnings.
