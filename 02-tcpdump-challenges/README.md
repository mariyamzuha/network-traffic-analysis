# tcpdump: Command-Line Packet Analysis

## Goal

Analyse packet captures from the Linux terminal with tcpdump instead of Wireshark: read PCAP files, filter traffic, and pull evidence out with grep and other command-line tools. Also capture and read live traffic.

Environment: Kali Linux in VirtualBox.

## Setup and basic commands

```bash
sudo apt update
sudo apt install tcpdump
tcpdump --version
sudo tcpdump -D                  # list available interfaces
sudo tcpdump -i any              # live capture on all interfaces
```

Reading a saved capture:

```bash
tcpdump -nn -r file.pcap         # read a PCAP file
tcpdump -nn -XX -r file.pcap     # show packet contents in hex and ASCII
tcpdump -nn -A -r file.pcap      # show packet contents in ASCII
tcpdump -r file.pcap | cut -d ">" -f1 | head   # source side of the first 10 packets
```

| Option | Meaning |
|---|---|
| `-r` | read from a capture file |
| `-n` / `-nn` | don't resolve hostnames / also don't resolve port names |
| `-A` | print payload as ASCII |
| `-XX` | print payload as hex and ASCII, including the Ethernet header |
| `-vvv` | most detailed output (shows checksums, TTL, etc.) |
| `-i` | choose the interface |

---

## Challenge 1: SBT-PCAP4

| # | Question | Command |
|---|---|---|
| 1 | How many UDP packets? | `tcpdump -nn -r SBT-PCAP4.pcap udp \| wc -l` |
| 2 | How many TCP packets have both SYN and ACK set? | `tcpdump -nn -r SBT-PCAP4.pcap 'tcp[tcpflags]== 18' \| wc -l` |
| 3 | Which Chrome version connected to securityblue.team? | `tcpdump -nn -A -r SBT-PCAP4.pcap > capture.txt grep -i 'security blue' capture.txt` |
| 4 | How many packets have TTL 38? | `tcpdump -nn -r SBT-PCAP4.pcap 'ip[8] == 38' \| wc -l` |

- **UDP count:** the `udp` filter keeps only UDP packets and `wc -l` counts the lines.
- **SYN+ACK:** SYN starts a TCP connection and the server replies with SYN+ACK (second step of the three-way handshake). The filter is written as `tcp[tcpflags] == 18`, because SYN (2) + ACK (16) = 18.
- **TTL:** TTL is byte 8 of the IPv4 header, so `ip[8] == 38` matches packets with TTL 38.

---

## Challenge 2: SBT-PCAP5

The capture involves a web server at 192.168.56.111.

| # | Question | Command |
|---|---|---|
| 1 | Name of the PNG file on the web server | `tcpdump -nn -A -r SBT-PCAP5.pcap 'host 192.168.56.111' \| grep -i 'png'` |
| 2 | OpenSSH version on the server | `tcpdump -nn -A -r SBT-PCAP5.pcap \| grep -i 'openssh'` |
| 3 | Port the ZIP file was served on | `tcpdump -nn -A -r SBT-PCAP5.pcap \| grep -i -B 5 -A 5 '\.zip'` |
| 4 | When was the packet with TCP checksum 53203 captured? | `tcpdump -nn -vvv -r SBT-PCAP5.pcap \| grep -i -B 5 -A 10 'cksum 0xcfd3'` |

- **ZIP port:** the lines around the `.zip` match show the IP addresses and ports. The server's port is the fixed one. The client uses a temporary high port.
- **Checksum:** tcpdump prints checksums in hex. 53203 in decimal is `0xcfd3` (`printf '%x\n' 53203`). The timestamp is at the start of the matching packet line, which is why grep shows lines before and after. The same packet can also be found with `tcp[16:2] = 53203`, since the TCP checksum sits at byte 16 of the TCP header.
---

## Live capture: browsing Mozilla

I captured live traffic in Kali while opening Mozilla websites, to connect theory to real traffic.

```bash
sudo tcpdump -i any
```

What I observed:

- Kali VM IP, VirtualBox gateway and DNS server.
- ARP traffic, used to find the gateway's MAC address.
- DNS queries for `ads.mozilla.org`, with resolution involving `mozilla.map.fastly.net`.
- Over 100,000 packets from normal browsing.

Loading one webpage is not one request. It involves DNS lookups, ARP, TCP connections to many servers, and encrypted HTTPS traffic. Even without reading the encrypted content, the IP addresses, ports and domain lookups still show where the browser is connecting.

---

## Conclusion

- tcpdump plus grep can answer most questions that I would otherwise open Wireshark for.
- BPF filters like `ip[8]` and `tcp[tcpflags]` read specific header fields directly.
