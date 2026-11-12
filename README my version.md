# Wazuh-Linux-Firewall-Configuration-Change-Investigation
## Overview
### Introduction

A Linux firewall is a security control that regulates network traffic entering and leaving a system. It uses rules to determine which connections are permitted and which are restricted.

On Rocky Linux, firewalld provides a management interface for configuring firewall zones, services, ports, and network access policies.

Monitoring firewall configuration changes is important because unexpected modifications can weaken network security and expose services that were previously restricted.

### 1. Linux Firewall Architecture

Firewalld manages firewall policies through zones and configuration rules.

It works with the underlying Linux packet-filtering framework to enforce network traffic decisions.

Important components include:

- firewalld – Firewall management service.
- Zones – Define different trust levels and network policies.
- Services – Predefined firewall service definitions.
- Ports – Individual TCP or UDP port permissions.
- nftables – Packet-filtering framework used by the Linux kernel on modern Rocky Linux installations.

### 2. Firewall Zones

Zones represent different levels of trust and determine how network traffic is handled.

Common zones include:

- public
- home
- work
- internal
- trusted
- drop
- block

The default zone and active zone assignments are important investigation details because they determine which firewall policy applies to network interfaces.

### 3. Runtime and Permanent Configuration

Firewalld maintains two configuration states.

#### Runtime configuration

Runtime configuration represents the firewall rules currently active in memory.

Changes generally take effect immediately but may be lost after a reload or restart unless saved permanently.

#### Permanent configuration

Permanent configuration stores settings intended to persist across reloads and restarts.

Permanent changes generally require a firewalld reload before becoming active.

An analyst must examine both states to understand whether a rule is currently active and whether it has been configured to persist.

### 4. Firewall Configuration Changes

Firewall configuration changes include:

- Adding a permitted port.
- Removing an existing restriction.
- Enabling or disabling a service.
- Changing the default zone.
- Modifying interface-to-zone assignments.
- Changing permanent firewall configuration files.
- Disabling the firewall service.

These activities can affect the system's network exposure.

However, a firewall change may also be part of legitimate system administration. Context and authorization must be considered.

### 5. Role of Auditd

Auditd provides security auditing capabilities on Linux.

A directory watch can monitor permanent firewall configuration changes:

```bash
sudo auditctl -w /etc/firewalld/ -p wa -k lab135_firewall_changes
```

This monitors write and attribute-change activity under the selected directory.

Auditd records can help identify the affected file, executable, process, user identity, and operation result.

Directory monitoring does not guarantee detection of every runtime-only firewall change because those operations may not modify permanent configuration files.

### 6. Role of System Logs

System logs can provide additional context about firewalld activity.

For example:

```bash
sudo journalctl -u firewalld --since "15 minutes ago"
```

These records can supplement Auditd evidence.

The availability and detail of journal messages depend on the operation and system logging configuration.

### 7. Role of Wazuh

Wazuh collects and analyzes security telemetry.

It can help analysts examine sudo events, Auditd records, and other system activity.

A Wazuh event should be interpreted according to its raw log, decoder, rule description, and available fields.

For example, a sudo event containing:

```text
firewall-cmd --query-port=8080/tcp
```

confirms execution of a firewall status query. It does not prove that the port was opened or that firewall configuration was modified.

### 8. Investigation Methodology

A firewall investigation should follow a structured process:

1. Verify firewalld service status.
2. Identify default and active zones.
3. Collect runtime and permanent firewall configurations.
4. Establish a baseline of configuration files.
5. Configure targeted Auditd monitoring.
6. Examine firewall-related administrative commands.
7. Investigate available audit and system logs.
8. Validate relevant telemetry in Wazuh.
9. Correlate the evidence and identify limitations.
10. Restore the original firewall state and remove temporary monitoring rules.

### 9. Evidence Interpretation

Different evidence sources establish different facts.

- Firewall queries establish the result of a status check when the command output is available.
- Configuration files establish saved firewall settings.
- Auditd records provide evidence of monitored system operations.
- Sudo events provide context about privileged command execution.
- Journal records provide available firewalld service messages.
- Network connection evidence helps establish whether communication actually occurred.

A rule being allowed does not prove that a service was listening or that a remote system connected successfully.

### Conclusion

Linux firewall monitoring provides visibility into changes that can affect network security.

An effective investigation combines firewall state, configuration integrity, administrative activity, Auditd records, system logs, and Wazuh telemetry.

The objective is to establish what happened and what the available evidence proves, without treating every administrative command as malicious activity.

This lab focuses on investigating Linux firewall configuration and administrative activity using firewalld, Auditd, and Wazuh on Rocky Linux 9.8.

Firewalls control network traffic entering and leaving a system. Changes to firewall rules can affect network exposure, allow previously restricted services, or weaken existing security controls.

The investigation examines firewall zones, runtime and permanent configurations, administrative commands, and available security telemetry.

A controlled firewall investigation workflow is followed to understand how firewall-related activity can be identified and correlated with Wazuh events.

## Lab Objectives

### Primary Goal

Develop practical skills for examining Linux firewall administration and investigating changes that may affect a system's network security posture.

### Technical Objectives

1. Understand the purpose and operation of firewalld on Rocky Linux.
2. Identify the default firewall zone and active network interfaces.
3. Examine allowed services and TCP/UDP ports.
4. Understand the difference between runtime and permanent firewall settings.
5. Establish a baseline of the existing firewall configuration.
6. Identify the configuration files responsible for persistent firewall settings.
7. Configure Auditd to monitor relevant firewall configuration changes.
8. Examine privileged firewall-related command execution.
9. Investigate available system and audit records.
10. Review sudo and firewall-related events in Wazuh.
11. Correlate firewall state with administrative and audit telemetry.
12. Restore the original firewall configuration and validate cleanup.

### Investigation Skills

- Recognize potentially suspicious firewall modifications.
- Distinguish firewall status queries from configuration changes.
- Identify the limitations of monitoring permanent configuration files.
- Understand how runtime and permanent firewall rules differ.
- Interpret sudo-related Wazuh alerts without assuming malicious intent.
- Differentiate an allowed network port from a successful network connection.
- Identify the additional evidence required to establish firewall tampering.

### Expected Learning Outcome

By completing this lab, the analyst should be able to examine Linux firewall settings, investigate administrative activity, interpret relevant Auditd and Wazuh events, and explain whether the available evidence supports a firewall configuration change.


## Lab 135 – Linux Firewall Configuration Change Investigation Scenario

### Scenario

A SOC analyst is reviewing administrative activity on a Rocky Linux server after identifying firewall-related commands in the system's security telemetry.

The server uses firewalld to control network access. Since firewall rules determine which services and ports are permitted, unexpected changes could expose additional services or weaken existing network restrictions.

The analyst is asked to examine the firewall configuration and determine whether any administrative activity resulted in an actual change to the system's security policy.

### Investigation Context

The investigation begins with a review of firewalld service status, active zones, permitted services, and allowed ports.

The analyst collects both runtime and permanent firewall configurations to establish the system's existing state.

Auditd monitoring is considered for permanent firewall configuration files, while system logs and sudo records provide additional context about administrative activity.

During the investigation, Wazuh records a root-level command:

```text
/bin/firewall-cmd --query-port=8080/tcp
```

The event is classified as successful sudo execution to root.

This observation raises an investigation question: was the administrator only checking the firewall state, or was there additional activity that changed the firewall configuration?

### Analyst Tasks

The analyst must:

- Identify the current firewall service status.
- Examine default and active zones.
- Record runtime and permanent firewall rules.
- Identify relevant firewall configuration files.
- Establish a configuration baseline.
- Review the Wazuh sudo event and its raw log.
- Determine what the recorded firewall command actually does.
- Examine Auditd and journal records for additional evidence.
- Identify whether any firewall configuration modification is supported by the evidence.
- Preserve the original firewall state during cleanup.

### Investigation Challenge

The analyst must distinguish administrative activity from an actual security configuration change.

A command that queries whether a port is allowed does not add or remove a firewall rule. Similarly, a sudo event does not establish malicious privilege escalation simply because the command was executed with root privileges.

The analyst must also consider the difference between runtime and permanent firewall configuration. Monitoring configuration files alone may not capture every runtime operation.

### Expected Outcome

The analyst should produce an investigation record describing the firewall state, the observed administrative command, relevant audit and Wazuh evidence, and any remaining evidence gaps.

The final conclusion must clearly distinguish confirmed sudo activity from unverified firewall modification, network exposure, or malicious intent.

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

