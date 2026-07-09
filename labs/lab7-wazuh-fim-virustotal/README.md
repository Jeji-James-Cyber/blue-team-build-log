# Lab : Wazuh File Integrity Monitoring & VirusTotal Malware Detection

*Blue Team Build Log *

---

## What this lab covers

In this lab, I configured Wazuh's File Integrity Monitoring (FIM) to detect file changes across both Ubuntu and Windows agents, then integrated VirusTotal's API to automatically scan new files for malware. This is a core workflow in real-world SOC operations — detecting unauthorized file changes and triaging potential threats without manual intervention.

---

## Lab environment

| Machine | Role | IP |
|---|---|---|
| Machine A | Wazuh Server (Ubuntu 24.04) | 192.168.1.8 |
| Machine B | Ubuntu Agent | 192.168.1.5 |
| Machine C | Windows 10 Agent | 192.168.1.9 |

All VMs run on VirtualBox in a NAT network (192.168.1.0/24).

---

## Part 1: File Integrity Monitoring

### Why FIM matters

FIM is how defenders detect ransomware dropping payloads, attackers planting backdoors, or insiders tampering with configs. Wazuh's `syscheck` module monitors directories in real time and alerts on any file creation, modification, or deletion.

### Step 1 — Configure agent.conf on the Wazuh Server

Instead of editing each agent individually, Wazuh lets you push config to all agents from one central file:

```
/var/ossec/etc/shared/default/agent.conf
```

Added the following XML block to monitor target directories on both agents:

```xml
<agent_config>
  <syscheck>
    <directories realtime="yes" report_changes="yes">/etc/tmp</directories>
    <directories realtime="yes" report_changes="yes">C:\Users\Public\tmp</directories>
  </syscheck>
</agent_config>
```

`realtime="yes"` enables instant detection rather than waiting for scheduled scans. `report_changes="yes"` logs the actual content that changed.

### Step 2 — Restart services

```bash
# Machine A — Wazuh Server
sudo systemctl restart wazuh-manager

# Machine B — Ubuntu Agent
sudo systemctl restart wazuh-agent

# Machine C — Windows Agent (PowerShell Admin)
Restart-Service -Name "WazuhSvc"
```

### Step 3 — Trigger FIM events

Created and modified test files in both monitored directories:

```bash
# Ubuntu Agent
sudo touch /etc/tmp/testfile.txt
echo "Hello from Ubuntu FIM test" | sudo tee /etc/tmp/testfile.txt
echo "Modified content" | sudo tee -a /etc/tmp/testfile.txt
```

```powershell
# Windows Agent
"Hello from Windows FIM test" | Out-File "C:\Users\Public\tmp\testfile.txt"
"Modified content" | Out-File -Append "C:\Users\Public\tmp\testfile.txt"
```

### Step 4 — Verify on Wazuh Dashboard

Navigated to each agent's **Integrity Monitoring → Inventory** tab and confirmed both monitored files appeared with their metadata (path, user, last modified, hash values).

---

## Part 2: VirusTotal Malware Detection

### How it works

When Wazuh's FIM detects a new file, it computes the file's MD5/SHA1 hash and sends it to VirusTotal's API. VirusTotal checks the hash against 65+ antivirus engines and returns a verdict. If the file is flagged, Wazuh generates a high-severity alert.

### Step 5 — Add VirusTotal integration

Added this block to `/var/ossec/etc/ossec.conf` before the closing `</ossec_config>` tag:

```xml
<integration>
  <name>virustotal</name>
  <api_key>YOUR_API_KEY_HERE</api_key>
  <rule_id>554, 553, 550</rule_id>
  <alert_format>json</alert_format>
</integration>
```

The `rule_id` values (550, 553, 554) are Wazuh's built-in FIM alert rules — this tells the integration to send VirusTotal lookups whenever those rules fire.

### Step 6 — Drop the EICAR test file

EICAR (European Institute for Computer Antivirus Research) provides a harmless test file that every antivirus engine worldwide is programmed to flag. It's the standard way to test detection without real malware.

```bash
# Ubuntu Agent
sudo wget -P /etc/tmp/ https://secure.eicar.org/eicar.com.txt
```

```powershell
# Windows Agent
Invoke-WebRequest -Uri "https://secure.eicar.org/eicar.com.txt" -OutFile "C:\Users\Public\tmp\eicar.com.txt"
```

Windows Defender immediately flagged and quarantined the file on Machine C — expected behavior.

### Step 7 — Results on Wazuh Dashboard

Navigating to **Security Events** and searching for `virustotal` revealed the following alerts:

| Rule ID | Description | Severity |
|---|---|---|
| 87103 | No records in VirusTotal database (normal file) | 3 |
| 87104 | No positives found (Windows EICAR — deleted by Defender before hash) | 3 |
| 87105 | 63 engines detected `/etc/tmp/eicar.com.txt` | **12** |

The rule 87105 alert contained full forensic detail:

- `data.virustotal.positives: 63`
- `data.virustotal.total: 65`
- `data.virustotal.malicious: 1`
- `rule.mitre.id: T1203`
- `rule.mitre.tactic: Execution`
- A direct VirusTotal permalink for independent verification

---

## Key takeaways

- Wazuh's shared `agent.conf` lets you manage FIM across all agents from a single server-side file — no touching individual agent configs
- FIM Inventory gives a real-time database view of all monitored files with cryptographic hashes
- VirusTotal integration turns FIM into an automated malware triage pipeline — the SOC analyst gets a severity-12 alert with a permalink rather than having to manually investigate every new file
- The EICAR test file is flagged by 63/65 engines, confirming the detection pipeline is working end-to-end

---

## Tools used

- Wazuh 4.7.5 (SIEM/XDR)
- VirtualBox 7.x
- Ubuntu Server 24.04 (Wazuh Server)
- Ubuntu 22.04 Desktop (Agent)
- Windows 10 Home (Agent)
- VirusTotal Free API

---

*Part of the Blue Team Build Log series — documenting my journey from MEng student to SOC analyst.*

*GitHub: [blue-team-build-log](https://github.com/Jeji-James-Cyber/blue-team-build-log)*
