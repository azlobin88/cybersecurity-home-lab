# SOC Home Lab — Incident Reports & Detection Engineering

Домашняя SOC-лаборатория (VirtualBox): Wazuh SIEM + Suricata IDS + Zeek NSM.
Эмуляция атак с Kali → детектирование → расследование → отчёт.

## Архитектура
- Wazuh 4.14.5 (SIEM) | Debian (victim, Wazuh agent, DVWA) | Kali (attacker)
- Suricata (IDS) + Zeek + JA4 (NSM), парсинг conn/dns/http.log через ossec.conf + Community-id.

## Отчёты по инцидентам

| # | Инцидент | Техника (MITRE) | Инструмент | Отчёт |
|---|----------|-----------------|------------|-------|
| 1 | SSH Brute-Force | T1110.001 | Hydra | [→](./ssh-bruteforce-debian) |
| 2 | Boss_of_the_SOC | - | Splunk | [→](./Boss_of_the_SOC) |
| 3 | SQL Injection | T1190 | DVWA | [→](./sqli-dvwa-debian) |
| 4 | Malicious C2 | T1071.001 | Wireshark | [→](./malware-traffic-analysis/easyas123) |
| 5 | Lockdown Lab | - | Wireshark | [→](./Lockdown_Lab) |
| 6 | RetailBreach | - | Wireshark | [→](./RetailBreach) |

## Стек и навыки
Wazuh · Suricata · Zeek · MITRE ATT&CK · Wireshark · Linux

