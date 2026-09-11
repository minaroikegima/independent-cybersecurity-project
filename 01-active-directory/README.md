\# Active Directory Deployment \& Hardening



\## Overview

Built a Windows Server 2019 Active Directory domain controller from scratch, then hardened it with domain-wide security policies and enabled advanced logging to support future SIEM detection work.



\## What Was Built

\- Windows Server 2019 Domain Controller (`homelab.local`)

\- Organizational Unit (OU) structure: `\_ADMIN`, `Employees`, `IT`, `Servers`, `Workstations`

\- User accounts and an `IT-Admins` security group

\- Domain-wide Group Policy configuration (password policy and audit policy)

\- PowerShell Module Logging and Script Block Logging enabled via GPO

\- Sysmon installed with the SwiftOnSecurity community configuration



\## Password Policy (via Group Policy)

\- Minimum password length: 12 characters

\- Complexity requirements: enabled

\- Maximum password age: 60 days

\- Password history: 10 remembered



\## Audit Policy (via Group Policy)

Enabled Success/Failure auditing for:

\- Credential Validation

\- User Account Management

\- Logon / Logoff

\- Special Logon



Without this, Windows doesn't log most authentication or account activity by default, so there'd be nothing for a SIEM to work with later. Turning this on now is what makes the detection work in later phases possible.



\## PowerShell Logging

\- Module Logging: enabled for all modules (`\*`)

\- Script Block Logging: enabled



This captures the actual commands run in PowerShell, including the real underlying code even when an attacker tries to obfuscate or encode a malicious script. PowerShell is a common tool used in real attacks since it's already built into every Windows machine, so this visibility matters.



\## Sysmon

Installed Sysmon using the community-maintained SwiftOnSecurity configuration, which logs process creation, network connections, and other system-level activity in much more detail than Windows provides by default.



\## Screenshots

\- `ad-ou-structure.png`: OU structure and user accounts in AD Users and Computers

\- `group-policy-password-policy.png`: Domain password policy settings

\- `group-policy-audit-policy.png`: Advanced audit policy configuration

\- `sysmon-event-viewer.png`: Sysmon actively logging events





