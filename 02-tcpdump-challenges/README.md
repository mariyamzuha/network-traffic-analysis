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

## Challenge 1: 
The exercise involved analysing a saved packet capture using tcpdump and command-line filters.
| Purpose | tcpdump command |
|---|---|
| Count UDP packets | `tcpdump -nn -r sample.pcap udp \| wc -l` |
| Count TCP packets with SYN and ACK flags set | `tcpdump -nn -r sample.pcap 'tcp[tcpflags] == 18' \| wc -l` |
| Extract ASCII packet contents into a text file | `tcpdump -nn -A -r sample.pcap > capture.txt` |
| Search the extracted text for a specific string | `grep -i '<search-term>' capture.txt` |
| Count packets with a specific TTL value | `tcpdump -nn -r sample.pcap 'ip[8] == 38' \| wc -l` |

---

## Challenge 2: 
This exercise involved inspecting web traffic, server banners, file requests, and TCP header information.

| Purpose | tcpdump command |
|---|---|
| Search for PNG file references | `tcpdump -nn -A -r sample.pcap 'host <server-IP>' \| grep -i 'png'` |
| Identify OpenSSH version information | `tcpdump -nn -A -r sample.pcap \| grep -i 'openssh'` |
| Search for ZIP file references and surrounding information | `tcpdump -nn -A -r sample.pcap \| grep -i -B 5 -A 5 '\.zip'` |
| Search for a specific TCP checksum value | `tcpdump -nn -vvv -r sample.pcap \| grep -i -B 5 -A 10 'cksum <hex-value>'` |
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
