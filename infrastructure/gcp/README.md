# GCP Infrastructure

## VPC

- Name: `lab-vpc`
- Mode: custom (dedicated subnet, no auto-generated subnets)
- Subnet: `10.10.0.0/24`, region `us-central1`

## Firewall Rules

| Rule | Direction | Source | Port | Purpose |
|---|---|---|---|---|
| `lab-allow-internal` | Ingress | `10.10.0.0/24` | tcp, udp, icmp | Unrestricted communication between lab VMs |
| `lab-allow-iap-ssh` | Ingress | `35.235.240.0/20` (Cloud IAP) | tcp:22 | SSH access exclusively via IAP |
| `lab-allow-dashboard` | Ingress | `35.235.240.0/20` (Cloud IAP) | tcp:443 | Wazuh dashboard access exclusively via IAP |

**Principle applied**: no rule allows `0.0.0.0/0`. Any broad exposure was identified as a risk (see [Incident 002](../../incidents/002-port-scan/README.md), "Unexpected Finding" section) and corrected.

## Cloud IAP

Cloud Identity-Aware Proxy is used as the sole external entry point to lab VMs, instead of directly exposing SSH/HTTPS ports to the internet.

**Why**:
- IAP's IP range (`35.235.240.0/20`) is fixed and documented by Google — unlike a dynamic personal IP, it never needs updating
- Eliminates direct exposure of sensitive services to constant automated internet-wide scanning

**Usage** — tunnel to the Wazuh dashboard:
```bash
gcloud compute start-iap-tunnel vm-manager 443 --local-host-port=localhost:8443 --zone=us-central1-a
```
Then access via `https://localhost:8443`.

## Virtual Machines

| VM | Zone | Machine Type | Role |
|---|---|---|---|
| vm-attaquant | us-central1-a | c4a (ARM64) | Attack simulation |
| vm-cible | us-central1-c | c4a (ARM64) | Monitored endpoint |
| vm-manager | us-central1-a | c4a (ARM64) | Wazuh Manager |

**Note**: ARM64 architecture (Axion) was used due to temporary unavailability of `e2` machine types in the region. Software packages (Wazuh, Suricata) were installed via their official repositories, which natively support ARM64 without additional configuration.
