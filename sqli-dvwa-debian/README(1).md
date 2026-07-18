# SQL Injection Enumeration — DVWA / SQL-инъекция (Enumeration) — DVWA

**EN:** Manual UNION-based SQL injection against DVWA, detected via a two-level Wazuh rule chain (built-in atomic signature → custom frequency-based correlation rule). Simulates the enumeration phase of an attack: an attacker probing column counts, then walking `information_schema` to map the database before a targeted data-extraction attempt.

**RU:** Ручная UNION-based SQL-инъекция против DVWA, задетектированная через двухуровневую цепочку правил Wazuh (встроенная атомарная сигнатура → кастомное корреляционное правило по частоте). Симулирует фазу enumeration атаки: атакующий определяет количество колонок, затем перечисляет `information_schema`, чтобы картировать базу данных перед прицельной попыткой извлечения данных.

---

## Environment / Окружение

| | |
|---|---|
| Target / Цель | DVWA on Debian VM (Apache + MariaDB) — 192.168.1.196 |
| SIEM | Wazuh manager — 192.168.1.153 |
| Attack source / Источник атаки | 192.168.1.133 |

## Detection summary / Кратко о детектировании

- **Rule 31103 / 31106** (built-in) — atomic SQLi signature + confirmed HTTP 200 (successful injection) / атомарная SQLi-сигнатура + подтверждённый HTTP 200 (успешная инъекция)
- **Rule 100101** (custom) — level 12, frequency=5 / timeframe=120s, `same_source_ip` — flags enumeration behavior, not just a single attempt / кастомное правило — уровень 12, частота=5 / окно=120с — фиксирует именно поведение enumeration, а не единичную попытку
- MITRE ATT&CK: **T1190** (Exploit Public-Facing Application), **T1590** (Gather Victim Network Information)

## Files / Файлы

| File / Файл | Description / Описание |
|---|---|
| `incident-report-sqli-dvwa-EN.md` | Full incident report (English) — IRM-3 format / Полный отчёт об инциденте (английский) |
| `incident-report-sqli-dvwa-RU.md` | Full incident report (Russian) — IRM-3 format / Полный отчёт об инциденте (русский) |
| `alert-31106.PNG` / `.json` | Atomic signature alert — single successful injection / Алерт атомарной сигнатуры — одна успешная инъекция |
| `alert-100101.PNG` / `.json` | Correlation alert — 5-request enumeration chain / Корреляционный алерт — цепочка из 5 запросов enumeration |
| `wazuh-alerts-overview.PNG` | Discover view showing the full alert timeline / Общий вид в Discover с таймлайном алертов |

---

*Part of the [cybersecurity-home-lab](https://github.com/azlobin88/cybersecurity-home-lab) portfolio project.*
*Часть портфолио-проекта [cybersecurity-home-lab](https://github.com/azlobin88/cybersecurity-home-lab).*
