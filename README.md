# Lab 135 – Linux Firewall Configuration Change Investigation

## Overview

This lab focuses on investigating Linux firewall configuration and administrative activity using firewalld, Auditd, and Wazuh on Rocky Linux 9.8.

Firewalls control network traffic entering and leaving a system. Changes to firewall rules can affect network exposure, allow previously restricted services, or weaken existing security controls.

The investigation examines firewall zones, runtime and permanent configurations, administrative commands, and available security telemetry.

A controlled firewall investigation workflow is followed to understand how firewall-related activity can be identified and correlated with Wazuh events.

## Objectives

- Understand Linux firewall architecture and firewalld.
- Examine active firewall zones, services, and permitted ports.
- Understand runtime and permanent firewall configurations.
- Establish a baseline of the existing firewall state.
- Monitor permanent firewall configuration files using Auditd.
- Examine firewall administrative activity.
- Investigate sudo and firewall-related events in Wazuh.
- Distinguish firewall rule queries from actual configuration modifications.
- Understand the limitations of file-based firewall monitoring.
- Preserve the original firewall configuration during cleanup.

## Environment

| Component | Details |
|---|---|
| Operating System | Rocky Linux 9.8 |
| Firewall Management | firewalld |
| Package Manager | DNF |
| Audit Framework | Auditd 3.1.5 |
| SIEM | Wazuh 4.13.1 |
| Firewall Configuration | `/etc/firewalld/` |
| Audit Log | `/var/log/audit/audit.log` |
| System Logging | journald |

## Investigation Scope

The investigation focuses on the following areas:

- Firewall service status.
- Default and active firewall zones.
- Allowed services and ports.
- Runtime firewall configuration.
- Permanent firewall configuration.
- Administrative commands executed through sudo.
- Auditd records associated with permanent configuration changes.
- Wazuh telemetry related to firewall administration.

## Firewall Configuration Monitoring

The permanent firewall configuration directory is:

`/etc/firewalld/`

A targeted Auditd watch can monitor write and attribute-change activity:

```bash
sudo auditctl -w /etc/firewalld/ -p wa -k lab135_firewall_changes
```

This provides visibility into relevant filesystem modifications under the monitored directory.

Runtime firewall operations do not necessarily modify files in this directory. Therefore, file monitoring alone cannot identify every firewall rule change.

## Investigation Workflow

1. Verify firewalld service status.
2. Identify the default and active zones.
3. Examine existing runtime firewall rules.
4. Examine permanent firewall rules.
5. Identify firewall configuration files.
6. Record configuration metadata and checksums.
7. Verify Auditd status.
8. Configure targeted firewall directory monitoring.
9. Examine firewall-related administrative commands.
10. Review relevant Auditd and system log records.
11. Validate available telemetry in Wazuh.
12. Restore the original firewall state and remove temporary monitoring rules.

## Wazuh Event Analysis

A Wazuh event was observed on November 12, 2026.

| Field | Observed value |
|---|---|
| Timestamp | Nov 12, 2026, 08:07:25.058 |
| Agent ID | 000 |
| Agent Name | localhost.localdomain |
| Location | journald |
| Decoder | sudo |
| Command | `/bin/firewall-cmd --query-port=8080/tcp` |
| Source User | root |
| Destination User | root |
| Rule ID | 5402 |
| Rule Description | Successful sudo to ROOT executed. |
| Rule Level | 3 |
| MITRE Technique | T1548.003 – Sudo and Sudo Caching |

The event confirms execution of a firewall port query through sudo.

It does not establish whether port 8080 was allowed or whether any firewall configuration was modified.

## Key Investigation Distinctions

- A firewall rule query is not a firewall configuration change.
- A runtime modification is not necessarily a permanent modification.
- A permanent configuration change may require a reload before becoming active.
- A firewall rule allowing a port does not prove that a service is listening.
- An allowed port does not prove that a remote connection occurred.
- A sudo event confirms administrative command execution, not necessarily a successful firewall modification.

## Conclusion

This lab demonstrates how Linux firewall administration can be examined using firewalld, Auditd, system logs, and Wazuh.

The investigation emphasizes evidence correlation and accurate event interpretation. Administrative activity, firewall configuration changes, and network access must be treated as separate observations.

The final assessment should be based on the actual firewall state, audit records, and available SIEM telemetry rather than assumptions based on a single event.
