# Cloud-Based SOC & Detection Engineering Lab (Microsoft Sentinel)

## Overview

This project demonstrates the deployment of a cloud-native Security Operations Center (SOC) using Microsoft Sentinel, Azure Monitor Agent (AMA), and a Windows Server endpoint. The objective was to simulate common attack techniques, engineer custom detections, tune alerts to reduce analyst fatigue, and generate high-confidence incidents through event correlation.

Unlike many introductory SIEM projects that generate incidents for every detection, this lab focuses on alert prioritization and SOC-driven decision making.

---

## Skills Demonstrated

- Microsoft Sentinel Deployment
- Azure Monitor Agent (AMA)
- Azure Log Analytics
- Detection Engineering
- Kusto Query Language (KQL)
- Alert Tuning
- Incident Response
- Security Event Analysis
- MITRE ATT&CK Mapping
- Cloud Security Monitoring

---

## Lab Architecture

### Environment Components

- Windows Server VM
- Azure Monitor Agent (AMA)
- Data Collection Rule (DCR)
- Log Analytics Workspace
- Microsoft Sentinel

**[INSERT SCREENSHOT OF AZURE RESOURCE GROUP]**

**[INSERT SCREENSHOT OF WINDOWS SERVER VM]**

### Architecture Diagram

```text
Windows Server VM
        ↓
Azure Monitor Agent
        ↓
Log Analytics Workspace
        ↓
Microsoft Sentinel
        ↓
Analytics Rules
        ↓
Alerts / Incidents
```

**[INSERT ARCHITECTURE DIAGRAM]**

---

## Telemetry Collection

The Azure Monitor Agent was deployed to collect Windows Security Events and forward them into Microsoft Sentinel through a Log Analytics Workspace.

### Collected Event IDs

| Event ID | Description |
|-----------|-------------|
| 4624 | Successful Logon |
| 4625 | Failed Logon |
| 4672 | Special Privileges Assigned |
| 4720 | User Account Created |

**[INSERT SCREENSHOT OF AMA EXTENSION]**

**[INSERT SCREENSHOT OF SECURITY LOGS ARRIVING IN LOG ANALYTICS]**

---

# Detection Engineering

A key objective of this project was demonstrating how SOC analysts balance visibility and alert fatigue.

Not every detection should generate an incident.

Some detections provide context and hunting capability, while others represent higher-confidence indicators that warrant analyst investigation.

---

## Detection: Failed Authentication Burst

### Purpose

Detect repeated failed authentication attempts that may indicate password guessing or password spraying activity.

### Event ID

| Event ID | Description |
|-----------|-------------|
| 4625 | Failed Logon |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1110 | Brute Force |

### Severity

Low

### Incident Created?

No

### Why?

Failed logins are common in enterprise environments and often result from user mistakes, expired credentials, or service account issues.

Generating incidents for every authentication failure would overwhelm analysts.

**[INSERT SCREENSHOT OF EVENT ID 4625 RESULTS]**

---

## Detection: Successful Authentication Activity

### Purpose

Provide visibility into successful authentication activity.

### Event ID

| Event ID | Description |
|-----------|-------------|
| 4624 | Successful Logon |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1078 | Valid Accounts |

### Severity

Informational

### Incident Created?

No

### Why?

Successful logons occur constantly and provide valuable context during investigations but do not independently indicate malicious activity.

---

## Detection: Privileged Logon Detected

### Purpose

Monitor administrative account activity.

### Event ID

| Event ID | Description |
|-----------|-------------|
| 4672 | Special Privileges Assigned |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1078 | Valid Accounts |

### Severity

Informational

### Incident Created?

No

### Why?

Privileged logons occur frequently in enterprise environments and are most valuable when correlated with other suspicious events.

**[INSERT SCREENSHOT OF EVENT ID 4672 RESULTS]**

---

## Detection: Unauthorized Local Account Persistence Detection

### Purpose

Detect local account creation events that may indicate persistence activity.

### Event ID

| Event ID | Description |
|-----------|-------------|
| 4720 | User Account Created |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1136 | Create Account |

### Severity

Medium

### Incident Created?

Yes

### Why?

Account creation is a known persistence mechanism used by attackers to maintain access after initial compromise.

While account creation may be legitimate, it warrants analyst review.

**[INSERT SCREENSHOT OF EVENT ID 4720 RESULTS]**

**[INSERT SCREENSHOT OF GENERATED INCIDENT]**

---

## Detection: Potential Password Spray Success

### Purpose

Detect successful authentication following multiple failed authentication attempts.

### Event IDs

| Event ID | Description |
|-----------|-------------|
| 4625 | Failed Logon |
| 4624 | Successful Logon |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1110.003 | Password Spraying |
| T1078 | Valid Accounts |

### Severity

Medium

### Incident Created?

Yes

### Why?

Repeated authentication failures followed by successful access may indicate compromised credentials.

This provides significantly higher confidence than failed logins alone.

**[INSERT SCREENSHOT OF PASSWORD SPRAY INCIDENT]**

---

## Detection: Potential Account Compromise and Persistence

### Purpose

Correlate authentication failures, successful authentication, and account creation activity into a single high-confidence incident.

### Event IDs

| Event ID | Description |
|-----------|-------------|
| 4625 | Failed Logon |
| 4624 | Successful Logon |
| 4720 | User Account Created |

### MITRE ATT&CK

| Technique | Description |
|------------|------------|
| T1110 | Brute Force |
| T1110.003 | Password Spraying |
| T1078 | Valid Accounts |
| T1136 | Create Account |

### Severity

High

### Incident Created?

Yes

### Why?

This rule correlates multiple attack stages into a single incident, significantly increasing confidence that malicious activity occurred.

### Attack Chain

```text
Failed Authentication Attempts
            ↓
Successful Authentication
            ↓
Local Account Creation
            ↓
Persistence Established
```

**[INSERT SCREENSHOT OF CORRELATION QUERY RESULTS]**

**[INSERT SCREENSHOT OF HIGH SEVERITY INCIDENT]**

---

# Alert Tuning Strategy

One of the primary goals of this project was demonstrating alert tuning and analyst prioritization.

### Detections Only

- Failed Authentication Burst
- Successful Authentication Activity
- Privileged Logon Detected

These detections provide visibility and investigative context but are not independently strong indicators of malicious activity.

### Incident-Generating Detections

- Unauthorized Local Account Persistence Detection
- Potential Password Spray Success
- Potential Account Compromise and Persistence

These detections provide higher confidence indicators that warrant analyst investigation.

This approach reduces alert fatigue while preserving visibility into important security events.

---

# Incident Investigation

## Password Spray Success

### Findings

- Multiple failed logon attempts observed
- Successful authentication followed
- Activity aligned with password spraying behavior

### Recommendation

- Investigate authentication source
- Review account activity
- Enforce MFA

---

## Potential Account Compromise and Persistence

### Findings

- Failed logons detected
- Successful authentication observed
- Local account creation detected
- Multiple ATT&CK techniques observed

### Recommendation

- Validate account legitimacy
- Review authentication activity
- Remove unauthorized accounts
- Investigate potential credential compromise

**[INSERT SCREENSHOT OF INCIDENT INVESTIGATION VIEW]**

---

# MITRE ATT&CK Mapping

| Detection | ATT&CK Technique |
|------------|------------|
| Failed Authentication Burst | T1110 |
| Successful Authentication Activity | T1078 |
| Privileged Logon Detected | T1078 |
| Unauthorized Local Account Persistence Detection | T1136 |
| Potential Password Spray Success | T1110.003, T1078 |
| Potential Account Compromise and Persistence | T1110, T1110.003, T1078, T1136 |

---

# Lessons Learned

This project demonstrated how Microsoft Sentinel transforms endpoint telemetry into actionable security incidents.

The most important lesson learned was that effective SOC operations require alert tuning. Not every detection should become an incident. High-confidence detections should be escalated for investigation, while lower-confidence detections provide context, visibility, and hunting opportunities.

By correlating multiple security events into a single incident, analysts can reduce noise, improve detection confidence, and prioritize meaningful threats more effectively.