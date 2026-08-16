# Wazuh — SIEM

## Role
Wazuh is the central SIEM of the lab. It collects, correlates, and centralizes security events from multiple sources: system logs (auth.log, PAM), application logs, and — after integration — network alerts from Suricata.

## Architecture
| Component | VM | Role |
|---|---|---|
| Wazuh Manager | vm-manager (10.10.0.2) | Reception, correlation, dashboard |
| Wazuh Agent | vm-cible (10.10.0.3) | Local log collection, transmission to manager |

Agent → manager communication over ports **1514** (event transmission) and **1515** (agent enrollment).

## Installation
Installed via the official Wazuh APT repository (recommended method, ensures version consistency between agent and manager):

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" \
  | sudo tee -a /etc/apt/sources.list.d/wazuh.list
sudo apt-get update
sudo WAZUH_MANAGER="10.10.0.2" apt-get install wazuh-agent
```

## Log Collection
Monitored sources configured in `/var/ossec/etc/ossec.conf` (`<localfile>`), including:
- `/var/log/auth.log` (SSH/PAM authentication)
- `/var/log/suricata/eve.json` (network alerts, see Suricata integration)
- Periodic system commands (`netstat`, `last`, `df`)

## Rules and Detection
Key rules used across the lab's incidents:

| Rule ID | Description | Level |
|---|---|---|
| 5760 | sshd: authentication failed | 5 |
| 5710 | Attempt to login using a non-existent user | 5 |
| 40112 | Multiple authentication failures followed by a success | 12 |
| 86601 | Generic Suricata alerts (via eve.json) | 3 |

## Dashboard & MITRE ATT&CK
Every alert is automatically enriched with:
- Compliance classification (GDPR, HIPAA, PCI-DSS, NIST 800-53)
- MITRE ATT&CK mapping (technique + tactic)

Accessible via **Threat Hunting → Events**, with DQL filtering (`rule.level`, `data.srcip`, `agent.name`, etc.).

## Secure Access
The dashboard (port 443) is never exposed directly to the internet. Access is exclusively through a Cloud IAP tunnel:

```bash
gcloud compute start-iap-tunnel vm-manager 443 --local-host-port=localhost:8443 --zone=us-central1-a
```
