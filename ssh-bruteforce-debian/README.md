# Incident Report: SSH Brute-Force Attack Against Debian Host

**Environment:** Home SOC Lab (VirtualBox, Bridged Adapter networking)
**SIEM:** Wazuh 4.14.5
**Analyst:** Alex

---

## Time of Activity

- **Start:** 2026-07-06 19:03:32 MSK (16:03:32 UTC)
- **End:** 2026-07-06 19:03:36 MSK (16:03:36 UTC)
- **Duration:** ~4 seconds
- **Total events generated:** 125 (Wazuh alert count for the affected agent during the activity window)

## Affected Entities

| Entity | Value |
|---|---|
| Target host | Debian (agent name: `debian`, agent ID: 001) |
| Target IP | 192.168.1.196 |
| Targeted account | `debian` |
| Source host | Kali Linux (attacker) |
| Source IP | 192.168.1.210 |
| Attack tool | THC-Hydra v9.7 |
| Attack vector | SSH (port 22) |

## Summary

A password-guessing (brute-force) attack was conducted against the SSH service on host `debian` from source IP `192.168.1.210`. The attacking host issued a rapid sequence of authentication attempts against the `debian` account using a curated password list. Wazuh detected the activity via the `sshd` and `pam_unix`/`unix_chkpwd` log sources and correctly escalated it from individual authentication-failure events into a single aggregated brute-force alert.

![Wazuh Alerts](screenshots/wazuh-alerts-overview.png)

Full alert JSON: [alert-5763.json](./alert-5763.json)

## Reason for Classification

Multiple corroborating log sources confirmed malicious, automated credential-guessing behavior rather than a legitimate user mistyping a password:

- **Rule 5760** (`sshd: authentication failed`, level 5) fired repeatedly — individual failed login attempts.
- **Rule 5763** (`sshd: brute force trying to get access to the system. Authentication failed.`, level 10) fired after Wazuh's frequency-based correlation detected 8+ failures in a short window (`frequency: 8`), consistent with automated tooling rather than human input.
- **Rule 5557** (`unix_chkpwd: Password check failed`, level 5) confirmed the failures were also processed at the PAM layer, ruling out an SSH-daemon-only anomaly.
- Raw log evidence (`previous_output`) shows multiple parallel connections from the same source IP within the same one-second window, each using a different source port (e.g., 46622, 46624, 46648, 46660, 46666, 46682, 46702, 46708) — a strong indicator of a multi-threaded automated tool rather than manual login attempts.
- MITRE ATT&CK mapping: **T1110 (Brute Force)** / **T1110.001 (Password Guessing)**, Tactic: **Credential Access**.

## Reason for Escalation

- Alert severity jumped from level 5 (single failed login) to **level 10** once Wazuh's correlation engine identified the pattern as brute-force, which is treated as a high-priority credential-access event in most SOC severity matrices.
- The account targeted (`debian`) is a valid, existing local account — a successful compromise would have granted the attacker direct shell access to the host.
- The attack pattern (parallel connections, multiple source ports, sub-second timing) is consistent with adversary tooling (e.g., Hydra, Medusa) rather than benign misconfiguration or user error, warranting containment action rather than passive monitoring.

## Remediation

Aligned with CERT Société Générale **IRM-3 (Unix/Linux Intrusion Detection)** containment/eradication/recovery guidance:

**Containment**
- Block source IP `192.168.1.210` at the host firewall / `iptables` (or `fail2ban` ban rule) pending investigation.
- Temporarily disable or monitor the `debian` account for further suspicious activity.

**Eradication**
- Confirm the account password was not compromised (in this lab exercise, the credential was not present in the attacker's wordlist, and no successful authentication occurred).
- Rotate the account password as a precaution.
- Check `~/.ssh/authorized_keys` for unauthorized key entries (persistence check).

**Recovery / Hardening**
- Deploy **fail2ban** on the SSH service to automatically rate-limit and ban repeat offenders.
- Disable SSH password authentication in favor of key-based authentication (`PasswordAuthentication no` in `sshd_config`).
- Enforce MFA for SSH access where feasible.
- Consider moving SSH off the default port 22 as a low-value deterrent against opportunistic scanning (not a substitute for the controls above).

## Indicators of Compromise (IOCs)

| Type | Value |
|---|---|
| Source IP | 192.168.1.210 |
| Target IP | 192.168.1.196 |
| Targeted account | debian |
| Service/Port | SSH / TCP 22 |
| Tool signature | THC-Hydra v9.7 |
| Wazuh Rule IDs | 5760, 5763, 5557 |
| MITRE ATT&CK | T1110, T1110.001 |
| Time window (UTC) | 2026-07-06 16:03:32–16:03:36 |

---
*Note: This incident was generated as part of a controlled home-lab exercise to validate Wazuh SIEM detection coverage for SSH brute-force activity (VirtualBox lab: Wazuh SIEM server, Debian victim, Kali attacker, Bridged Adapter networking).*
