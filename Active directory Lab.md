
![[Screenshot 2026-03-07 125345.png]]
### Step 1: Creating the Virtual Machine
**What I Did:**
- Created a new VirtualBox VM named "DC01" (Domain Controller 01)
- Allocated 4096 MB RAM and 2 processors
- Set up 50GB storage for the operating system
- Configured boot order: Floppy, Optical, Hard Disk
**My Thinking:** I needed a dedicated VM for the domain controller that would serve as the central authentication point. The name DC01 follows enterprise naming conventions. I allocated sufficient resources (4GB RAM) for Active Directory services to run smoothly.

![[adding-iso-file.png]]
![[installing-windows.png]]
![[installingwindows.png]]
### Step 2: Installing Windows Server Operating System
**What I Did:**
- Inserted Windows Server 2022 ISO into the virtual optical drive
- Booted the VM from the ISO
- Selected "Windows Server 2022 Standard Evaluation" for installation
- Completed the OS installation
**My Thinking:** I chose Windows Server 2022 Standard because it's current and includes all necessary AD services. The Standard edition is sufficient for a home lab (Enterprise is overkill for learning purposes). The installation process is straightforward—mount ISO, boot, select OS variant, and let it install. I had extreme troubles booting the iso file, but after further evaluations I realized i downloaded only the core and not the Gui itself. After downloading the correct download, everything went smoothly.
![[configuering-network.png]]
![[cofiguering-dns.png]]
### Step 3: Setting Static IP Address & DNS Configuration
**What I Did:**
- Configured static IP, Set Default Gateway, Set DNS
**The Problem:** After configuration, the ping failed with "Destination host unreachable." My IP settings were perfect, but the VM couldn't reach the network.
**The Fix:** The issue wasn't my DNS/IP config—it was VirtualBox's bridged adapter. I was bridging to the wrong physical network adapter. I changed VirtualBox Settings → Network to use **"Ethernet adapter 3"** (the actual adapter my host uses), then ran `ipconfig /renew`, and ping worked immediately.
**My Thinking:** This taught me that network troubleshooting has layers. The VM's settings were correct, but the hypervisor wasn't connected to the right physical port. On machines with multiple adapters, you have to explicitly tell VirtualBox which one to use.
![[installing-AD.png]]
![[dns-to-router.png]]
### Step 4: Installing Active Directory Domain Services
**What I Did:**
- Opened Server Manager
- Selected "Add Roles and Features"
- Checked "Active Directory Domain Services"
- Selected additional features: DNS Server, Group Policy Management
- Completed the role installation wizard
**My Thinking:** Active Directory Domain Services is the core role for any domain controller. I also installed DNS because AD relies on DNS for service discovery—clients use DNS to find domain controllers (SRV records). Group Policy Management tools allow me to create policies that affect all users and computers in the domain.
### Step 5: Promoting to Domain Controller
**What I Did:**
- Created a new forest and domain (e.g., `lab.local`)
- Set Forest Functional Level to Windows Server 2016+
- Configured NetBIOS name
- Promoted the server to Domain Controller
**My Thinking:** A fresh forest is appropriate for a home lab—I'm not joining an existing corporate domain. `lab.local` is a standard private domain name (TLD reserved for testing). The functional level determines what features are available; 2016+ supports modern security features.
![[hr-team.png]]
![[created-users-groups.png]]
### Step 6: Creating Organizational Units (OUs)
**What I Did:**
- Opened "Active Directory Users and Computers"
- Created OUs for organizational structure:
- Users2 (for general users)
- HR-Team (for HR department)
- IT-Helpdesk (for IT support staff)
**My Thinking:** Organizational Units allow logical grouping of users for policy application and delegation. Rather than dumping all users in the default "Users" container, I created departmental OUs. This mimics real enterprise structure where different departments have different security policies and access levels.
![[assigning-roles-johndoe.png]]
![[assigning-role-asmith.png]]
### Step 7: Adding Users to Groups
**What I Did:**
- Selected the "HR-Team" group
- Opened group properties → "Members" tab
- Added "John Doe" to the HR-Team group
- Added "Alice Smith" to the IT-Helpdesk group
- Clicked Apply
**My Thinking:** Group membership determines what permissions and policies a user receives. By adding John Doe to HR-Team, he'll inherit:
- Any Group Policies applied to that group
- Permissions on shared folders assigned to "HR-Team"
- Department-specific security policies
This is the foundation for role-based access control (RBAC).
![[setting-password-policy.png]]
### Step 12: Setting Password Policy
**What I Did:**
- Opened "Group Policy Management Editor"
- Navigated to: Default Domain Policy → Computer Configuration → Windows Settings → Security Settings → Account Policies → Password Policy
- Set "Minimum password length" to 12 characters
- Confirmed changes apply to all domain users
**My Thinking:** Password policy is a critical security control. By enforcing 12-character minimum passwords domain-wide, I ensure users can't choose weak passwords like "Password1". This Group Policy applies to all user accounts in the domain automatically. A longer password makes brute-force attacks exponentially harder.
![[verify.png]]
**What I Did:**
- Opened PowerShell as Administrator
- Ran `Get-ADUser jdoe` to verify user creation
- Ran `Get-ADGroupMember "HR-Team"` to list group members
- Verified output shows correct user attributes:
    - DistinguishedName: `CN=John Doe,OU=Users2,DC=lab,DC=local`
    - Enabled: True
    - UserPrincipalName: `jdoe@lab.local`
**My Thinking:** PowerShell provides programmatic verification of AD changes. Rather than clicking through the GUI, I can verify in seconds that:
1. Users exist and have correct attributes
2. Group memberships are correct
3. Users are enabled/disabled as intended
This is how enterprise admins work at scale—PowerShell automation rather than manual clicks.