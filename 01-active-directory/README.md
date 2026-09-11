\# Active Directory Deployment \& Hardening



!\[Windows Server](https://img.shields.io/badge/Windows%20Server-2019-0078D6?logo=windows\&logoColor=white)

!\[Active Directory](https://img.shields.io/badge/Active%20Directory-Hardened-blue)

!\[PowerShell Logging](https://img.shields.io/badge/PowerShell%20Logging-Enabled-5391FE?logo=powershell\&logoColor=white)

!\[Sysmon](https://img.shields.io/badge/Sysmon-Enabled-brightgreen)



Built a Windows Server 2019 Active Directory domain controller from scratch, then hardened it with domain-wide security policies and enabled advanced logging to support the SIEM detection work planned for later phases.



\## Contents

\- \[What Was Built](#what-was-built)

\- \[Password Policy](#password-policy)

\- \[Audit Policy](#audit-policy)

\- \[PowerShell Logging](#powershell-logging)

\- \[Sysmon](#sysmon)

\- \[Challenges and Fixes](#challenges-and-fixes)



\## What Was Built



| Component | Details |

|---|---|

| Domain Controller | Windows Server 2019, domain `homelab.local` |

| OU Structure | `\_ADMIN`, `Employees`, `IT`, `Servers`, `Workstations` |

| Accounts | User accounts and an `IT-Admins` security group |

| Group Policy | Domain-wide password policy and audit policy |

| PowerShell Logging | Module Logging + Script Block Logging, via GPO |

| Sysmon | Installed with the SwiftOnSecurity community config |



!\[OU structure in Active Directory Users and Computers](ad-ou-structure.png)



\## Password Policy



| Setting | Value |

|---|---|

| Minimum password length | 12 characters |

| Complexity requirements | Enabled |

| Maximum password age | 60 days |

| Password history | 10 remembered |



!\[Domain password policy settings](group-policy-password-policy.png)



\## Audit Policy



Windows doesn't log most authentication or account activity by default. Turning this on was a prerequisite for the SIEM detection work in later phases — no audit logging means no data to alert on.



Enabled Success and Failure auditing for:



\- Credential Validation

\- User Account Management

\- Logon / Logoff

\- Special Logon



!\[Advanced audit policy configuration](group-policy-audit-policy.png)



\## PowerShell Logging



\- \*\*Module Logging:\*\* enabled for all modules (`\*`)

\- \*\*Script Block Logging:\*\* enabled



This captures the actual commands run in PowerShell, including the real underlying code even when someone tries to obfuscate or encode a script. PowerShell is a common tool in real attacks since it's already built into every Windows machine, so this visibility matters.



\## Sysmon



Installed Sysmon using the community-maintained SwiftOnSecurity configuration, which logs process creation, network connections, and other system-level activity in far more detail than Windows provides by default.



!\[Sysmon logging events in Event Viewer](sysmon-event-viewer.png)



\## Challenges and Fixes



Lost access to the original server after forgetting the local Administrator password, with no snapshot available to roll back to. Rebuilt the VM from scratch rather than fighting with boot media, and started saving credentials to a password manager going forward.

