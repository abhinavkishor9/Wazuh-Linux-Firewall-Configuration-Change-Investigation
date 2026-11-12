# Lab 135 – Troubleshooting Notes

## 1. Runtime and Permanent Configuration Differences

Firewalld maintains separate runtime and permanent configurations.

Runtime configuration:

```bash
sudo firewall-cmd --list-all
```

Permanent configuration:

```bash
sudo firewall-cmd --permanent --list-all
```

A rule may be present in one configuration and absent from the other.

Always compare both states before concluding that a firewall rule was added or removed.

## 2. Understanding firewall port queries

Command:

```bash
sudo firewall-cmd --query-port=8080/tcp
```

Possible results:

- `yes` – The queried port is allowed in the selected runtime zone.
- `no` – The queried port is not allowed in the selected runtime zone.

For permanent configuration:

```bash
sudo firewall-cmd --permanent --query-port=8080/tcp
```

The command is read-only. It checks firewall state and does not add or remove rules.

A sudo event containing this command should not be classified as proof of firewall modification.

## 3. Understanding the Wazuh sudo event

Observed rule:

- Rule ID: `5402`
- Description: `Successful sudo to ROOT executed.`
- Level: `3`
- Decoder: `sudo`

The event records the command:

```text
/bin/firewall-cmd --query-port=8080/tcp
```

The event establishes sudo command execution but does not include the firewall query result.

To determine whether the port was allowed, use the command output or corroborating firewall state.

## 4. Understanding the MITRE mapping

The event contains:

```text
T1548.003
Sudo and Sudo Caching
```

This is the sudo-related technique mapping assigned to the Wazuh rule.

It does not independently prove:

- Malicious privilege escalation.
- Unauthorized administrative access.
- Firewall tampering.
- Defense evasion.

Interpret the mapping in the context of the raw event and the activity being investigated.

## 5. Auditd Directory Watch Limitations

Example rule:

```bash
sudo auditctl -w /etc/firewalld/ -p wa -k lab135_firewall_changes
```

This monitors write and attribute-change operations under the permanent firewall configuration directory.

Runtime firewall changes may not modify files in this directory.

Therefore, an empty result from the directory watch does not establish that no runtime firewall operation occurred.

## 6. Investigating Auditd Records

Search:

```bash
sudo ausearch -k lab135_firewall_changes -i
```

Review:

- `SYSCALL`
- `PATH`
- `PROCTITLE`
- `CWD`
- `exe`
- `comm`
- `uid`
- `auid`
- `success`

Correlate records belonging to the same audit event using their event identifiers.

## 7. Wazuh Event Collection Versus Alert Generation

Wazuh can receive a sudo or Auditd event without generating a dedicated firewall-tampering alert.

When investigating an event:

1. Check its raw log.
2. Identify the decoder.
3. Review the rule description.
4. Examine the rule ID and level.
5. Determine what activity the event actually establishes.

Do not interpret every firewall-related command as a firewall configuration change.

## 8. Safe Firewall Cleanup

Before removing a test rule, verify whether it existed before the lab.

If TCP port 8080 was absent before the test and was added during the exercise, remove the temporary runtime and permanent rules separately.

Runtime removal:

```bash
sudo firewall-cmd --remove-port=8080/tcp
```

Permanent removal:

```bash
sudo firewall-cmd --permanent --remove-port=8080/tcp
```

Verify both states:

```bash
sudo firewall-cmd --query-port=8080/tcp
sudo firewall-cmd --permanent --query-port=8080/tcp
```

Both should return `no` if the port was originally absent and the test changes were successfully removed.

Do not remove a pre-existing firewall rule.

## 9. Auditd Rule Cleanup

Remove only the Lab 135 monitoring rule:

```bash
sudo auditctl -W /etc/firewalld/ -p wa -k lab135_firewall_changes
```

Verify:

```bash
sudo auditctl -l | grep lab135_firewall_changes
```

Do not use `auditctl -D`, because it removes all loaded Auditd rules, including rules from other labs.
