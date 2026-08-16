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
alert tcp any any -> $HOME_NET any (msg:"Possible port scan detected"; flags:S; threshold:type both, track by_src, count 15, seconds 3; sid:1000001; rev:1;)


This rule triggers when a single source initiates 15+ TCP SYN connections within a 3-second window.

## 4. Attack Simulation
A port scan was launched from the attacker VM:

```bash
nmap -sS -T4 -p 1-1000 10.10.0.3
```

## 5. Detection

[1:1000001:1] Possible port scan detected [Priority: 3] {TCP} 10.10.0.4 -> 10.10.0.3


The custom rule (sid:1000001) fired correctly, confirming detection of the rapid multi-port connection pattern.

## 6. Wazuh Integration
The Wazuh agent was configured to ingest Suricata's `eve.json` log:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/var/log/suricata/eve.json</location>
</localfile>
```

Result: Suricata alerts now appear in the Wazuh dashboard under `rule.id: 86601`, preserving original fields (`data.alert.signature`, `data.alert.signature_id`, `data.src_ip`, `data.dest_ip`), enabling centralized SIEM + IDS visibility.

## 7. Unexpected Finding — Real Malicious Traffic
While reviewing Suricata logs, genuine malicious traffic was identified: multiple IPs listed on threat-reputation feeds (**Spamhaus DROP**, **CINS Active Threat Intelligence**, **Dshield**) were probing port 443 on the target VM. Root cause: a firewall rule temporarily left open to `0.0.0.0/0` during dashboard access troubleshooting.

### Corrective Action
- Firewall rule restricted to the Cloud IAP range only (`35.235.240.0/20`)
- Dashboard access secured via IAP tunnel (`gcloud compute start-iap-tunnel`), removing direct port 443 exposure to the internet

## 8. MITRE ATT&CK
| Technique | Tactic |
|---|---|
| T1046 — Network Service Discovery | Discovery |

## 9. Lessons Learned
- Signature-based rulesets do not detect behavioral/statistical patterns (e.g. port scans) — dedicated threshold rules are required
- Network-level detection (IDS) is complementary to system-log detection (SIEM), not redundant — port scans leave no trace in system logs
- A temporarily permissive firewall rule gets exploited within minutes by automated internet-wide scanning bots
- SIEM + IDS integration provides a single pane of glass, essential for real-world SOC workflows

## 10. Recommendations
- Keep firewall rules scoped to the minimum necessary range at all times, even temporarily
- Extend custom Suricata rules to cover other behavioral patterns (e.g. DNS tunneling, slow scans)
- Consider periodic firewall rule audits to catch configuration drift
