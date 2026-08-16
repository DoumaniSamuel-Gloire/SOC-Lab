# Incident 001 — SSH Brute Force

## 1. Objective
Simulate an SSH brute-force attack in a controlled lab environment and evaluate Wazuh's ability to detect the activity.

## 2. Lab Environment
| Component | Details |
|---|---|
| Attacker | 10.10.0.4 |
| Target | 10.10.0.3 |
| Manager | 10.10.0.2 |
| Target OS | Ubuntu |
| SIEM | Wazuh |
| Service | SSH |
| Attack Tool | Hydra |

## 3. Attack Simulation
A controlled SSH brute-force attack was launched from the attacker VM against the target VM using Hydra, with a custom wordlist containing the real account password.

```bash
hydra -l <user> -P wordlist.txt ssh://10.10.0.3 -t 8 -V
```

## 4. Detection
Wazuh detected the activity through several rules:

| Rule ID | Description | Level |
|---|---|---|
| 5760 | sshd: authentication failed | 5 |
| 5710 | Attempt to login using a non-existent user | 5 |
| 40112 | Multiple authentication failures followed by a success | 12 |

## 5. Alert Analysis
Rule 40112 is the critical alert — it doesn't just count failures, it specifically detects the pattern *repeated failures → success*, the signature of a successful brute-force attack.

`rule.frequency: 2` confirms the correlation was based on two conditions (failures + success).

## 6. Investigation
Post-compromise verification was performed on the target:
- `.bash_history` — no malicious commands, only legitimate lab administration commands
- `authorized_keys` — no unauthorized key added
- `crontab` — no scheduled persistence

**Conclusion**: authentication succeeded, but no interactive shell session followed — consistent with Hydra's automated behavior (it validates credentials and closes the connection without maintaining a session).

## 7. MITRE ATT&CK
| Technique | Tactics |
|---|---|
| T1110 — Brute Force (T1110.001 Password Guessing) | Credential Access |
| T1078 — Valid Accounts | Defense Evasion, Persistence, Privilege Escalation, Initial Access |

## 8. Incident Response
Full NIST-based cycle applied:
1. **Containment** — compromised account locked (`passwd -l`); validated by re-running Hydra with the correct password, which then failed
2. **Investigation** — see section 6
3. **Eradication** — password changed, account unlocked
4. **Lessons Learned** — see section 9

## 9. Lessons Learned
- Weak, short passwords fall in seconds against even a small wordlist
- "Successful authentication" does not automatically mean "active compromise" — manual session verification is required
- Correlation rules (failures + success) provide a much stronger signal than isolated failure alerts

## 10. Recommendations
- Enforce strong password policy or disable password authentication (`PasswordAuthentication no`)
- Use SSH key-based authentication
- Implement brute-force protection (e.g. Fail2ban)
- Monitor repeated authentication failures via SIEM correlation rules
