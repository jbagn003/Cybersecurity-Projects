# Enterprise Identity Governance & Google Workspace Security Lab

## Overview

This project simulates the design and implementation of an enterprise Identity & Access Management (IAM) program for a fictional healthcare organization using Google Workspace.

The objective was to implement security controls commonly owned by Security Analysts and IAM Administrators, including user lifecycle management, RBAC, least privilege, Google Workspace administration, OAuth governance, email security, and access reviews.

The lab was built to mirror responsibilities commonly found in enterprise healthcare environments requiring HIPAA-aligned identity governance and cloud security administration.

---

# Environment

## Organization

**Bagnis Labs**

Industry:
Healthcare Technology

Identity Platform:
Google Workspace Business Standard

Domain:
bagnislabs.com

Users:
- Executive
- IT Administrator
- HR
- Clinical Staff
- Reception
- Finance
- Test Users

---

# Objectives

- Build a secure Google Workspace tenant
- Implement enterprise IAM concepts
- Enforce least privilege
- Design role-based access control (RBAC)
- Simulate Joiner-Mover-Leaver lifecycle
- Perform quarterly access reviews
- Configure Google Workspace security controls
- Review OAuth activity
- Configure SPF, DKIM, and DMARC
- Document security findings and remediation

---

# Identity Architecture

Designed an organizational structure using Organizational Units, Google Groups, and Shared Drives to separate departments and enforce role-based permissions.

Departments included:

- Executive
- IT
- Clinical
- HR
- Finance
- Reception

### Evidence

![sasdasd](vscode-userdata:/User/caches/remote-clipboard/09e39482-4655-4816-a275-e3824fc4c0cd/d6c461fd-3ea9-4cf5-85d3-05ddcb15aea5/joiner%20creation.PNG)

`/Screenshots/shared drive creation.png`

`/Screenshots/All shared drives.png`

---

# Role-Based Access Control (RBAC)

Implemented RBAC using Google Groups instead of assigning permissions directly to users.

Department groups were granted access to Shared Drives according to business function.

Examples:

Clinical
- Clinical Records
- No Finance

HR
- HR Documents
- No Clinical

Finance
- Finance Records
- Read Company Policies

Reception
- Company Policies Only

IT
- Administrative Access

Least privilege was validated by testing cross-department access restrictions.

### Evidence

`/Screenshots/Finance drive members.png`

`/Screenshots/Request Access shared drive for nurse at finance docs.png`

`/Screenshots/doc in finance group misconfig.png`

---

# Identity Lifecycle Management (Joiner / Mover / Leaver)

Simulated enterprise onboarding, role transitions, and offboarding.

## Joiner

Provisioned a new employee by:

- Creating account
- Assigning Organizational Unit
- Assigning Google Groups
- Applying password policy
- Requiring MFA

### Evidence

`/Screenshots/joiner creation.png`

`/Screenshots/add joiner to group.png`

`/Screenshots/joiner OU assignment.png`

---

## Mover

Simulated an internal promotion by updating group membership and department permissions while maintaining least privilege.

(Changes documented within repository.)

---

## Leaver

Performed account suspension and removal from the identity environment.

### Evidence

`/Screenshots/JOINER SUSPENDED.png`

---

# Identity Governance

Performed a simulated quarterly access review.

Introduced intentional privilege misconfigurations including:

- Reception assigned to HR
- Doctor assigned Finance permissions
- HR assigned Super Admin

Performed review and documented remediation.

### Findings

Unauthorized HR Membership

Excessive Administrative Privileges

Improper Finance Access

### Evidence

`/Screenshots/reception in HR misconfig.png`

`/Screenshots/doc in finance group misconfig.png`

`/Screenshots/Super admin misconfig.png`

---

# Authentication Controls

Implemented enterprise authentication controls.

Configured:

- Multi-Factor Authentication
- Password Policies
- Administrative Roles

### Evidence

`/Screenshots/MFA enablement.png`

`/Screenshots/Password management.png`

---

# Google Drive Security

Implemented department-specific Shared Drives.

Restricted sharing using organizational policies.

Validated access restrictions by testing user permissions.

Attempted external sharing and confirmed policy enforcement prevented data exposure.

### Evidence

`/Screenshots/cant share (nurse to external).png`

`/Screenshots/EXTERNAL DRIVE SHARING.png`

---

# OAuth Governance

Simulated third-party application authorization using non-administrative users.

Installed:

- Adobe Acrobat
- SmallPDF

Reviewed OAuth authorization events through Google Workspace Audit Logs.

Configured Google Workspace to require administrator approval before users could authorize future third-party applications.

Validated policy enforcement by confirming application access was denied after policy implementation.

### Evidence

`/Screenshots/OAuth SmallPDF add-on install permissions request for nurse.png`

`/Screenshots/OAuth Log events showing small pdf and adobe acrobat granted by nurse.png`

`/Screenshots/configuring access for third party integration (squarespace login).png`

`/Screenshots/Access blocked for small pdf for nurse after change.png`

---

# Gmail Security

Reviewed Gmail security settings and verified protections including:

- External sender warnings
- Anti-spoofing protections
- Safe browsing
- Spam protections

Validated external email identification using a simulated phishing email.

### Evidence

`/Screenshots/gmail safety settings.png`

`/Screenshots/external email phishing attempt marked as external.png`

---

# Email Authentication

Configured enterprise email authentication using DNS records for the custom domain.

Implemented:

- SPF
- DKIM
- DMARC

Validated successful authentication by reviewing Gmail message headers.

Results:

- SPF = PASS
- DKIM = PASS
- DMARC = PASS

### Evidence

`/Screenshots/showing DNS of DMARC added.png`

`/Screenshots/Squarespace DNS management showing v=SPF and DKIM.png`

`/Screenshots/Nurse to external SPF PASS.png`

`/Screenshots/nurse to external showing spf pass and dmarc pass.png`

---

# Skills Demonstrated

Identity & Access Management (IAM)

Google Workspace Administration

Role-Based Access Control (RBAC)

Least Privilege

Google Groups

Organizational Units

User Provisioning

Joiner-Mover-Leaver Lifecycle

Access Governance

Quarterly Access Reviews

OAuth Governance

Google Workspace Audit Logs

Google Drive Security

Multi-Factor Authentication

Password Policy Administration

Gmail Security

SPF

DKIM

DMARC

DNS Administration

Security Documentation

---

# Key Takeaways

This project demonstrates the implementation of an enterprise identity governance program using Google Workspace.

Rather than focusing solely on administration, the project emphasizes security operations including identity lifecycle management, least privilege, access governance, OAuth review, audit logging, and email authentication.

The controls implemented closely align with responsibilities commonly assigned to Security Analysts responsible for IAM and SaaS security administration in cloud-first healthcare environments.