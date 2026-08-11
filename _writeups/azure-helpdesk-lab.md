---
title: "Azure Help Desk Lab"
---

## Overview
**Goal:** Build a functional small-business Help Desk environment in Microsoft Azure, demonstrate Tier 1 support workflows end-to-end, and configure Windows security event monitoring to simulate SOC analyst review of Active Directory activity.

**Environment:** Microsoft Azure (West US 2), Windows Server 2022, Ubuntu 22.04 LTS, Windows 11 Pro, osTicket 1.18.1, Azure Log Analytics, Azure Monitor Agent, Active Directory Users and Computers (ADUC), PowerShell, KQL.

**Data/Evidence:** Windows Security event logs (EventIDs 4624, 4625, 4724, 4728, 4732, 4740) generated through real Tier 1 workflows and ingested into Azure Log Analytics for query.

---

## Scenario
A small-business IT environment requires the foundational components of any Help Desk operation: centralized authentication, a ticketing system, end-user workstations, and visibility into security events. The lab was built to mirror that operational footprint and to support repeated practice of the day-to-day tasks a Tier 1 technician performs, including password resets, account unlocks, group membership changes, and OU transfers. A SOC monitoring layer was added to validate that administrative actions and authentication events were being captured, ingested, and queryable through KQL.

---

## Build Methodology
1. Provisioned Azure foundation resources: resource group, virtual network, subnet, and network security group with inbound rules scoped to a single home IP address.
2. Deployed DC01 (Windows Server 2022), promoted it to domain controller for the `corp.lab` forest, and built out the OU, security group, and user structure in ADUC and PowerShell (`seed-corp-lab.ps1`).
3. Deployed TICKET01 (Ubuntu 22.04), installed the LAMP stack, deployed osTicket 1.18.1, and configured departments, help topics, and agent accounts to mirror the AD organizational structure.
4. Deployed CLIENT01 (Windows 11 Pro), configured DNS resolution to the domain controller, and joined the workstation to the `corp.lab` domain.
5. Configured Azure Log Analytics workspace and Data Collection Rule to collect Windows Event Logs from DC01 via Azure Monitor Agent.
6. Enabled Windows audit policies for Logon/Logoff, Account Management, Privilege Use, and Directory Service Changes using `auditpol`.
7. Generated representative security events (failed logons, password resets, group membership changes) and validated end-to-end log flow into Log Analytics.
8. Practiced five Tier 1 Help Desk tasks using both the ADUC interface and the equivalent PowerShell commands.

---

## Evidence

Active Directory Users and Computers showing the `corp.lab` domain structure with Helpdesk, Sales, and IT OUs populated with security groups and test users.
![ADUC Structure](/assets/images/azure-helpdesk-lab/aduc-structure.png)

osTicket ticket submission from Sarah Patel (Sales OU) reporting a software install request from her domain-joined Windows 11 workstation.
![osTicket Submission](/assets/images/azure-helpdesk-lab/ticket-submission.png)

osTicket agent dashboard showing the open ticket in Maria Lopez's queue, demonstrating the end-to-end ticket lifecycle from end-user to Help Desk technician.
![Agent Dashboard](/assets/images/azure-helpdesk-lab/agent-dashboard.png)

CLIENT01 (Windows 11 Pro) logged in as `CORP\spatel` and connected to the `corp.lab` domain. Command-line verification confirms domain authentication.
![Domain Logon](/assets/images/azure-helpdesk-lab/client01-domain-logon.png)

KQL query in Azure Log Analytics returning failed logon events (EventID 4625) captured from DC01, demonstrating end-to-end log pipeline from VM to workspace.
![KQL Failed Logons](/assets/images/azure-helpdesk-lab/kql-failed-logons.png)

Brute force detection query aggregating repeated failed logon attempts per source computer and computing the time window of attempts, flagging clusters consistent with automated attacks.
![KQL Brute Force](/assets/images/azure-helpdesk-lab/kql-brute-force.png)

---

## Findings

- **Finding 1 – Active Directory Lockout Disabled by Default:**
Out-of-the-box Windows Server 2022 ships with `LockoutThreshold=0`, meaning user accounts cannot be locked out regardless of how many failed authentication attempts occur. Confirmed by querying the default domain password policy with `Get-ADDefaultDomainPasswordPolicy`. A baseline lockout policy of 5 failed attempts and a 15-minute lockout duration was applied as a hardening step before account unlock workflows could be practiced.

- **Finding 2 – Password Flag Conflict on Reset:**
Forcing a password change at next logon (`Set-ADUser -ChangePasswordAtLogon $true`) fails silently with a warning when the account has `PasswordNeverExpires=$true`. The never-expires flag must be cleared first. This is a real production-environment gotcha and affected the password reset task practice in this lab.

- **Finding 3 – Azure Monitor Agent Table Schema:**
Windows security events ingested through the newer Azure Monitor Agent and Data Collection Rule pipeline are written to the `Event` table rather than the `SecurityEvent` table referenced in legacy Log Analytics documentation. All five documented KQL queries were adjusted to use the `Event` table.

- **Finding 4 – Ingestion Lag and Time Filters:**
Initial KQL queries using `where TimeGenerated > ago(1h)` returned empty result sets despite events being present locally on DC01. Investigation showed that Azure Monitor Agent batches and ships events with a delay of approximately 5 to 15 minutes. Removing the time filter or extending the window resolved the issue.

- **Finding 5 – Default Windows Audit Policy Insufficient for SOC Use:**
Default Windows audit policy does not log most events relevant to SOC monitoring. `auditpol` configuration was required to enable Success and Failure auditing for Logon, Logoff, Account Lockout, User Account Management, Security Group Management, Sensitive Privilege Use, and Directory Service Changes. After enabling, the expected EventIDs (4624, 4625, 4724, 4728, 4732, 4740) began appearing in the local Security log and were successfully ingested into Log Analytics.

- **Finding 6 – AMA Auto-Installation via DCR:**
Creating the Data Collection Rule and adding DC01 as a resource automatically triggered the installation of the `AzureMonitorWindowsAgent` extension on the VM. No manual agent installation was required. Provisioning status was visible under the VM's Extensions and applications blade and reported `Provisioning succeeded` within approximately three minutes.

---

## Tier 1 Help Desk Tasks Practiced

| Task                     | GUI (ADUC)                                       | PowerShell                                  |
|--------------------------|--------------------------------------------------|---------------------------------------------|
| Password reset           | Right-click user > Reset Password                | `Set-ADAccountPassword` + `Set-ADUser`      |
| Account unlock           | Properties > Account > Unlock                    | `Unlock-ADAccount`                          |
| Add to security group    | Right-click user > Add to a group                | `Add-ADGroupMember`                         |
| Disable account          | Right-click user > Disable Account               | `Disable-ADAccount`                         |
| Move between OUs         | Right-click user > Move                          | `Move-ADObject`                             |

Each task was performed against the lab environment, and corresponding security events were captured in Azure Log Analytics.

---

## Indicators and Artifacts

**Domain:**
- corp.lab

**Domain Controller:**
- DC01.corp.lab (10.10.1.4)

**Organizational Units:**
- Helpdesk, Sales, IT

**Security Groups:**
- HelpdeskAgents, SalesUsers, ITAdmins

**Key Security EventIDs Captured:**
- 4624 (Successful logon)
- 4625 (Failed logon)
- 4672 (Special privileges assigned)
- 4673 (Sensitive privilege use)
- 4724 (Password reset, admin-initiated)
- 4728 (Member added to global group)
- 4732 (Member added to local group)
- 4740 (Account locked out)

**Log Analytics:**
- Workspace: law-helpdesk-lab
- Data table: `Event` (AMA / DCR schema)

---

## What I Would Do Next

1. Add a second domain controller for high-availability and demonstrate AD replication using `repadmin`.
2. Implement Group Policy Objects for password complexity, screen lock, and PowerShell logging, and validate enforcement on CLIENT01.
3. Onboard CLIENT01 to Microsoft Defender for Endpoint and correlate endpoint telemetry with Log Analytics data.
4. Build a small Microsoft Sentinel deployment on top of the existing Log Analytics workspace and convert the documented KQL queries into scheduled analytics rules with incident creation.
5. Extend the lab with a second domain-joined client to enable lateral movement detection scenarios.

---

## Key Takeaway
Building a Help Desk environment manually, rather than relying on pre-built lab images, surfaces the real-world configuration details that documentation alone does not. The default Active Directory lockout setting, the password flag conflict, the AMA table schema, and the ingestion lag are all examples of details that only become visible when the environment is built and operated end-to-end. The same applies to the Tier 1 tasks themselves: the workflow looks simple until it is performed on a live domain where the order of operations matters.

---

**Project repository:** <https://github.com/RickJaimesDez/azure-helpdesk-lab>
