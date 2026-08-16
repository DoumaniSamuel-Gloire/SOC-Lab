# Incident 002 — Port Scan Detection (Suricata)

## 1. Objective
Deploy a network-based IDS (Suricata) to detect activity that leaves no trace in system logs, and evaluate its integration with Wazuh for centralized alerting.

## 2. Lab Environment
| Component | Details |
|---|---|
| Attacker | 10.10.0.4 |
| Target | 10.10.0.3 |
| Manager | 10.10.0.2 |
| Target OS | Ubuntu |
| IDS | Suricata 6.0.4 |
| SIEM | Wazuh (integrated) |
| Scan Tool | Nmap |

## 3. IDS Deployment
Suricata was installed and configured on the target VM:
- Monitored interface: `enp0s0`
- Ruleset: Emerging Threats Open (68,155 rules) via `suricata-update`

### Custom Detection Rule
Generic threat-signature rulesets do not cover behavioral patterns like port scanning by default. A dedicated threshold-based rule was written:
