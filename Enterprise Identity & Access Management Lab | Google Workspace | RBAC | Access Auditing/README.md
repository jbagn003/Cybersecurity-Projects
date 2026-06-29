# Enterprise Identity & Access Management Lab | Google Workspace | RBAC | Access Auditing

**A simulated IAM program for a healthcare organization, built to mirror the day to day work of a Security Analyst owning access governance, identity lifecycle, and email security in a HIPAA-aligned environment.**

---

## Overview

- [Role Based Access Control](#role-based-access-control-rbac--least-privilege)
- [Identity Lifecycle Management](#identity-lifecycle-management-joiner--mover--leaver)
- [Access Governance: The Quarterly Access Review](#access-governance-the-quarterly-access-review)
- [Authentication Controls](#authentication-controls)
- [Google Drive Security](#google-drive-security)
- [OAuth Governance](#oauth-governance)
- [Gmail Security](#gmail-security)
- [Email Authentication](#email-authentication-spf--dkim--dmarc)
- [Key Performance Indicators](#key-performance-indicators)
- [HIPAA Alignment](#how-this-maps-to-hipaa)
- [Lessons Learned](#lessons-learned--what-id-do-differently-at-enterprise-scale)
- [Skills Demonstrated](#skills-demonstrated)

---

## Why I Built This

Access misconfiguration is one of the most common root causes behind real healthcare data breaches. Over-permissioned accounts, stale group memberships, and unreviewed admin roles don't show up in a vulnerability scan. They show up in an audit, usually after something has already gone wrong. That's the part of this project I wanted to spend the most time on.

**Bagnis Labs** is a fictional healthcare technology company inside a real Google Workspace tenant where I walk through a complete identity governance lifecycle: design the access model, onboard and offboard users, simulate the kind of privilege creep that happens in every real org, and catch it in a quarterly access review. That access review is the centerpiece of this project. I also configured the email authentication stack (SPF/DKIM/DMARC) and reviewed OAuth app governance, since both come up constantly in real SOC and IAM analyst work.

---

## Environment

| | |
|---|---|
| **Organization** | Bagnis Labs (fictional) |
| **Industry** | Healthcare Technology |
| **Identity Platform** | Google Workspace Business Standard |
| **Domain** | bagnislabs.com |
| **Departments** | Executive, IT, Clinical, HR, Finance, Reception |

I structured the org with six departments, each represented by a dedicated Organizational Unit, a Google Group, and a corresponding Shared Drive, so that every access decision could be made (and reviewed) at the group level rather than the individual level. This is the same pattern most enterprises use because it scales: you manage 6 groups, not 30 individual users.

<img src="Screenshots/All%20Groups.PNG" alt="All Groups" width="400">

*Department-level Google Groups created to drive group-based RBAC rather than per-user permissions. Each group maps 1:1 to an OU and a Shared Drive, which is what makes the access review later on actually auditable. I can check group membership instead of tracing individual ACLs.*

<img src="Screenshots/All%20shared%20drives.PNG" alt="All Shared Drives" width="500">

*The resulting Shared Drive layout: one drive per department plus a shared "Company Policies" drive that's read-accessible org-wide.*

<img src="Screenshots/shared%20drive%20creation%20(users%20not%20allowed).PNG" alt="Shared Drive Creation" width="400">

*Restricted Shared Drive creation to admins only via Workspace settings, so end users can't spin up ungoverned drives outside the access model.*

---

## Role-Based Access Control (RBAC) & Least Privilege

The core design principle: **no permission is ever assigned to a person directly, only to a group, and a person only ever joins the group that matches their job function.** This is what makes least privilege enforceable instead of aspirational, and it's also what makes the access review further down possible at all.

I documented the intended access model as a formal baseline matrix before touching the admin console, so that every access decision afterward had something concrete to be checked against:

| Resource | Executive | HR | Clinical | Finance | Reception | IT |
|---|---|---|---|---|---|---|
| HR Shared Drive | R | RW | No | No | No | RW |
| Clinical Records | R | No | RW | No | No | RW |
| Finance Drive | R | No | No | RW | R | RW |
| IT Documentation | No | No | No | No | No | RW |

*(Full baseline with formatting and legend: [`Bagnis_Labs_Q2_2026_Access_Review.xlsx`](./Audit%20Report/Bagnis_Labs_Q2_2026_Access_Review.xlsx), "Access Baseline" tab)*

A few deliberate design choices worth calling out:
- **Clinical has zero access to Finance, and vice versa.** In a real healthcare org this isn't just tidy IAM, it's the kind of boundary HIPAA's access control requirements and general financial-data segregation expect to see.
- **IT has read/write everywhere.** That's intentional for an admin function, but it's also exactly the kind of broad access that should get the closest scrutiny in an access review, which is part of why I treated an IT-adjacent privilege escalation as the highest-severity finding later on.
- **Reception gets read-only on Finance, not no-access.** Front-desk billing questions are a real, common business need; the lab models giving the minimum access that supports that, not zero access "for safety."

I validated the model by testing cross-department access directly, logging in as a non-IT user and confirming a restricted Shared Drive correctly refused access rather than just assuming the group settings were doing their job. This baseline is also the exact thing the quarterly access review checks everything against later on.

<img src="Screenshots/Finance%20drive%20members.PNG" alt="Finance Drive Members" width="400">

*Confirms Finance Shared Drive membership matches the baseline. Only the Finance and IT groups have access to make changes, while other roles get read-only viewing access.*

<img src="Screenshots/Request%20Access%20shared%20drive%20for%20nurse%20at%20finance%20docs.PNG" alt="Finance Access Request" width="300">

*A Clinical-department test user (nurse) attempting to access the Finance Drive and being correctly prompted to request access rather than getting it automatically. This validates that least privilege is enforced at the platform level, not just on paper.*

---

## Identity Lifecycle Management (Joiner / Mover / Leaver)

Access control design is only half the job. The other half is making sure people actually end up with the right access as they join, move within, and eventually leave the organization. I simulated all three stages, partly because lifecycle gaps are exactly where the misconfigurations in the access review below tend to come from.

### Joiner
Provisioned a new employee end-to-end: created the account, assigned it to the correct OU, added it to the relevant department group, and enforced the org's password policy and MFA requirement immediately at creation, not as a follow-up task.

<img src="Screenshots/joiner%20creation.PNG" alt="Joiner Creation" width="350">

*New user account created in the Workspace Admin Console with department and title fields set, which drive downstream OU placement.*

<img src="Screenshots/joiner%20OU%20assingment.PNG" alt="Joiner OU Assignment" width="300">

*Confirms the new account landed in the correct Organizational Unit. This matters because OU placement is what inherits department-wide policies (password rules, app access) automatically, rather than requiring manual policy assignment per user.*

<img src="Screenshots/add%20joiner%20to%20group.PNG" alt="Add Joiner to Group" width="300">

*Adding the new user to their department Google Group. This single action is what grants them their entire Shared Drive access profile, since permissions are never assigned individually.*

### Mover
When there is an internal promotion or transfer, I can change the user's group membership. First, I removed them from their old department group and added them to the new one, then explicitly verified the old access was actually gone, not just that the new access was present. This distinction matters: a lot of real-world access creep happens specifically because the "remove old access" half of a move gets skipped, which is exactly what one of the findings in the access review below turned out to be.

### Leaver
Performed a full account suspension as part of offboarding, immediately cutting off authentication rather than relying on group removal alone (suspension blocks login entirely; just removing groups can still leave residual access depending on caching and sync timing).

<img src="Screenshots/JOINER%20SUSPENDED.PNG" alt="Joiner Suspended" width="600">

*Account suspended in the Admin Console. Status changes to suspended and the user can no longer authenticate, which I confirmed by attempting a login with the suspended credentials.*

---

## Access Governance: The Quarterly Access Review

**This is the core of the entire project.** Designing a clean RBAC model and provisioning users correctly matters, but it doesn't mean much if nobody ever checks whether reality still matches the plan six months later. Almost every real access-related incident I've read about didn't come from a broken control, it came from a control that used to be correct and quietly drifted. This section is me building and running the process that's supposed to catch that.

**The setup:** I intentionally introduced three privilege misconfigurations into the environment, the kind that happen in real organizations through normal operational sloppiness: someone added to the wrong group during a rushed onboarding, an admin role never walked back after a temporary need, a transfer that only added new access without removing old. Then I ran a structured access review against the baseline to find them, the same kind of exercise a security analyst would run quarterly across critical systems in a real environment.

**The findings:**

| Finding | Department | Issue | Risk |
|---|---|---|---|
| Unauthorized HR membership | Reception | Reception user added to HR Google Group, gaining RW access to HR Shared Drive with zero business justification | High |
| Excessive Finance access | Clinical | Clinical staff member added to Finance Google Group, gaining RW access to financial records | High |
| Super Admin over-assignment | HR | HR account granted Super Admin role in Workspace, giving full administrative control over the entire tenant | Critical |

I rated the Super Admin finding Critical rather than just High because of blast radius. A compromised or simply careless Super Admin account in a non-IT department isn't just an over-permissioned drive, it's a credential that can alter security policy itself, including MFA enforcement and other admins' access. That's a materially different risk category from "can see records they shouldn't."

All three findings were remediated in the same review cycle: group memberships were corrected, the Super Admin role was downgraded to standard user, and I reviewed the Workspace admin audit log to confirm no actions were taken during the exposure window.

I documented this as a formal access review artifact rather than just narrating it in prose: **[`Bagnis_Labs_Q2_2026_Access_Review.xlsx`](./Audit%20Report/Bagnis_Labs_Q2_2026_Access_Review.xlsx)**, with three tabs:
- **Summary**: KPIs (entitlements reviewed, findings, remediation rate, MFA adoption) and an executive summary
- **Access Baseline**: the approved least-privilege policy matrix
- **Audit Findings**: finding-by-finding detail, expected vs. observed access, variance, risk rating, remediation action, status, and remediation date

<img src="Audit%20Report/summary%20tab%20excel.PNG" alt="Access Review Summary Tab" width="700">

*Summary tab: KPIs and executive summary, the page I'd lead with if asked to walk through the review.*

<img src="Audit%20Report/Access%20Baseline%20excel.PNG" alt="Access Baseline Tab" width="700">

*Access Baseline tab: the approved least-privilege policy matrix, color-coded by access level (RW / R / No), used as the control every observed permission gets checked against.*

<img src="Audit%20Report/Audit%20Findings%20Excel.PNG" alt="Audit Findings Tab" width="1000">

*Audit Findings tab: the three misconfigurations logged as formal findings with expected vs. observed access, risk rating, remediation action, and closure status.*

<img src="Screenshots/reception%20in%20HR%20misconfig.PNG" alt="Reception in HR Misconfiguration" width="600">

*Google Admin Console showing the Reception department user as an active member of the HR Google Group, the unauthorized membership identified as Finding AR-2026Q2-01.*

**Likely Root Cause:** The user was likely assigned to the incorrect Google Group during onboarding or a role change without validation against the approved RBAC baseline.

**Potential Impact:** The user gained unnecessary read and write access to HR resources containing sensitive employee information, violating least privilege and increasing the risk of unauthorized data exposure.

---

<img src="Screenshots/doc%20in%20finance%20group%20misconfig.PNG" alt="Finance Group Misconfiguration" width="600">

*Clinical department user shown as a member of the Finance Google Group, violating the documented access baseline and identified as Finding AR-2026Q2-02.*

**Likely Root Cause:** Excess permissions were likely introduced during a department transfer or manual group assignment without removing previous access.

**Potential Impact:** The user received unauthorized access to financial records, increasing the risk of accidental disclosure or misuse of confidential business information.

---

<img src="Screenshots/Super%20admin%20misconfig.PNG" alt="Super Admin Misconfiguration" width="600">

*Admin role assignment page showing an HR department account holding the Super Admin role, identified as Finding AR-2026Q2-03 and rated Critical.*

**Likely Root Cause:** Administrative privileges were granted temporarily or assigned incorrectly and were never reviewed or removed through a formal access governance process.

**Potential Impact:** A compromised or misused Super Admin account could bypass security controls, modify tenant-wide security settings, create privileged accounts, and compromise the entire Google Workspace environment.

**What I'd do differently at enterprise scale:** A quarterly manual review catches drift after the fact, but it doesn't prevent it between cycles. In production, I'd push for an automated weekly or daily group-membership export via the Admin SDK / Reports API, diffed against the approved baseline, so deviations get flagged in days instead of up to a full quarter.

---

## Authentication Controls

Configured baseline authentication hardening across the tenant: MFA enforcement, password complexity policy, and admin role restrictions.

<img src="Screenshots/MFA%20enablement.PNG" alt="MFA Enablement" width="600">

*Multi-Factor Authentication enforcement enabled at the organizational level in the Admin Console. Applies across all departments, with no exceptions carved out.*

<img src="Screenshots/Password%20management.PNG" alt="Password Management" width="400">

*Password policy configuration showing complexity and length requirements enforced tenant-wide.*

---

## Google Drive Security

Beyond department-level Shared Drive segregation, I tested whether sharing policy actually held up against a realistic exfiltration attempt: a user trying to share a restricted document externally.

<img src="Screenshots/cant%20share(nurse%20to%20external).PNG" alt="External Sharing Blocked" width="300">

*A Clinical-department test user attempting to share a document with an external (non-organizational) email address, blocked by Workspace sharing policy. This confirms the control works against an actual attempt rather than just existing in settings.*

<img src="Screenshots/EXTERNAL%20DRIVE%20SHARING.PNG" alt="External Drive Sharing Policy" width="600">

*The Admin Console sharing policy configuration that enforces the block shown above. Sharing outside the organization is restricted by default.*

---

## OAuth Governance

Third-party OAuth apps are an underrated access-control gap. A user can grant a random PDF tool read/write access to their Drive content without IT ever knowing, unless someone is reviewing app authorizations and audit logs. I simulated this exact scenario.

A non-admin test user authorized two third-party apps (Adobe Acrobat, SmallPDF) requesting Drive scopes. I reviewed the resulting authorization events in the Workspace Audit Log to confirm visibility into what had been granted and by whom, then locked down the policy so future third-party app authorizations require admin approval before they can complete, closing the gap rather than just documenting it.

<img src="Screenshots/OAuth%20SMallPDF%20add-on%20install%20permisions%20request%20for%20nurse.PNG" alt="OAuth SmallPDF Permission Request" width="400">

*The OAuth consent screen a test user sees when authorizing SmallPDF, showing the Drive scopes being requested. This is the exact moment unreviewed third-party access typically gets granted in real organizations.*

<img src="Screenshots/OAuth%20Log%20events%20showing%20small%20psf%20and%20adobe%20acropat%20granted%20by%20nurse.PNG" alt="OAuth Audit Log" width="600">

*Workspace Audit Log entries showing both third-party app authorizations (SmallPDF and Adobe Acrobat) granted by the test user, confirming these events are logged and reviewable.*

<img src="Screenshots/configuring%20accesss%20for%20third%20party%20integratino%20(squarespace%20login).PNG" alt="Third-Party Integration Configuration" width="400">

*Configuring Workspace's third-party app access policy to require administrator approval going forward.*

<img src="Screenshots/Access%20blocked%20for%20small%20pdf%20for%20nurse%20after%20change.PNG" alt="Access Blocked After Policy Change" width="600">

*After the policy change, the same test user attempting to authorize SmallPDF again is now blocked pending admin approval. This validates the control actually took effect rather than just existing in a settings page.*

---

## Gmail Security

Reviewed and validated Gmail's built-in protections, then tested them against a simulated phishing attempt rather than just confirming the toggles were on.

<img src="Screenshots/gmail%20safety%20settings.PNG" alt="Gmail Safety Settings" width="600">

*Gmail security settings in the Admin Console. External sender warnings, anti-spoofing protections, Safe Browsing, and spam filtering all enabled at the domain level.*

<img src="Screenshots/external%20email%20phising%20attempt%20marked%20as%20external.PNG" alt="External Email Marked as External" width="600">

*A simulated phishing email correctly flagged with Gmail's external-sender warning banner, confirming the protection is active and visible to end users, not just configured silently in the backend.*

---

## Email Authentication (SPF / DKIM / DMARC)

Configured the full email authentication stack at the DNS level for the custom domain. These controls matter because they stop domain spoofing, which is the basis for most business email compromise attacks. Without them, anyone can send an email that claims to be from billing@bagnislabs.com with no technical barrier at all. That's the kind of thing attackers use to impersonate a CFO for a wire transfer, or impersonate IT to harvest credentials.

Quick rundown on what each one does:

- **SPF** lists which mail servers are allowed to send for the domain. If the sending server isn't on the list, it fails.
- **DKIM** signs outgoing mail with a private key, and the receiving server checks it against a public key in DNS. This checks integrity, not just origin.
- **DMARC** tells receiving servers what to do when a message fails SPF or DKIM, and gives the domain owner reporting on who's sending mail claiming to be them.

<img src="Screenshots/showing%20DNS%20of%20DMARC%20added.PNG" alt="DMARC DNS Configuration" width="600">

*DMARC TXT record added at the DNS provider, specifying policy and reporting configuration for the domain.*

This is published at _dmarc.bagnislabs.com and sets the enforcement policy for mail that fails SPF/DKIM, plus an address for aggregate reports. In a real rollout I'd start this in monitor-only mode first to make sure legitimate mail isn't getting caught before tightening it.

<img src="Screenshots/Squarespace%20DNS%20management%20showing%20v=SPF%20and%20DKIM.PNG" alt="SPF &amp; DKIM DNS Records" width="600">

*SPF and DKIM records configured in DNS management, authorizing Google Workspace as a legitimate sender for the domain and enabling cryptographic signing of outbound mail.*

The SPF record lists Google's sending infrastructure as authorized for the domain. The DKIM record is the public half of the signing key; Google Workspace holds the private key and signs outbound mail with it.

<img src="Screenshots/Nurse%20to%20external%20SPF%20PASS.PNG" alt="SPF PASS" width="600">

*Message header inspection confirming SPF returns a PASS result on outbound mail. The sending source is authorized per the domain's SPF record.*

This is the validation step, actually checking a sent message's headers instead of assuming the DNS records work just because they were entered correctly. The Authentication-Results header shows spf=pass here, confirming the message came from an authorized source.

<img src="Screenshots/nurse%20to%20external%20showing%20spf%20pass%20and%20dmarc%20pass.PNG" alt="SPF, DKIM &amp; DMARC PASS" width="600">

*Full header validation showing SPF, DKIM, and DMARC all returning PASS on a test message sent externally, confirming the complete authentication chain is functioning end to end, not just individually configured.*

All three pass here together, which matters because they can fail independently for different reasons. Seeing spf=pass, dkim=pass, and dmarc=pass all at once is what it looks like when the whole chain is actually working, not just one piece quietly covering for another.

---

## Key Performance Indicators

Borrowed directly from how this would actually get reported to leadership: the kind of KPI table a security analyst owns and presents.

| Metric | Result | Target | Status |
|---|---|---|---|
| Access entitlements reviewed | 18 | 100% of active entitlements | ✅ Met |
| Unauthorized access findings | 3 | 0 | ⚠️ Action Required |
| Findings remediated same-cycle | 3 of 3 (100%) | 100% | ✅ Met |
| MFA adoption | 100% (6 of 6 users) | 100% | ✅ Met |
| Users with excess privilege post-remediation | 0 | 0 | ✅ Met |
| Open findings carried to next cycle | 0 | 0 | ✅ Met |

*(Full KPI breakdown: [`Bagnis_Labs_Q2_2026_Access_Review.xlsx`](./Audit%20Report/Bagnis_Labs_Q2_2026_Access_Review.xlsx), "Summary" tab)*

---

## How This Maps to HIPAA

This lab wasn't built around a specific compliance checklist, but several of the HIPAA Security Rule's core requirements showed up naturally because they're just good IAM practice:

- **Access Control (§164.312(a))**: unique user identification, the entire RBAC/least-privilege model, and automatic logoff/MFA enforcement
- **Audit Controls (§164.312(b))**: Workspace Audit Logs used to review OAuth grants and admin activity
- **Transmission Security (§164.312(e))**: SPF/DKIM/DMARC reducing the risk of spoofed mail carrying PHI-adjacent content
- **Periodic Access Review**: the quarterly access review itself, which the Security Rule's administrative safeguards expect as ongoing practice, not a one-time setup task

---

## Lessons Learned / What I'd Do Differently at Enterprise Scale

- **Manual quarterly review has a blind spot between cycles.** I'd push for automated, more frequent group-membership diffing (Admin SDK / Reports API) rather than relying solely on point-in-time manual review. Drift can sit undetected for up to three months otherwise.
- **Google Groups as the sole RBAC mechanism doesn't scale past a certain org size.** At enterprise scale this would likely integrate with a dedicated IdP (Okta, Entra ID) for SSO and a proper IGA/GRC platform to automate access certifications instead of a manually-built spreadsheet. The spreadsheet works at six departments, not six hundred.
- **The Super Admin finding was the most important lesson of the whole project.** Least-privilege design failures on regular drives are bad; least-privilege failures on administrative roles are categorically worse, because they can undermine the very controls meant to catch them. I'd advocate for break-glass/just-in-time admin access patterns over standing Super Admin assignments in any real deployment.

---

## Skills Demonstrated

Identity & Access Management (IAM) · Google Workspace Administration · Role-Based Access Control (RBAC) · Least Privilege Design · Google Groups & Organizational Units · User Provisioning · Joiner Mover Leaver Lifecycle · Access Governance & Quarterly Access Reviews · OAuth Governance · Audit Log Review · Google Drive Security · Multi-Factor Authentication · Password Policy Administration · Gmail Security · SPF / DKIM / DMARC · DNS Administration · HIPAA-Aligned Security Documentation