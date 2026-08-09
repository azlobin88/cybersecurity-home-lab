# SOC Home Lab — Incident Reports & Detection Engineering

Домашняя SOC-лаборатория (VirtualBox): Wazuh SIEM + Suricata IDS + Zeek NSM.
Эмуляция атак с Kali → детектирование → расследование → отчёт.

## Архитектура
![lab diagram](docs/architecture.png)
- Wazuh 4.14.5 (SIEM) | Debian (victim, Wazuh agent) | Kali (attacker)
- Suricata (IDS) + Zeek (NSM), парсинг conn/dns/http.log через ossec.conf

## Отчёты по инцидентам

| # | Инцидент | Техника (MITRE) | Инструмент | Отчёт |
|---|----------|-----------------|------------|-------|
| 1 | SSH Brute-Force | T1110.001 | Hydra | [→](./ssh-bruteforce-debian) |
| 2 | Network Scan | T1046 | Nmap | [→](./...) |
| 3 | SQL Injection | T1190 | DVWA | [→](./sqli-dvwa-debian) |
| 4 | DNS Tunneling | T1071.004 | — | [→](./...) |

## Стек и навыки
Wazuh · Suricata · Zeek · MITRE ATT&CK · Wireshark · Linux
