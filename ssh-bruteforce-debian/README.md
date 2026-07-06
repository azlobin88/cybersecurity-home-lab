[alert-5763.json](https://github.com/user-attachments/files/29715972/alert-5763.json)
<img width="1879" height="956" alt="Снимок" src="https://github.com/user-attachments/assets/9e8bb309-92c6-4a57-89ce-b84027bb5aea" />
[IR-Report_SSH-Bruteforce-Debian_RU.md](https://github.com/user-attachments/files/29715675/IR-Report_SSH-Bruteforce-Debian_RU.md)
[IR-Report_SSH-Bruteforce-Debian.md](https://github.com/user-attachments/files/29715676/IR-Report_SSH-Bruteforce-Debian.md)
{
  "_index": "wazuh-alerts-4.x-2026.07.06",
  "_id": "Z-crOJ8B6XuHFEhKs4V1",
  "_score": null,
  "_source": {
    "predecoder": {
      "hostname": "Debian",
      "program_name": "sshd-session",
      "timestamp": "Jul 06 16:03:33"
    },
    "input": {
      "type": "log"
    },
    "agent": {
      "ip": "192.168.1.196",
      "name": "debian",
      "id": "001"
    },
    "previous_output": "Jul 06 16:03:32 Debian sshd-session[8292]: Failed password for debian from 192.168.1.210 port 46624 ssh2\nJul 06 16:03:32 Debian sshd-session[8295]: Failed password for debian from 192.168.1.210 port 46648 ssh2\nJul 06 16:03:32 Debian sshd-session[8308]: Failed password for debian from 192.168.1.210 port 46702 ssh2\nJul 06 16:03:32 Debian sshd-session[8293]: Failed password for debian from 192.168.1.210 port 46622 ssh2\nJul 06 16:03:32 Debian sshd-session[8309]: Failed password for debian from 192.168.1.210 port 46708 ssh2\nJul 06 16:03:32 Debian sshd-session[8303]: Failed password for debian from 192.168.1.210 port 46666 ssh2\nJul 06 16:03:32 Debian sshd-session[8297]: Failed password for debian from 192.168.1.210 port 46660 ssh2",
    "data": {
      "srcip": "192.168.1.210",
      "dstuser": "debian",
      "srcport": "46682"
    },
    "manager": {
      "name": "wazuh-server"
    },
    "rule": {
      "mail": false,
      "level": 10,
      "hipaa": [
        "164.312.b"
      ],
      "pci_dss": [
        "11.4",
        "10.2.4",
        "10.2.5"
      ],
      "tsc": [
        "CC6.1",
        "CC6.8",
        "CC7.2",
        "CC7.3"
      ],
      "description": "sshd: brute force trying to get access to the system. Authentication failed.",
      "groups": [
        "syslog",
        "sshd",
        "authentication_failures"
      ],
      "nist_800_53": [
        "SI.4",
        "AU.14",
        "AC.7"
      ],
      "frequency": 8,
      "gdpr": [
        "IV_35.7.d",
        "IV_32.2"
      ],
      "firedtimes": 1,
      "mitre": {
        "technique": [
          "Brute Force"
        ],
        "id": [
          "T1110"
        ],
        "tactic": [
          "Credential Access"
        ]
      },
      "id": "5763"
    },
    "location": "journald",
    "decoder": {
      "parent": "sshd",
      "name": "sshd"
    },
    "id": "1783353814.65323",
    "full_log": "Jul 06 16:03:33 Debian sshd-session[8306]: Failed password for debian from 192.168.1.210 port 46682 ssh2",
    "timestamp": "2026-07-06T16:03:34.432+0000"
  },
  "fields": {
    "timestamp": [
      "2026-07-06T16:03:34.432Z"
    ]
  },
  "sort": [
    1783353814432
  ]
}
