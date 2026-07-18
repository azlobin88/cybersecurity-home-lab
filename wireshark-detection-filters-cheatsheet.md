# Wireshark Detection Filters Cheatsheet
### Шпаргалка по фильтрам Wireshark для детект-сценариев

A quick-reference of Wireshark display filters organized by SOC detection scenario: port scanning, suspicious DNS activity, and data exfiltration.

Краткий справочник фильтров Wireshark, сгруппированных по сценариям детектирования: сканирование портов, подозрительный DNS, эксфильтрация данных.

---

## 1. Port Scanning / Сканирование портов

**EN:** Attackers probe multiple ports/hosts to map open services before exploitation.
**RU:** Атакующий перебирает порты/хосты, чтобы найти открытые сервисы перед эксплуатацией.

| Filter | What it shows / Что показывает |
|---|---|
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | SYN packets only — classic sign of a SYN scan / только SYN-пакеты — признак SYN-скана |
| `tcp.flags.reset == 1` | RST responses — many in a row from one host = closed ports being probed / много RST от одного хоста = перебор закрытых портов |
| `tcp.flags.syn==1 and tcp.flags.ack==0 and ip.src==<IP>` | Isolate scan attempts from a specific source / Выделить попытки скана от конкретного источника |
| `tcp.analysis.retransmission` | Excessive retransmissions — can indicate scanning against filtered ports / Избыточные ретрансмиссии — возможный признак скана против фильтруемых портов |
| `icmp.type == 3 && icmp.code == 3` | ICMP "port unreachable" — common in UDP scans / ICMP "порт недоступен" — типично для UDP-сканов |

**Statistics tip / Совет по статистике:** `Statistics → Conversations`, sort by number of ports per source IP — a single IP touching dozens of ports in seconds is a strong scan indicator.
`Статистика → Разговоры (Conversations)`, сортировка по числу портов на исходный IP — если один IP за секунды достучался до десятков портов, это явный признак скана.

---

## 2. Suspicious DNS / Подозрительный DNS

**EN:** Malware often uses DNS for C2 beaconing, tunneling, or DGA (domain generation algorithms).
**RU:** Вредоносное ПО часто использует DNS для C2-связи, туннелирования или DGA (алгоритмов генерации доменов).

| Filter | What it shows / Что показывает |
|---|---|
| `dns` | All DNS traffic / Весь DNS-трафик |
| `dns.qry.name contains "xn--"` | Punycode/IDN domains, sometimes used for phishing lookalikes / Punycode-домены, иногда используются для фишинговых подделок |
| `dns.flags.rcode != 0` | Failed DNS lookups — many NXDOMAIN can indicate DGA malware / Неудачные DNS-запросы — много NXDOMAIN может указывать на DGA-вредонос |
| `dns.qry.name matches "^[a-z0-9]{16,}\\."` | Long random-looking subdomains — typical of DNS tunneling / Длинные "случайные" поддомены — типичный признак DNS-туннелирования |
| `dns.qry.type == 16` | TXT record queries — a common DNS tunneling / C2 channel / Запросы TXT-записей — частый канал DNS-туннелирования или C2 |
| `dns.count.answers > 10` | Unusually large answer sets / Необычно большое число ответов в одном пакете |

**Behavioral tip / Поведенческий признак:** repeated queries to the same domain at fixed intervals = classic C2 beaconing pattern. Check `Statistics → DNS` for query frequency per domain.
Повторяющиеся запросы к одному домену с фиксированным интервалом — классический паттерн C2-маячка (beaconing). Смотри частоту запросов по доменам в `Статистика → DNS`.

---

## 3. Data Exfiltration / Эксфильтрация данных

**EN:** Look for unusually large or unusual-direction data transfers, especially over uncommon ports or protocols.
**RU:** Ищи аномально большие передачи данных или необычное направление трафика, особенно по нестандартным портам/протоколам.

| Filter | What it shows / Что показывает |
|---|---|
| `tcp.len > 1000` | Large TCP payloads — combine with a specific IP to spot bulk transfers / Большие TCP-пакеты — в сочетании с конкретным IP помогает найти массовую передачу |
| `http.request.method == "POST" && http.content_length > 100000` | Large HTTP POST bodies — possible upload of stolen data / Большие тела HTTP POST-запросов — возможная выгрузка украденных данных |
| `ftp-data` | FTP data transfer traffic / Трафик передачи данных по FTP |
| `tcp.port == 443 && ip.dst == <external IP>` | Encrypted outbound traffic to an unusual destination — needs correlation with reputation data / Зашифрованный исходящий трафик на подозрительный адрес — требует сверки с репутационными данными |
| `dns.qry.name contains "base64"` or long encoded-looking subdomains | Data encoded in DNS queries (tunneling-based exfil) / Данные, закодированные в DNS-запросах (эксфильтрация через туннель) |
| `frame.len > 1400 && tcp.flags.push == 1` | Large pushed frames — worth reviewing during off-hours traffic spikes / Крупные frame-ы с флагом PUSH — стоит проверить при всплесках трафика в нерабочее время |

**Statistics tip / Совет по статистике:** `Statistics → Conversations → sort by Bytes` to quickly find the largest data flows, then filter on that IP/port pair.
`Статистика → Разговоры → сортировка по Bytes`, чтобы быстро найти самые объёмные потоки, затем отфильтровать по этой паре IP/порт.

---

## Quick combo filters / Быстрые комбинированные фильтры

```
# Traffic to/from a specific suspicious host
ip.addr == <IP>

# Only new connections in a time window
tcp.flags.syn == 1 && frame.time >= "2026-07-18 00:00:00"

# Exclude noisy broadcast/multicast traffic while hunting
!(ip.dst == 255.255.255.255) && !(ip.dst[0:3] == 224.0.0)
```

---

*Part of the [cybersecurity-home-lab](https://github.com/azlobin88/cybersecurity-home-lab) portfolio project.*
*Часть портфолио-проекта [cybersecurity-home-lab](https://github.com/azlobin88/cybersecurity-home-lab).*
