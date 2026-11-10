# Operation-WeakLink-AD-Credential-Compromise
Active Directory credential compromise lab focused on password spraying, SMB access, Wazuh detection, and logging gaps.

## Project Overview

I built and tested a small enterprise-style Active Directory environment to evaluate how far an attacker could progress using valid credentials and how much of that activity was actually detected.

The environment included:

- Windows Server domain controller
- Domain-joined Windows endpoint
- Kali Linux attacker system
- Wazuh SIEM
- Active Directory domain services
- Centralized authentication and logging

The attack simulation progressed from reconnaissance through credential-based access and SMB share interaction.

## Environment

### AD01
Windows Server domain controller for the `corp.local` domain.

### AD02
Domain-joined Windows workstation used to simulate user activity and authentication events.

### Kali Linux
Used for reconnaissance, enumeration, password spraying, authentication testing, and SMB interaction.

### Wazuh
Used for centralized logging, event analysis, and detection validation.

## Attack Path

1. Reconnaissance
2. User enumeration
3. Credential access
4. Initial access
5. SMB enumeration
6. SMB share access
7. Detection and analysis

## Detection Analysis

Wazuh collected important Windows authentication activity, including:

- Event ID `4625` — failed authentication
- Event ID `4624` — successful authentication

Several detection gaps were identified:

- No threshold-based alerting for repeated failed logins
- No correlation between failed and successful authentication
- Missing SMB share auditing for Event ID `5140`

- ## Detection Engineering Recommendations

Based on the findings from Operation WeakLink, I identified several opportunities to improve authentication monitoring, event correlation, and SMB visibility within the Active Directory environment.

### 1. Password Spraying Detection

Implement threshold-based alerting for repeated Windows Event ID `4625` authentication failures across multiple user accounts.

Detection logic should consider:

- Multiple failed authentication attempts within a defined time window
- Repeated failures originating from a common source IP address
- Authentication attempts targeting multiple distinct accounts
- Patterns consistent with password spraying rather than individual password errors

### 2. Failed-to-Successful Authentication Correlation

Correlate Windows Event IDs `4625` and `4624` to identify successful authentication following repeated failures.

Prioritize investigations when:

- Multiple failed logins precede a successful authentication
- The activity originates from the same source IP or associated host
- The successful authentication involves an account targeted during the earlier failures

### 3. SMB Access Monitoring

Enable appropriate Windows auditing to improve visibility into SMB share access.

Relevant events include:

- `5140` — Network share object accessed
- `5145` — Network share object checked for access permissions

Review access to sensitive shares and investigate unusual access patterns, particularly following suspicious authentication activity.

### 4. Detection Validation

After implementing detection improvements, repeat the authorized lab simulations to verify whether Wazuh generates actionable alerts.

Validation should include:

- Confirming required Windows events are collected
- Testing detection thresholds and correlation logic
- Evaluating potential false positives
- Documenting alert results and remaining visibility gaps

**Status:** Proposed improvements based on observed lab findings. Detection rules and enhanced auditing require implementation and validation.

## MITRE ATT&CK Mapping

- `T1046` — Network Service Discovery
- `T1087` — Account Discovery
- `T1110` — Brute Force / Password Spraying
- `T1078` — Valid Accounts
- `T1021.002` — SMB / Windows Admin Shares

## Tools Used

- Wazuh
- Windows Server / Active Directory
- Windows 10
- Kali Linux
- Nmap
- Kerbrute
- CrackMapExec
- smbclient
- enum4linux-ng
- BloodHound

## Key Takeaway

This lab showed that logging alone is not enough.

Important attacker activity was visible in telemetry, but missing alerting, event correlation, and SMB auditing created gaps that allowed credential-based access to progress without generating actionable detection.
