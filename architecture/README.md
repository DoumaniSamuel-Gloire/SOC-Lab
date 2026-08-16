# Lab Architecture

## Overview

The lab simulates a small-scale SOC environment: an attacker, a monitored target, and a manager centralizing detection.

```
Attacker (10.10.0.4)
        │
        │ SSH brute-force / port scan
        ▼
Target (10.10.0.3)
   ├── Wazuh Agent
   ├── Suricata (IDS)
   └── SSH (sshd)
        │
        │ logs (1514) / eve.json
        ▼
Wazuh Manager (10.10.0.2)
   ├── Rule correlation
   ├── MITRE ATT&CK mapping
   └── Dashboard (Threat Hunting)
```

## Virtual Machines

| VM | Role | Internal IP | OS |
|---|---|---|---|
| vm-attaquant | Attack simulation (Hydra, Nmap) | 10.10.0.4 | Ubuntu |
| vm-cible | Monitored endpoint (Wazuh Agent + Suricata) | 10.10.0.3 | Ubuntu |
| vm-manager | Wazuh Manager + Dashboard | 10.10.0.2 | Ubuntu |

## Communication Flows

| Source | Destination | Port | Purpose |
|---|---|---|---|
| vm-attaquant | vm-cible | 22 | SSH brute force (Hydra) |
| vm-attaquant | vm-cible | 1-1000 | Port scan (Nmap) |
| vm-cible | vm-manager | 1514 | Wazuh event transmission |
| vm-cible | vm-manager | 1515 | Agent enrollment |
| Analyst (me) | vm-manager | 443 (via IAP tunnel) | Wazuh dashboard access |

## Security Principle Applied

No direct access from the internet is allowed to any lab VM. All external connections (SSH, dashboard) go exclusively through **Cloud IAP** (range `35.235.240.0/20`), ensuring no sensitive port is publicly exposed — see [infrastructure/gcp](../infrastructure/gcp/README.md) for firewall rule details.

