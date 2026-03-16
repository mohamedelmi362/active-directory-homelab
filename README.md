# Active Directory Home Lab — DC01

A hands-on Windows Server 2022 Active Directory lab built with VirtualBox, simulating a real enterprise environment with a domain controller, organizational units, user accounts, group memberships, and enforced security policies.

---

## Overview

This project sets up a fully functional Active Directory domain (`lab.local`) from scratch inside a virtual machine. It covers the full lifecycle of standing up a domain controller: OS installation, network configuration, AD DS role deployment, domain promotion, OU/user/group management, and Group Policy hardening — all verified with PowerShell.

---

## Lab Environment

| Component | Details |
|---|---|
| Hypervisor | Oracle VirtualBox |
| OS | Windows Server 2022 Standard Evaluation |
| VM Name | DC01 (Domain Controller 01) |
| RAM | 4096 MB |
| CPU | 2 processors |
| Storage | 50 GB |
| Domain | `lab.local` |
| Forest Functional Level | Windows Server 2016+ |
| Static IP | `10.0.0.50/24` |
| Default Gateway | `10.0.0.1` |
| DNS (Primary) | `127.0.0.1` (self) |
| DNS (Fallback) | `10.0.0.1` |

---

## Steps Completed

### 1. Virtual Machine Creation
Created a VirtualBox VM named **DC01** following enterprise naming conventions. Allocated 4 GB RAM and 2 vCPUs to support Active Directory services, with a 50 GB virtual disk.

### 2. OS Installation
Installed **Windows Server 2022 Standard (Desktop Experience)**. Initially downloaded the Core (no-GUI) version by mistake — after downloading the correct Desktop Experience ISO, installation completed without issues.

### 3. Static IP & DNS Configuration
Assigned a static IP (`10.0.0.50/24`) with the gateway and self-referencing DNS so the server can resolve its own AD records. Ran into a connectivity issue caused by VirtualBox bridging to the wrong physical adapter — fixed by selecting **Ethernet Adapter 3** in VirtualBox network settings, followed by `ipconfig /renew`.

### 4. Active Directory Domain Services Installation
Via **Server Manager → Add Roles and Features**, installed:
- Active Directory Domain Services (AD DS)
- DNS Server
- Group Policy Management

DNS is required because AD relies on SRV records for domain controller discovery by clients.

### 5. Domain Controller Promotion
Promoted DC01 to a domain controller for a new forest:
- **Domain:** `lab.local`
- **Forest Functional Level:** Windows Server 2016+
- **NetBIOS Name:** LAB

### 6. Organizational Units (OUs)
Created a logical OU structure in **Active Directory Users and Computers**:

```
lab.local
├── Users2          (general users)
├── HR-Team         (HR department)
└── IT-Helpdesk     (IT support staff)
```

OUs allow departmental Group Policy application and permission delegation, mirroring real enterprise structure.

### 7. Users & Group Memberships
Created user accounts and assigned them to their respective groups:

| User | Group |
|---|---|
| John Doe (`jdoe`) | HR-Team |
| Alice Smith | IT-Helpdesk |

Group membership drives role-based access control (RBAC) — users inherit policies and permissions from the groups they belong to.

### 8. Password Policy (Group Policy)
Configured the **Default Domain Policy** to enforce a minimum password length of **12 characters**:

```
Default Domain Policy
└── Computer Configuration
    └── Windows Settings
        └── Security Settings
            └── Account Policies
                └── Password Policy
                    └── Minimum password length: 12
```

This policy applies automatically to all user accounts in the domain, making brute-force attacks significantly harder.

### 9. PowerShell Verification
Verified all configuration programmatically:

```powershell
# Verify user creation and attributes
Get-ADUser jdoe

# Verify group membership
Get-ADGroupMember "HR-Team"
```

Expected output for `jdoe`:
```
DistinguishedName : CN=John Doe,OU=Users2,DC=lab,DC=local
Enabled           : True
UserPrincipalName : jdoe@lab.local
```

---

## Key Troubleshooting Notes

| Issue | Root Cause | Fix |
|---|---|---|
| ISO wouldn't boot correctly | Downloaded Core (no-GUI) version | Re-downloaded Desktop Experience ISO |
| `ping 10.0.0.1` failed after static IP config | VirtualBox bridged to wrong physical adapter | Changed bridged adapter to **Ethernet Adapter 3** and ran `ipconfig /renew` |

---

## Concepts Demonstrated

- **Active Directory Domain Services (AD DS)** — centralized authentication and authorization
- **DNS integration with AD** — SRV records for DC discovery
- **Organizational Units** — logical grouping for policy and delegation
- **Role-Based Access Control (RBAC)** — group-based permissions
- **Group Policy Objects (GPOs)** — domain-wide security enforcement
- **PowerShell for AD administration** — scripted verification over GUI clicks
- **Network troubleshooting** — layered debugging from VM settings to hypervisor config

---

## Skills Practiced

- Windows Server 2022 deployment
- VirtualBox VM configuration and networking
- Active Directory setup and administration
- Group Policy Management
- PowerShell (`Get-ADUser`, `Get-ADGroupMember`)
- Enterprise naming conventions and network architecture
