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
