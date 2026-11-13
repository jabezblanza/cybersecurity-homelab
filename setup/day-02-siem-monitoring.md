# Day 02 — SIEM Configuration & SSH Authentication Monitoring

**Project:** Cybersecurity Home Lab — Offensive Security & Incident Response  
**Date:** November 13, 2026  
**Status:** Completed  
**Environment:** Oracle VirtualBox | Ubuntu Server | Kali Linux | Wazuh SIEM

---

## 1. Objective

Deploy and configure the Wazuh security monitoring agent on Ubuntu Server, establish centralized log collection, and verify threat detection through a controlled SSH authentication failure.

The exercise demonstrates the process of generating a security event, collecting endpoint logs, analyzing alerts, and validating detection results using a Security Information and Event Management (SIEM) platform.

## 2. Lab Environment

| Component | Role | IP Address |
|---|---|---|
| Kali Linux | Attacker / Test Machine | 192.168.56.10 |
| Ubuntu Server 24.04.5 LTS | Monitored Endpoint | 192.168.56.20 |
| Wazuh SIEM | Security Monitoring | 192.168.56.30 |
| Windows Host | Dashboard Access | 192.168.56.8 |

**Monitoring tools:** Wazuh Manager, Wazuh Agent 4.14.10, Wazuh Dashboard, Linux systemd journal.

**Network:** VirtualBox Host-only (192.168.56.0/24), with NAT available for software installation and updates.

## 3. Wazuh Agent Deployment

### 3.1 Agent Installation

Downloaded the Wazuh Agent 4.14.10 package from the official Wazuh repository.

```bash
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.14.10-1_amd64.deb
```

Installed the agent and specified its manager and endpoint identity.

```bash
sudo env WAZUH_MANAGER='192.168.56.30' WAZUH_AGENT_NAME='ubuntu-victim' dpkg -i ./wazuh-agent_4.14.10-1_amd64.deb
```

### 3.2 Agent Service Configuration

```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
sudo systemctl status wazuh-agent --no-pager
```

**Result:** The Wazuh agent started successfully and was enabled for automatic startup.

### 3.3 Agent Registration

Verified endpoint registration in the Wazuh dashboard.

| Field | Value |
|---|---|
| Agent ID | 001 |
| Agent Name | ubuntu-victim |
| Operating System | Ubuntu 24.04.5 LTS |
| Agent IP | 192.168.56.20 |
| Agent Version | 4.14.10 |
| Agent Status | Active |

**Result:** The Ubuntu endpoint successfully connected to the Wazuh Manager.

## 4. Log Collection Verification

Reviewed the Ubuntu SSH service logs.

```bash
sudo journalctl -u ssh --since "today" --no-pager
```

Verified the presence of the authentication log file.

```bash
ls -l /var/log/auth.log
```

Inspected the Wazuh agent's log collection configuration.

```bash
sudo grep -A 5 '<localfile>' /var/ossec/etc/ossec.conf
```

Identified the following systemd journal collection configuration:

```xml
<localfile>
  <log_format>journald</log_format>
  <location>journald</location>
</localfile>
```

**Result:** The agent was configured to monitor systemd journal events. Wazuh subsequently confirmed ingestion of the SSH authentication failure through the `journald` source.

## 5. Controlled SSH Authentication Test

### 5.1 Scenario

A simulated unauthorized SSH authentication attempt was performed from Kali Linux against Ubuntu Server.

**Purpose:** Verify that a failed authentication attempt is recorded, forwarded, and classified by Wazuh SIEM.

### 5.2 Test Execution

Executed the following command from Kali Linux:

```bash
ssh jabb@192.168.56.20
```

Entered an intentionally incorrect password to generate a failed authentication event.

**Observed result:** Ubuntu rejected the authentication attempt.

### 5.3 Local Log Investigation

Queried Ubuntu's authentication logs:

```bash
sudo grep 'Failed password' /var/log/auth.log | tail -n 10
```

The following event was identified:

```text
2026-11-13T01:46:39.136056+00:00 jab-server sshd[4842]: Failed password for jabb from 192.168.56.10 port 42104 ssh2
```

**Finding:** Ubuntu recorded an unsuccessful SSH authentication attempt originating from the Kali Linux host.

## 6. SIEM Alert Investigation

Opened the Wazuh Threat Hunting interface and filtered events using:

```text
rule.groups: authentication_failed
```

The dashboard displayed two authentication-related alerts:

| Rule ID | Description | Level |
|---|---|---|
| 5760 | sshd: authentication failed. | 5 |
| 5503 | PAM: User login failed. | 5 |

These were observed in the same test period and may represent different authentication records associated with the test.

### 6.1 Verified SSH Alert Details

| Field | Observed Value |
|---|---|
| Rule ID | 5760 |
| Alert Level | 5 |
| Source IP | 192.168.56.10 |
| Source Port | 42104 |
| Destination Agent | ubuntu-victim |
| Destination Agent IP | 192.168.56.20 |
| Target Username | jabb |
| Decoder | sshd |
| Log Source | journald |
| Alert Timestamp | November 13, 2026 — 09:46:39 PHT |

The Wazuh `full_log` field contained the failed SSH authentication message matching the Ubuntu log.

### 6.2 MITRE ATT&CK Mapping

The Wazuh detection rule was associated with:

- **T1110.001 — Password Guessing**
- **T1021.004 — SSH**

These mappings describe relevant attack techniques associated with the rule. They do not establish that a brute-force attack, lateral movement, or successful compromise occurred.

### 6.3 Investigation Conclusion

The controlled SSH login failed, and Wazuh generated a corresponding authentication failure alert.

The source IP, target username, port, timestamp, and original log message were successfully correlated between Ubuntu and Wazuh.

**Classification:** Authorized security monitoring validation exercise.

**Observed outcome:** Failed authentication. No successful login was demonstrated by this test.

## 7. Verification Results

| Validation | Result |
|---|---|
| Ubuntu-to-Wazuh connectivity | Passed |
| Wazuh Agent installation | Passed |
| Agent registration | Passed |
| Agent status | Active |
| SSH authentication logging | Verified |
| Systemd journal collection | Verified |
| Authentication failure detection | Verified |
| Event correlation | Verified |

## 8. Lessons Learned

- Learned how to deploy and register a Wazuh endpoint agent.
- Identified the difference between local operating system logs and SIEM alerts.
- Practiced inspecting SSH authentication events using Linux commands.
- Verified that Wazuh collected events through the systemd journal.
- Analyzed Wazuh rule IDs, severity levels, source addresses, and authentication details.
- Learned that multiple related alerts do not necessarily indicate multiple independent attack attempts.
- Distinguished failed authentication from confirmed unauthorized access.

## 9. Outcome

Successfully implemented endpoint monitoring and validated the detection of a controlled SSH authentication failure.

The exercise demonstrated an operational logging and analysis workflow between Kali Linux, Ubuntu Server, and Wazuh SIEM.

## 10. Next Steps

- Perform controlled network reconnaissance with Nmap.
- Identify exposed services and investigate the resulting telemetry.
- Study additional detection methods for network reconnaissance.
- Continue documenting security observations and investigation findings.
