# Identity Inventory 

- Who or what can log into a company’s environment?
- Identity is not just a person.
It includes:

| Human Users | Privileged Accounts | Service Accounts | Application Identities | Device Identities |
|-------------|--------------------|------------------|------------------------|-------------------|
| Employees | Domain admins | Backup services | API tokens | Laptops |
| Contractors | Cloud admins | Database services | Service principals | Servers |
| Vendors | Security admins | Automation scripts | SaaS integrations | Virtual machines |
| Interns |  |  |  |  |

----

## Storing an Identity

- Depends on the company’s identity platform.
- Common systems:
    - Microsoft Entra ID (Azure AD)
    - Okta
    - Active Directory Users and Computers
    - AWS Identity and Access Management
- Each company maintains its own identity inventory

## Microsoft Entra: 

## Okta:

## Active Directory Users and Computers: 

----

## Information Included in Identity Inventory
Typically contains:
- Username / ID
- Email address
- Department
- Job role
- Group memberships
- Privilege level
- MFA status
- Last login
- Account status (Active / Disabled)

----

## Why Identity Inventory Is Important in SOC

Most modern attacks target identities instead of systems.
SOC uses identity inventory to:
-	Detect compromised accounts
-	Identify privilege abuse
-	Detect impossible travel
-	Monitor admin accounts
-	Investigate suspicious logins
-	Find unused or orphaned accounts

----

## How SOC L1 Uses Identity Inventory

When an alert is triggered, L1 checks:
1.	Who is the user?
2.	What access do they have?
3.	Is the login location normal?
4.	Is MFA enabled?
5.	Is this a human or service account?
6.	Is this a high-privilege account?

Based on that, they:
- Close the alert
or 
- Escalate to L2

----

## Risk Areas in Identity Inventory

SOC pays special attention to:
- Accounts without MFA
- Privileged accounts
- Service accounts with admin rights
- Dormant accounts
- Recently modified roles



