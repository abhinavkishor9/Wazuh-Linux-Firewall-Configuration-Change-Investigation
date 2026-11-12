#  Investigation Timeline
| Stage | Activity | Evidence / Purpose |
|---|---|---|
| 01 | Firewall service discovery | Identify firewalld service status. |
| 02 | Zone discovery | Identify the default and active firewall zones. |
| 03 | Runtime inspection | Examine currently active firewall rules. |
| 04 | Permanent configuration inspection | Examine persistent firewall rules. |
| 05 | Configuration discovery | Identify files under `/etc/firewalld/`. |
| 06 | Baseline collection | Record firewall configuration metadata and checksums. |
| 07 | Auditd verification | Examine audit subsystem status. |
| 08 | Auditd monitoring | Configure a targeted firewall configuration watch. |
| 09 | Firewall administration review | Examine commands used to query or modify firewall rules. |
| 10 | Wazuh event analysis | Review sudo and firewall-related telemetry. |
| 11 | Evidence correlation | Compare firewall state, audit records, and SIEM events. |
| 12 | Restoration and cleanup | Preserve the original firewall configuration and remove temporary monitoring rules. |

## Confirmed Wazuh Event

| Field | Value |
|---|---|
| Date | November 12, 2026 |
| Time | 08:07:25.058 |
| Agent | localhost.localdomain |
| Agent ID | 000 |
| Location | journald |
| Decoder | sudo |
| Command | `/bin/firewall-cmd --query-port=8080/tcp` |
| User | root |
| Rule ID | 5402 |
| Rule Level | 3 |
| Description | Successful sudo to ROOT executed. |

Raw journal timestamp:

`Nov 12 02:37:23`

The raw journal timestamp and displayed Wazuh timestamp are consistent with the UTC+05:30 local timezone.

