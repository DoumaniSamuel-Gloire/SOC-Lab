# Suricata — Network IDS

## Role
Suricata monitors raw network traffic on `vm-cible`, complementing Wazuh. It detects behavior that leaves no trace in system logs — port scans, known attack signatures, traffic from poorly-reputed IPs.

## Installation
```bash
sudo apt update
sudo apt install suricata -y
```

## Configuration
- Monitored interface: `enp0s0`
- Config file: `/etc/suricata/suricata.yaml`
- `default-rule-path` pointed to `/var/lib/suricata/rules` (the actual location where `suricata-update` writes the rules)

## Ruleset
Emerging Threats Open ruleset, downloaded and managed via `suricata-update`:

```bash
sudo suricata-update
```

68,155 rules loaded (52,216 enabled) on initial deployment.

## Custom Rule — Port Scan Detection
The generic ruleset does not cover behavioral patterns like port scanning by default. A dedicated threshold-based rule was written in `rules/custom.rules`:

```
alert tcp any any -> $HOME_NET any (msg:"Possible port scan detected"; flags:S; threshold:type both, track by_src, count 15, seconds 3; sid:1000001; rev:1;)
```

**Logic**: triggers an alert when a single source IP initiates 15+ TCP SYN connections within a 3-second window — the typical signature of an automated scan (nmap).

## Logs
- `/var/log/suricata/fast.log` — condensed text format, one line per alert
- `/var/log/suricata/eve.json` — detailed JSON format, all event types (alert, flow, dns, tls...)

## Wazuh Integration
The Wazuh agent continuously reads `eve.json` and forwards events to the manager, where they appear under `rule.id: 86601` with original Suricata fields preserved (`data.alert.signature`, `data.alert.signature_id`, etc.). See [Incident 002](../../incidents/002-port-scan/README.md) for the full demonstration.

## Testing / Validation
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -i enp0s0
```
