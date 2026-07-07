\# Lab 5 — Network Packet Analysis \& SIEM Deployment



\## Overview

Hands-on network forensics lab covering packet capture, 

traffic analysis, and SIEM deployment using Wireshark, 

TShark, and Wazuh 4.7.



\## Tools Used

\- Wireshark — GUI packet analyser

\- TShark — Command-line packet analyser  

\- Wazuh 4.7 — Open-source SIEM/XDR

\- VirtualBox — Ubuntu Agent, Windows 10, Wazuh Server VMs



\## Environment

\- Wazuh Server : Ubuntu Server 24.04 (192.168.1.8)

\- Linux Agent  : Ubuntu 22.04 (192.168.1.5)

\- Windows Agent: Windows 10 (192.168.1.9)

\- Network      : ULNAT NAT Network (192.168.1.0/24)



\## Task Summary



| Task | Description | Key Finding |

|------|-------------|-------------|

| 1 | DNS query count | 878 DNS packets |

| 2 | Filter HTTPS from machine | tcp.port == 443 \&\& ip.src == 192.168.1.5 |

| 3 | TCP handshake identification | SYN → SYN-ACK → ACK on stream 0 |

| 4 | File extraction from packets | apple-touch-icon.png from 172.67.132.115 |

| 5 | Unique destination IPs (TShark) | 26+ unique IPs identified |

| 6 | Most contacted IP | 192.168.1.5 (29,417 hits) |

| 7 | Highest traffic protocol | TCP — 96.5% of all traffic |

| 8 | TCP 443 without DNS | tcp.port == 443 \&\& !dns |

| 9 | TCP stream analysis | GET / HTTP/1.1 → 200 OK via Cloudflare |



\## Key TShark Commands



\### List unique destination IPs

```bash

tshark -r capture.pcapng -T fields -e ip.dst | sort -u

```



\### Find most contacted IP

```bash

tshark -r capture.pcapng -T fields -e ip.dst | sort | uniq -c | sort -rn | head -10

```



\## Wazuh SIEM Deployment



\### Installation

```bash

curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh \&\& \\

chmod +x wazuh-install.sh \&\& \\

sudo ./wazuh-install.sh -a --ignore-check

```



\### Agents Connected

| Agent | OS | Status |

|-------|----|--------|

| jeji-VirtualBox | Ubuntu 22.04.5 LTS | Active |

| Windows-Agent-Endpoint | Windows 10 Home | Active |



\## Blog Post

Read the full writeup on CyberTrail:

\[Network Packet Analysis \& SIEM Deployment](https://hashnode.com/@jeji-james)



\## Screenshots

All evidence screenshots are in the `/screenshots` folder.

