\# Lab 5 – DDoS Attack Simulation on Healthcare-IoT Devices



\## Objective

Simulate a DDoS attack on Healthcare-IoT devices using the Cooja 

network simulator and analyze the traffic logs to identify and 

quantify attack traffic versus normal traffic.



\## Tools Used

\- Contiki-OS 3.0 (Ubuntu 32-bit VM)

\- Cooja Network Simulator

\- wban-udp-ddos.c (C firmware)

\- grep, wc -l (Linux log analysis)



\## Network Topology

| Node | Role | Count |

|------|------|-------|

| Node 1 | Server | 1 |

| Nodes 2–16 | Normal Sensors (60s interval) | 15 |

| Nodes 17–20 | DDoS Attackers (30s interval) | 4 |



\## Key Findings

| Metric | Result |

|--------|--------|

| Total packets sent to server | 219 |

| Packets from DDoS Node 19 | 18 |

| Packets from Normal Node 10 | 9 |

| Total from all DDoS nodes (17-20) | 73 |



\## Analysis

\- DDoS nodes sent \*\*2x more packets\*\* than normal nodes

\- 4 DDoS nodes generated \*\*73/219 = 33%\*\* of all traffic

\- This proves higher transmission frequency = attack intensity



\## Files

\- `iotddoslog.txt` — Raw simulation log

\- `Simulation-20 Motes.png` — Cooja network topology

\- `Mote Traffic Output.png` — Live mote output during simulation

\- `grep command 1-2.png` — Log analysis commands 1 and 2

\- `grep command 3-5.png` — Log analysis commands 3 to 5

