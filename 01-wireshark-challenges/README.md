# Wireshark Basics and Packet Capture Challenges

## Goal

Learn to read packet captures in Wireshark and use filters and statistics to answer questions about the traffic in two PCAP challenges. This write-up focuses on the methods and commands I used.

## Wireshark skills covered

- Main window: menu bar, display filter bar, and the three panes: Packet List, Packet Details, Packet Bytes (hex and ASCII)
- Capture filters vs display filters (`tcp port 80` vs `tcp.port == 80`)
- Applying display filters to isolate traffic
- Following TCP streams to rebuild a conversation
- Right-click a field, then Apply as Column
- Statistics menu: Protocol Hierarchy, Conversations, Endpoints

---

## PCAP 1

| # | Question | Method |
|---|---|---|
| 1 | Which protocol is used over port 3942? | Filter `tcp.port == 3942 \|\| udp.port == 3942`, then read the Protocol column |
| 2 | Which host was pinged twice? | Filter `icmp`, look for echo requests sent to the same destination twice |
| 3 | How many DNS query response packets were captured? | Filter `dns.flags.response == 1`, read the displayed packet count in the status bar |
| 4 | Which host sent the most bytes? | Statistics > Endpoints > IPv4 tab, sort by Tx Bytes |

---

## PCAP 2

This capture involved an attack on a Windows host, so most questions were about application-level evidence.

| Question | Method |
|---|---|
| Find the WebAdmin password | Filter `http`, look at POST requests to the login page, then Follow > HTTP Stream |
| Find the attacker's FTP server version | Filter `ftp`, read the `220 ... ready` server banner |
| Find the confidential file name | Filter `ftp` and follow Stream |
| Find the log file created at 4:51 AM | Edit > Find Packet, search by .log string, then check the packet timestamps |

---

## Conclusions

- Sorting Endpoints by bytes is a fast way to find the busiest host.
- HTTP and FTP send data in plain text, so credentials and server versions can be read straight from the capture.
- Following a stream is much clearer than reading packets one by one.
- Searching packet contents by string saves a lot of time on large captures.
