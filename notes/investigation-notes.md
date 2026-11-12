# Lab 135 – Investigation Notes

## Investigation Overview

**Investigation:** Linux Firewall Configuration Change Investigation

**Lab:** 135

**Operating System:** Rocky Linux 9.8

**Firewall Management:** firewalld

**Monitoring:** Auditd and Wazuh 4.13.1

**Primary Configuration Directory:** `/etc/firewalld/`

## 1. Investigation Objective

The objective is to examine Linux firewall administrative activity and identify evidence of firewall configuration changes.

The investigation focuses on understanding the current firewall state, examining administrative commands, and validating relevant security telemetry in Wazuh.

## 2. Firewall Discovery

The initial investigation examines the firewalld service and its operational state.

Relevant commands:

```bash
sudo systemctl status firewalld --no-pager
sudo firewall-cmd --state
```

The default and active zones are identified using:

```bash
sudo firewall-cmd --get-default-zone
sudo firewall-cmd --get-active-zones
```

These details establish which firewall policies are associated with the system.

## 3. Runtime and Permanent Configuration

The runtime configuration represents the currently active firewall rules.

The permanent configuration represents settings saved for persistence across reloads and restarts.

Runtime inspection:

```bash
sudo firewall-cmd --list-all
```

Permanent configuration inspection:

```bash
sudo firewall-cmd --permanent --list-all
```

The two configurations must be examined separately because they may not contain identical rules.

## 4. Firewall Configuration Baseline

The primary configuration directory is:

`/etc/firewalld/`

Relevant inspection commands:

```bash
sudo ls -lah /etc/firewalld/
sudo find /etc/firewalld -maxdepth 2 -type f
```

File metadata and checksums can provide a baseline:

```bash
sudo stat /etc/firewalld/firewalld.conf
sudo sha256sum /etc/firewalld/firewalld.conf
```

Zone configuration checksums:

```bash
sudo find /etc/firewalld/zones -type f -exec sha256sum {} \;
```

These values can help identify changes to permanent configuration files.

## 5. Auditd Monitoring

A targeted Auditd watch can monitor permanent firewall configuration changes:

```bash
sudo auditctl -w /etc/firewalld/ -p wa -k lab135_firewall_changes
```

The rule monitors write and attribute-change operations.

Auditd records can provide information about:

- Affected file paths.
- Executables.
- Process names.
- User identities.
- System calls.
- Operation success or failure.

Search for related records:

```bash
sudo ausearch -k lab135_firewall_changes -i
```

A directory watch does not guarantee visibility into runtime-only firewall changes.

## 6. Wazuh Event Observation

A Wazuh event was observed on November 12, 2026.

### Event details

| Field | Value |
|---|---|
| Timestamp | Nov 12, 2026 @ 08:07:25.058 |
| Index | `wazuh-alerts-4.x-2026.11.12` |
| Agent ID | `000` |
| Agent Name | `localhost.localdomain` |
| Manager | `localhost.localdomain` |
| Location | `journald` |
| Decoder | `sudo` |
| Program | `sudo` |
| Command | `/bin/firewall-cmd --query-port=8080/tcp` |
| Source User | `root` |
| Destination User | `root` |
| Working Directory | `/root` |
| TTY | `pts/0` |
| Rule ID | `5402` |
| Rule Level | `3` |
| Rule Description | `Successful sudo to ROOT executed.` |

### Raw event

```text
Nov 12 02:37:23 localhost.localdomain sudo[8727]: root : TTY=pts/0 ; PWD=/root ; USER=root ; COMMAND=/bin/firewall-cmd --query-port=8080/tcp
```

### Event interpretation

The event confirms that the root user executed:

```bash
sudo firewall-cmd --query-port=8080/tcp
```

The command checks whether TCP port 8080 is allowed in the relevant runtime firewall zone.

The Wazuh rule classified the activity as successful sudo execution to root.

The event does not contain the result returned by `firewall-cmd`.

Therefore, the following conclusions cannot be established from this event alone:

- Whether port 8080 was allowed.
- Whether port 8080 was blocked.
- Whether a firewall rule was added.
- Whether a permanent firewall configuration was modified.
- Whether any remote system connected to port 8080.

### MITRE ATT&CK mapping

The event contains the following Wazuh MITRE mapping:

- Technique: T1548.003
- Technique Name: Sudo and Sudo Caching
- Tactics: Privilege Escalation, Defense Evasion

This mapping describes the sudo-related event classification. It does not establish malicious privilege escalation.

## 7. Event Timestamp Correlation

The Wazuh event contains:

- Displayed timestamp: November 12, 2026, 08:07:25.058.
- Raw journal timestamp: November 12, 2026, 02:37:23.

The difference is consistent with the local timezone offset of UTC+05:30.

Both timestamps refer to the same recorded sudo event.

## 8. Evidence Assessment

### Confirmed

- Wazuh received a sudo event through journald.
- The command recorded was `firewall-cmd --query-port=8080/tcp`.
- The source and destination user fields were both `root`.
- Wazuh classified the event using rule ID 5402.
- The rule level was 3.

### Not established by the available event

- Whether port 8080 was allowed.
- Whether a firewall rule was added or removed.
- Whether permanent firewall configuration changed.
- Whether the firewall service was restarted or reloaded.
- Whether a service was listening on port 8080.
- Whether network traffic reached the server through that port.

## 9. Further Evidence Required

To establish whether an actual firewall configuration change occurred, correlate the following:

1. Runtime firewall state before and after the activity.
2. Permanent firewall state before and after the activity.
3. Auditd records for relevant configuration files.
4. firewalld journal messages.
5. Sudo command records for add, remove, and reload operations.
6. Relevant network connection evidence, if investigating actual access.

A query command must not be reported as a configuration modification.

## 10. Investigation Conclusion

The available Wazuh event confirms administrative execution of a firewall port-status query for TCP port 8080.

It provides visibility into the command, user context, and sudo activity.

However, the event does not establish a firewall configuration change or malicious activity.

The investigation should remain classified as an examination of firewall administrative activity unless additional evidence confirms an actual configuration modification.
