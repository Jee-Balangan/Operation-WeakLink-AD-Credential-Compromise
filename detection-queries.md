# Operation WeakLink Detection Queries

This file documents the Windows security events and Wazuh queries used to review authentication activity and identify detection gaps during Operation WeakLink.

## Detection Objective

The goal of the defensive analysis was to determine whether credential-based attacker activity was:

- Logged
- Visible in Wazuh
- Correlated
- Alerted on
- Actionable for a security analyst

The lab showed that some attacker activity was successfully logged, but important detection and correlation gaps remained.

## Event ID 4625 — Failed Authentication

Repeated failed authentication attempts were generated during password spraying activity.

### Wazuh Query

```text
data.win.system.eventID:4625
```

### What This Event Showed

Event ID `4625` provided visibility into failed authentication attempts.

Multiple failed logins were recorded during the password spraying phase.

### Detection Gap

The events were present in the SIEM, but there was no threshold-based alerting configured.

This meant repeated failed authentication attempts could continue without generating an actionable alert.

## Event ID 4624 — Successful Authentication

A successful authentication event was recorded after the password spraying activity.

### Wazuh Query

```text
data.win.system.eventID:4624 AND data.win.eventdata.targetUserName:eren
```

### What This Event Showed

Event ID `4624` confirmed that the compromised account successfully authenticated to the domain controller.

The authentication originated from a non-domain system.

### Detection Gap

The successful authentication event was not correlated with the failed authentication events that occurred immediately before it.

This created a major detection gap because the sequence:

```text
Multiple failed logins → Successful login
```

is a strong indicator of possible credential compromise or password spraying.

## Event ID 4740 — Account Lockout

This query was used to check whether authentication failures resulted in an account lockout.

### Wazuh Query

```text
data.win.system.eventID:4740
```

### What This Event Was Used For

Event ID `4740` indicates that a Windows account has been locked out.

During testing, repeated authentication attempts were not stopped by effective account lockout controls.

This allowed password spraying activity to continue without interruption.

## Event ID 5140 — Network Share Access

Event ID `5140` is used to record access to Windows network shares.

### Expected Detection

SMB activity against shares such as:

- `NETLOGON`
- `SYSVOL`

should have generated share-access telemetry.

### Detection Gap

Event ID `5140` was not being generated because file-share auditing was not enabled.

This meant authenticated SMB share access could occur without corresponding visibility in the SIEM.

The absence of this telemetry prevented analysts from monitoring interaction with important domain resources.

## Authentication Correlation Gap

One of the most significant findings was that authentication events were logged individually but were not correlated.

Observed sequence:

```text
4625 → Multiple failed authentication attempts
4625 → Additional failed authentication attempts
4624 → Successful authentication
```

The SIEM did not combine these events into a credential-compromise alert.

In a production environment, this sequence should be treated as suspicious and investigated for:

- Password spraying
- Brute-force activity
- Compromised credentials
- Unauthorized account access

## Non-Domain Authentication Gap

The successful authentication originated from a system that was not joined to the Active Directory domain.

This behavior was visible in the logs but was not flagged as suspicious.

Authentication to critical domain resources from an unauthorized or non-domain system should receive additional scrutiny.

## SMB Visibility Gap

The attacker was able to authenticate over SMB and interact with accessible domain shares.

However, because Event ID `5140` auditing was not enabled, share access was not visible through the expected Windows security telemetry.

This created a blind spot after successful authentication.

## Detection Gaps Identified

The investigation identified the following gaps:

1. No threshold-based alerting for repeated Event ID `4625` failures
2. No correlation between failed `4625` events and successful `4624` authentication
3. No alerting for authentication originating from a non-domain system
4. Missing Event ID `5140` due to disabled SMB share auditing
5. Weak account lockout controls allowed repeated password attempts
6. Logged events were visible but were not consistently converted into actionable detections

## Key Detection Lesson

Operation WeakLink demonstrated that log collection alone does not equal detection.

The environment recorded important authentication events, but without correlation rules, threshold-based alerting, and complete audit configuration, attacker activity could progress without generating an actionable security alert.
