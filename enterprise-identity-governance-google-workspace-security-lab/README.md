# Enterprise Identity Governance & Google Workspace Security Lab

## Overview

This project simulates the design and implementation of an enterprise Identity & Access Management (IAM) program for a fictional healthcare organization using Google Workspace.

The objective was to implement security controls commonly owned by Security Analysts and IAM Administrators, including user lifecycle management, RBAC, least privilege, Google Workspace administration, OAuth governance, email security, and access reviews.

The lab was built to mirror responsibilities commonly found in enterprise healthcare environments requiring HIPAA-aligned identity governance and cloud security administration.

---

# Environment

## Organization

**Bagnis Labs**

**Industry:** Healthcare Technology

**Identity Platform:** Google Workspace Business Standard

**Domain:** bagnislabs.com

### Users

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
- Design Role-Based Access Control (RBAC)
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

## Evidence

![All Groups](Screenshots/All%20Groups.PNG)

![Shared Drive Creation](Screenshots/shared%20drive%20creation.PNG)

![All Shared Drives](Screenshots/All%20shared%20drives.PNG)

---

# Role-Based Access Control (RBAC)

Implemented RBAC using Google Groups instead of assigning permissions directly to users.

Department groups were granted access to Shared Drives according to business function.

Examples:

### Clinical

- Clinical Records
- No Finance Access

### HR

- HR Documents
- No Clinical Access

### Finance

- Finance Records
- Read Company Policies

### Reception

- Company Policies Only

### IT

- Administrative Access

Least privilege was validated by testing cross-department access restrictions.

## Evidence

![Finance Drive Members](Screenshots/Finance%20drive%20members.PNG)

![Finance Access Request](Screenshots/Request%20Access%20shared%20drive%20for%20nurse%20at%20finance%20docs.PNG)

![Finance Permission Misconfiguration](Screenshots/doc%20in%20finance%20group%20misconfig.PNG)

---

# Identity Lifecycle Management (Joiner / Mover / Leaver)

Simulated enterprise onboarding, role transitions, and offboarding.

## Joiner

Provisioned a new employee by:

- Creating the user account
- Assigning the Organizational Unit
- Assigning Google Groups
- Applying password policies
- Requiring Multi-Factor Authentication (MFA)

### Evidence

![Joiner Creation](Screenshots/joiner%20creation.PNG)

![Add Joiner to Group](Screenshots/add%20joiner%20to%20group.PNG)

![Joiner OU Assignment](Screenshots/joiner%20OU%20assignment.PNG)

---

## Mover

Simulated an internal promotion by updating group membership and department permissions while maintaining least privilege.

Changes were validated by confirming previous department permissions were removed and new permissions were successfully applied.

---

## Leaver

Performed account suspension and removal from the identity environment.

### Evidence

![Joiner Suspended](Screenshots/JOINER%20SUSPENDED.PNG)

---

# Identity Governance

Performed a simulated quarterly access review.

Introduced intentional privilege misconfigurations including:

- Reception assigned to HR
- Doctor assigned Finance permissions
- HR assigned Super Admin

Performed an access review and documented remediation.

## Findings

- Unauthorized HR Membership
- Excessive Administrative Privileges
- Improper Finance Access

## Evidence

![Reception in HR Misconfiguration](Screenshots/reception%20in%20HR%20misconfig.PNG)

![Finance Group Misconfiguration](Screenshots/doc%20in%20finance%20group%20misconfig.PNG)

![Super Admin Misconfiguration](Screenshots/Super%20admin%20misconfig.PNG)

---

# Authentication Controls

Implemented enterprise authentication controls.

Configured:

- Multi-Factor Authentication (MFA)
- Password Policies
- Administrative Roles

## Evidence

![MFA Enablement](Screenshots/MFA%20enablement.PNG)

![Password Management](Screenshots/Password%20management.PNG)

---

# Google Drive Security

Implemented department-specific Shared Drives.

Restricted sharing using organizational policies.

Validated access restrictions by testing user permissions.

Attempted external sharing and confirmed policy enforcement prevented unauthorized data exposure.

## Evidence

![External Sharing Blocked](Screenshots/cant%20share(nurse%20to%20external).PNG)

![External Drive Sharing Policy](Screenshots/EXTERNAL%20DRIVE%20SHARING.PNG)

---

# OAuth Governance

Simulated third-party application authorization using non-administrative users.

Installed:

- Adobe Acrobat
- SmallPDF

Reviewed OAuth authorization events through Google Workspace Audit Logs.

Configured Google Workspace to require administrator approval before users could authorize future third-party applications.

Validated policy enforcement by confirming application access was denied after policy implementation.

## Evidence

![OAuth Permission Request](Screenshots/OAuth%20SmallPDF%20add-on%20install%20permissions%20request%20for%20nurse.PNG)

![OAuth Audit Logs](Screenshots/OAuth%20Log%20events%20showing%20small%20pdf%20and%20adobe%20acrobat%20granted%20by%20nurse.PNG)

![Third-Party Integration Policy](Screenshots/configuring%20access%20for%20third%20party%20integration%20(squarespace%20login).PNG)

![Access Blocked After Policy Change](Screenshots/Access%20blocked%20for%20small%20pdf%20for%20nurse%20after%20change.PNG)

---

# Gmail Security

Reviewed Gmail security settings and verified protections including:

- External sender warnings
- Anti-spoofing protections
- Safe Browsing
- Spam protections

Validated external email identification using a simulated phishing email.

## Evidence

![Gmail Safety Settings](Screenshots/gmail%20safety%20settings.PNG)

![External Email Warning](Screenshots/external%20email%20phishing%20attempt%20marked%20as%20external.PNG)

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

## Evidence

![DMARC DNS Configuration](Screenshots/showing%20DNS%20of%20DMARC%20added.PNG)

![SPF & DKIM DNS Records](Screenshots/Squarespace%20DNS%20management%20showing%20v=SPF%20and%20DKIM.PNG)

![SPF PASS](Screenshots/Nurse%20to%20external%20SPF%20PASS.PNG)

![SPF, DKIM & DMARC PASS](Screenshots/nurse%20to%20external%20showing%20spf%20pass%20and%20dmarc%20pass.PNG)

---

# Skills Demonstrated

- Identity & Access Management (IAM)
- Google Workspace Administration
- Role-Based Access Control (RBAC)
- Least Privilege
- Google Groups
- Organizational Units
- User Provisioning
- Joiner-Mover-Leaver Lifecycle
- Access Governance
- Quarterly Access Reviews
- OAuth Governance
- Google Workspace Audit Logs
- Google Drive Security
- Multi-Factor Authentication
- Password Policy Administration
- Gmail Security
- SPF
- DKIM
- DMARC
- DNS Administration
- Security Documentation

---

# Key Takeaways

This project demonstrates the implementation of an enterprise identity governance program using Google Workspace.

Rather than focusing solely on administration, the project emphasizes security operations including identity lifecycle management, least privilege, access governance, OAuth review, audit logging, and email authentication.

The controls implemented closely align with responsibilities commonly assigned to Security Analysts responsible for IAM and SaaS security administration in cloud-first healthcare environments.