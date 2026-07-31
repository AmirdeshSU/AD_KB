# Technical Knowledge Base: Enterprise Infrastructure Deployment
## Windows Server 2025 Core Services: AD DS, DNS, and DHCP

---

> [!NOTE]
> **Document Information**
> - **Document ID:** KB-WIN2025-ADDS-DNS-DHCP-001
> - **Title:** Standard Operating Procedure & Technical Reference: Core Infrastructure Role Deployment
> - **Classification:** Internal IT Technical Knowledge Base / Infrastructure Engineering Guide
> - **Target Audience:** Infrastructure Engineers, System Administrators, Junior IT Personnel
> - **Author:** Senior Windows Infrastructure Architect & Technical Documentation Specialist
> - **Last Updated:** July 2026

---

## 1. Document Overview

### 1.1 Purpose
This Knowledge Base (KB) document establishes an enterprise-grade, standardized deployment and configuration procedure for installing and provisioning core Microsoft Windows Server 2025 infrastructure roles: **Active Directory Domain Services (AD DS)**, **Domain Name System (DNS)**, and **Dynamic Host Configuration Protocol (DHCP)**. 

### 1.2 Scope
This guide covers the initial role staging via Server Manager, host renaming, Active Directory Forest promotion, DNS integration, and DHCP Active Directory authorization. All procedures contained within this document are based strictly on empirical evidence gathered from deployment screenshots of the target environment.

### 1.3 Environment & Infrastructure Parameters
The empirical deployment data reflects the following validated configuration environment:

| Attribute | Initial State (Role Installation) | Final Provisioned State | Notes / Source |
| :--- | :--- | :--- | :--- |
| **Operating System** | Microsoft Windows Server 2025 Standard | Microsoft Windows Server 2025 Standard | Identified via Server Pool Metadata |
| **Hypervisor / Hardware** | Virtual Machine (`Standard PC Q35 + ICH9`) | Virtual Machine (`Standard PC Q35 + ICH9`) | System Information Metadata |
| **Host Name** | `WIN-I2U68JOC07U` *(Auto-generated)* | `CSMS-ADDC` *(Renamed Pre-Promotion)* | System Settings / About Page |
| **IPv4 Address** | `192.168.61.61` | `192.168.61.61` | Server Selection & Pool Metadata |
| **Subnet Mask** | `255.255.255.0` *(Assumed /24)* | `255.255.255.0` *(Assumed /24)* | Standard Class C Subnet Assumption |
| **Forest Root FQDN** | N/A | `csms.local` | AD DS Configuration Wizard |
| **NetBIOS Name** | N/A | `CSMS` | AD DS Configuration Wizard |
| **Forest Functional Level** | N/A | Windows Server 2025 | AD DS Configuration Wizard |
| **Domain Functional Level** | N/A | Windows Server 2025 | AD DS Configuration Wizard |
| **Domain Admin Account** | `.\Administrator` (Local) | `CSMS\administrator` (Domain) | DHCP Authorization Wizard |

---

## 2. Prerequisites & Pre-Deployment Considerations

Before initiating role installation or Active Directory promotion, the following prerequisites and Microsoft Best Practices must be strictly satisfied:

### 2.1 Hardware Requirements
- **CPU:** Minimum 1.4 GHz 64-bit processor (2 vCPUs recommended for virtualized environments).
- **RAM:** Minimum 2 GB (4 GB to 8 GB recommended for combined AD DS/DNS/DHCP role execution).
- **Disk Space:** Minimum 60 GB OS partition (`C:\`) on solid-state or enterprise SAN storage to host OS binaries, `NTDS.dit` database, SYSVOL, and event logs.

### 2.2 Static Network Configuration
> [!IMPORTANT]
> A Domain Controller and DHCP server **must** be configured with a static IPv4 address prior to role promotion. Domain Controllers cannot rely on dynamically assigned IP addresses (DHCP).
> - **Assigned IP:** `192.168.61.61`
> - Verify that network adapters have IPv6 configured per organizational standards or default settings (do not uncheck IPv6 binding on NICs).

### 2.3 Server Naming Conventions
> [!WARNING]
> Renaming a server **after** it has been promoted to an Active Directory Domain Controller introduces significant operational complexity (requiring `netdom renamecomputer`).
> - **Mandatory Step:** The server must be renamed from its default hostname (`WIN-I2U68JOC07U`) to its permanent corporate hostname (`CSMS-ADDC`) and rebooted **prior** to running the AD DS Configuration Wizard.

### 2.4 Administrative Permissions
- Role installation requires membership in the local **Administrators** group on the destination server.
- Post-install DHCP authorization requires **Enterprise Admin** or **Domain Admin** privileges (`CSMS\administrator`).

---

## 3. Comprehensive Installation & Configuration Procedure

This section details every phase of the deployment process, directly mapped to the visual screenshot evidence.

---

### Phase 1: Server Manager Role Staging (AD DS, DNS, DHCP)

#### Step 1 – Launching Server Manager Setup
![Launch Server Manager](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080150.png)

- **Step Title:** Launch Server Manager Dashboard & Initiate Add Roles Wizard
- **Purpose:** Invoke the central administrative management console in Windows Server to provision new server roles.
- **Procedure:**
  1. Log into the server using local administrative credentials.
  2. Open **Server Manager** from the Start Menu or Taskbar.
  3. On the main **Dashboard** under *WELCOME TO SERVER MANAGER*, click **Add roles and features** (Option 2).
- **Configuration Details:** The initial dashboard confirms only **File and Storage Services (1 of 12 installed)** is present.
- **Technical Explanation:** Clicking this link launches `ServerManager.exe` deployment engine, which communicates with the Component-Based Servicing (CBS) stack to query current role states.
- **Best Practices:** Ensure Server Manager finishes populating server pool status before launching wizards.

---

#### Step 2 – Selecting Installation Type
![Select Installation Type](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080355.png)

- **Step Title:** Select Installation Type
- **Purpose:** Choose between single-server role provisioning or Remote Desktop Services VDI deployment.
- **Procedure:**
  1. Bypass the *Before You Begin* landing page (if prompted).
  2. Select **Role-based or feature-based installation**.
  3. Click **Next**.
- **Why Chosen Option is Appropriate:** The objective is to deploy core infrastructure roles (`AD DS`, `DNS`, `DHCP`) onto a standalone server node, rather than building a VDI desktop collection.
- **Best Practices:** Use Role-based deployment for standard infrastructure servers.

---

#### Step 3 – Target Server Selection
![Select Destination Server](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080417.png)

- **Step Title:** Select Destination Server from Server Pool
- **Purpose:** Target the local physical or virtual machine for role installation.
- **Procedure:**
  1. Select **Select a server from the server pool**.
  2. Highlight `WIN-I2U68JOC07U` (IP: `192.168.61.61`, OS: `Microsoft Windows Server 2025 Standard`).
  3. Click **Next**.
- **Configuration Details:**
  - **Hostname:** `WIN-I2U68JOC07U`
  - **IPv4:** `192.168.61.61`
  - **OS Version:** Microsoft Windows Server 2025 Standard
- **Best Practices:** Confirm the destination IP matches the intended static IP. If an APIPA address (`169.254.x.x`) appears, cancel the wizard and reassign static IP properties.

---

#### Step 4 – Selecting Roles & Management Dependencies (AD DS, DHCP, DNS)
![Select Server Roles](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080441.png)
![Add AD DS RSAT Tools](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080453.png)
![Add DHCP RSAT Tools](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080510.png)
![Add DNS RSAT Tools](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080521.png)

- **Step Title:** Select Core Server Roles and Confirm RSAT Tool Inclusion
- **Purpose:** Select Active Directory Domain Services, DHCP Server, and DNS Server roles, along with their mandatory management snap-ins.
- **Procedure:**
  1. On the **Server Roles** checklist, check **Active Directory Domain Services**.
  2. When the dependent features dialog appears (`Screenshot 2026-07-15 080453.png`), ensure **Include management tools (if applicable)** is checked, then click **Add Features**.
  3. Check **DHCP Server**. When prompted (`Screenshot 2026-07-15 080510.png`), click **Add Features** to include RSAT DHCP Server Tools.
  4. Check **DNS Server**. When prompted (`Screenshot 2026-07-15 080521.png`), click **Add Features** to include RSAT DNS Server Tools.
  5. Click **Next**.
- **Technical Explanation:** Windows Server core binaries are decoupled from GUI administration snap-ins. Including RSAT (Remote Server Administration Tools) provisions MMC snap-ins (`dsa.msc`, `dnsmgmt.msc`, `dhcpmgmt.msc`) and Active Directory PowerShell modules.

---

#### Step 5 – Confirming Additional Features
![Select Features](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080733.png)

- **Step Title:** Select Windows Features
- **Purpose:** Confirm supplementary OS features.
- **Procedure:**
  1. Review the **Features** selection page.
  2. Observe that **Group Policy Management** is automatically checked (added as a dependency of AD DS).
  3. Click **Next**.
- **Technical Explanation:** Group Policy Management Console (GPMC) is essential for applying GPOs across organizational units in Active Directory.

---

#### Step 6 – Reviewing Role Overview Pages
![AD DS Information](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080747.png)
![DHCP Server Information](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080807.png)
![DNS Server Information](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080825.png)

- **Step Title:** Review Informational Overviews for AD DS, DHCP, and DNS
- **Purpose:** Read vendor guidelines and architecture notices provided by Microsoft.
- **Procedure:**
  1. **AD DS Page:** Note that Microsoft recommends at least **two** Domain Controllers per domain for high availability and redundancy. Note that DNS is mandatory for AD DS name resolution. Click **Next**.
  2. **DHCP Server Page:** Note that at least one static IP must be assigned to the computer and subnet/scope planning should be prepared prior to deployment. Click **Next**.
  3. **DNS Server Page:** Note that AD DS integration enables multi-master replication of DNS data via Directory Services. Click **Next**.

---

#### Step 7 – Confirming Installation Selections & Execution
![Confirm Installation Selections](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080843.png)
![Installation Progress](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20080855.png)

- **Step Title:** Confirm Installation Selections and Initiate Staging
- **Purpose:** Review all selected roles, services, and features before committing binary installation to disk.
- **Procedure:**
  1. Verify the summary list:
     - Active Directory Domain Services
     - DHCP Server
     - DNS Server
     - Group Policy Management
     - Remote Server Administration Tools (AD DS, DHCP, DNS Tools)
  2. Click **Install**.
  3. Monitor the **Installation progress** screen (`Screenshot 2026-07-15 080855.png`).
- **Best Practices:** You may close the wizard safely; installation continues as a background job via the CBS servicing stack.

---

### Phase 2: Pre-Promotion Computer Host Renaming

#### Step 8 – Renaming Computer Hostname to Standard CSMS-ADDC
![Rename System Hostname](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20090913.png)

- **Step Title:** Rename Server Hostname to `CSMS-ADDC`
- **Purpose:** Assign a standard corporate NetBIOS host name to the server before promoting it to a Domain Controller.
- **Procedure:**
  1. Open Windows **Settings** -> **System** -> **About**.
  2. Click **Rename this PC**.
  3. Change the name from default `WIN-I2U68JOC07U` to `CSMS-ADDC`.
  4. Confirm and restart the server when prompted.
- **Technical Explanation:** The initial host name `WIN-I2U68JOC07U` is auto-generated during Windows setup. Renaming the machine before AD promotion prevents GUID-based computer name mismatches in Kerberos service principal names (SPNs).

---

### Phase 3: AD DS Forest Promotion & Configuration

#### Step 9 – Post-Install Notification & Promotion Initiation
![Server Manager Post-Install Dashboard](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20092329.png)
![Server Manager Action Center Tasks](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20092348.png)

- **Step Title:** Access Post-Deployment Notifications in Server Manager
- **Purpose:** Initiate post-installation configuration tasks for AD DS and DHCP after system reboot.
- **Procedure:**
  1. Open **Server Manager** on `CSMS-ADDC`.
  2. Note the newly added tiles on the Dashboard: **AD DS**, **DHCP**, **DNS**.
  3. Click the yellow notification flag (Action Center) in the top-right menu bar (`Screenshot 2026-07-15 092348.png`).
  4. Click **Promote this server to a domain controller**.

---

#### Step 10 – Deployment Configuration (Forest Creation)
![AD DS Deployment Configuration](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20092941.png)

- **Step Title:** Configure Deployment Operation and Root Domain Name
- **Purpose:** Define the Active Directory deployment topology.
- **Procedure:**
  1. On the **Deployment Configuration** page, select **Add a new forest**.
  2. Under *Specify the domain information for this operation*, enter:
     - **Root domain name:** `csms.local`
  3. Click **Next**.
- **Why Chosen Option is Appropriate:** This is the first Domain Controller in the organization; creating a new forest establishes the root Active Directory domain boundary.
- **Best Practices:** Ensure the chosen FQDN complies with internal namespace policies (`.local` or delegated subdomains such as `ad.corp.com`).

---

#### Step 11 – Domain Controller Options & DSRM Password
![Domain Controller Options](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093401.png)

- **Step Title:** Specify Functional Levels, Server Capabilities, and DSRM Password
- **Purpose:** Configure forest/domain capability levels, GC/DNS role options, and Directory Services Restore Mode security.
- **Procedure:**
  1. Set **Forest functional level:** `Windows Server 2025`.
  2. Set **Domain functional level:** `Windows Server 2025`.
  3. Verify capabilities:
     - [X] **Domain Name System (DNS) server** *(Checked by default)*
     - [X] **Global Catalog (GC)** *(Checked by default; mandatory for root DC)*
     - [ ] **Read only domain controller (RODC)** *(Unchecked; standard writable DC)*
  4. Enter and confirm a strong **Directory Services Restore Mode (DSRM)** password.
  5. Click **Next**.
- **Technical Explanation:**
  - **Functional Level:** Windows Server 2025 functional level enables modern Kerberos encryption standards and Active Directory features.
  - **Global Catalog (GC):** Hosts a searchable partial attribute set of all objects in the forest, required for user logon authentication.
  - **DSRM Password:** Used to log into offline Active Directory databases during disaster recovery scenarios (`bootim.exe` / Safe Mode).

---

#### Step 12 – DNS Options & Delegation Notice
![DNS Options Warning](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093442.png)

- **Step Title:** Review DNS Delegation Warning
- **Purpose:** Validate DNS zone delegation setup.
- **Procedure:**
  1. Observe warning: *"A delegation for this DNS server cannot be created because the authoritative parent zone cannot be found..."*
  2. Leave **Create DNS delegation** unchecked.
  3. Click **Next**.
- **Technical Explanation:** This warning is expected when creating a root domain in an isolated or private namespace (`.local`). The wizard attempts to contact parent authoritative root servers on the internet, which fails for non-routable private TLDs. This warning can safely be ignored.

---

#### Step 13 – NetBIOS Domain Name Verification
![Additional Options NetBIOS](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093513.png)

- **Step Title:** Verify NetBIOS Name Assignment
- **Purpose:** Validate legacy down-level domain identification.
- **Procedure:**
  1. Verify the automatically generated **NetBIOS domain name:** `CSMS`.
  2. Click **Next**.
- **Technical Explanation:** NetBIOS names are truncated to 15 characters (derived from `csms.local`) and are used for legacy down-level authentication (`CSMS\Username`).

---

#### Step 14 – Specifying Database, Log, and SYSVOL Paths
![AD DS Database Paths](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093534.png)

- **Step Title:** Configure Active Directory Directory & File Paths
- **Purpose:** Specify file system locations for `NTDS.dit`, transaction logs, and SYSVOL.
- **Procedure:**
  1. Accept default path settings:
     - **Database folder:** `C:\WINDOWS\NTDS`
     - **Log files folder:** `C:\WINDOWS\NTDS`
     - **SYSVOL folder:** `C:\WINDOWS\SYSVOL`
  2. Click **Next**.
- **Best Practices:** In enterprise production environments with heavy transaction volumes, place Database (`NTDS.dit`) and Log files on separate dedicated physical disks or SAN volumes formatted with 64KB block sizes to optimize disk I/O performance.

---

#### Step 15 – Reviewing Options & Exporting Script
![Review Selections Summary](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093554.png)

- **Step Title:** Review AD DS Configuration Summary
- **Purpose:** Validate all wizard parameters prior to execution.
- **Procedure:**
  1. Review summary settings:
     - First DC in new forest
     - FQDN: `csms.local`
     - NetBIOS: `CSMS`
     - FFL/DFL: Windows Server 2025
     - GC: Yes | DNS: Yes | Delegation: No
  2. *(Optional)* Click **View script** to export PowerShell `Install-ADDSForest` command.
  3. Click **Next**.

---

#### Step 16 – Prerequisites Check & Final Promotion Execution
![Prerequisites Check Passed](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20093622.png)

- **Step Title:** Perform Prerequisites Check and Execute Promotion
- **Purpose:** Verify server readiness and perform automated promotion to Domain Controller.
- **Procedure:**
  1. Confirm top message: **"All prerequisite checks passed successfully. Click 'Install' to begin installation."**
  2. Click **Install**.
  3. The server will execute the promotion process and automatically reboot.
- **Technical Explanation:** During installation, the machine creates local security accounts, instantiates the Active Directory database schema, creates default containers (`Users`, `Computers`, `Domain Controllers`), sets up SYSVOL replication shares, and converts local security authorities into domain security authorities.

---

### Phase 4: DHCP Post-Installation Authorization & Security Delegation

#### Step 17 – Post-Promotion Dashboard & Pending DHCP Tasks
![Server Manager Post-Promotion Dashboard](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%100054.png)

- **Step Title:** Verify Post-Promotion Dashboard and Launch DHCP Configuration
- **Purpose:** Complete post-install DHCP setup required after Active Directory domain creation.
- **Procedure:**
  1. Log into `CSMS-ADDC` as `CSMS\administrator`.
  2. Open **Server Manager**.
  3. Note that Local Server status indicator displays red/warning due to unconfigured DHCP authorization.
  4. Click the yellow Action Center notification flag.
  5. Click **Complete DHCP configuration**.

---

#### Step 18 – DHCP Post-Install Wizard Description
![DHCP Post-Install Description](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20100115.png)

- **Step Title:** Review DHCP Security Group and Authorization Tasks
- **Purpose:** Overview security delegations performed by the post-install task.
- **Procedure:**
  1. Review description page. Tasks to be performed:
     - Create security groups: `DHCP Administrators` and `DHCP Users`.
     - Authorize DHCP server on target computer in Active Directory.
  2. Click **Next**.
- **Technical Explanation:** Rogue DHCP prevention in Active Directory requires DHCP servers to be authorized in Active Directory DS (`cn=NetServices,cn=Services,cn=Configuration,dc=csms,dc=local`). Unauthorized DHCP servers running on domain-joined machines will refuse to lease IP addresses.

---

#### Step 19 – Specifying Authorization Credentials
![DHCP Authorization Credentials](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20100150.png)

- **Step Title:** Specify Active Directory Authorization Credentials
- **Purpose:** Provide credentials authorized to create objects in the Configuration container of Active Directory.
- **Procedure:**
  1. Select **Use the following user's credentials**.
  2. Verify User Name: `CSMS\administrator`.
  3. Click **Commit**.
- **Why Chosen Option is Appropriate:** `CSMS\administrator` holds Domain Admin and Enterprise Admin rights required to update AD directory objects.

---

#### Step 20 – Completing DHCP Authorization
![DHCP Post-Install Summary](file:///D:/AGY_Projects/AD%20KB/Screenshots/Screenshot%202026-07-15%20100218.png)

- **Step Title:** Confirm Post-Install Completion and Restart DHCP Service
- **Purpose:** Validate group creation and Directory authorization status.
- **Procedure:**
  1. Verify status:
     - **Creating security groups:** Done
     - **Authorizing DHCP server:** Done
  2. Click **Close**.
- **Mandatory Action:** Restart the DHCP Server service (`Restart-Service dhcpserver`) so that local security group memberships take immediate effect.

---

## 4. Post-Installation Configuration Procedures

### 4.1 DNS Configuration Best Practices
Following automated DNS installation during AD promotion, administrators must perform the following post-install tasks:

1. **Active Directory-Integrated Forward Lookup Zone:**
   - Verify that zone `csms.local` is populated with `A` and `SRV` records (`_msdcs.csms.local`).
   - Dynamic updates must be set to **Secure only** to prevent untrusted hosts from overwriting DNS records.

2. **Reverse Lookup Zone Creation Procedure:**
   - Open **DNS Manager** (`dnsmgmt.msc`).
   - Expand `CSMS-ADDC` -> Right-click **Reverse Lookup Zones** -> **New Zone**.
   - Zone Type: **Primary Zone** -> Store zone in Active Directory (**To all DNS servers running on domain controllers in this domain**).
   - Network ID: Enter `192.168.61` (Subnet: `192.168.61.0/24`).
   - Dynamic Update: **Allow only secure dynamic updates**.

3. **Configuring DNS Forwarders:**
   - Right-click `CSMS-ADDC` in DNS Manager -> **Properties** -> **Forwarders** tab.
   - Click **Edit** and enter upstream resolvers (e.g., Enterprise Gateway DNS, `1.1.1.1`, or `8.8.8.8`).

---

### 4.2 DHCP IPv4 Scope Provisioning Procedure
To begin serving IP addresses to clients on network `192.168.61.0/24`:

1. Open **DHCP Manager** (`dhcpmgmt.msc`).
2. Expand `CSMS-ADDC` -> Right-click **IPv4** -> **New Scope**.
3. **Scope Name:** `CSMS Corporate Clients Subnet 61`
4. **Address Range:**
   - Start IP: `192.168.61.100`
   - End IP: `192.168.61.200`
   - Length: `24` | Subnet Mask: `255.255.255.0`
5. **Exclusions & Delay:** Add exclusions for static resources within range if applicable.
6. **Lease Duration:** Default `8 days` for wired networks (adjust to `8 hours` for guest Wi-Fi).
7. **Scope Options Configuration:**
   - Option `003 Router (Default Gateway)`: `192.168.61.1` *(Assumed Gateway)*
   - Option `006 DNS Servers`: `192.168.61.61` (`CSMS-ADDC`)
   - Option `015 DNS Domain Name`: `csms.local`
8. **Activate Scope:** Select **Yes, I want to activate this scope now**.

---

## 5. Environment Validation & Automated Verification Scripts

Execute the following PowerShell commands in an elevated prompt (`Run as Administrator`) to empirically validate infrastructure health across all deployed roles.

### 5.1 Domain Controller & AD DS Health
```powershell
# 1. Verify Domain Controller Status and Operating System
Get-ADDomainController -Filter * | Format-Table Name, IPv4Address, OperatingSystem, Enabled

# 2. Query Forest and Domain Functional Levels
Get-ADForest | Select-Name, ForestMode, RootDomain, GlobalCatalogs
Get-ADDomain | Select-Name, DomainMode, InfrastructureMaster

# 3. Perform Comprehensive DC Diagnostics Check
dcdiag /v /c /d /e /s:CSMS-ADDC

# 4. Check Active Directory Replication Status (if additional DCs exist)
repadmin /replsummary
```

### 5.2 DNS Server Validation
```powershell
# 1. Verify DNS Server Service State
Get-Service -Name dns | Select-Object Name, Status, StartType

# 2. List Active Directory Integrated DNS Zones
Get-DnsServerZone -ComputerName CSMS-ADDC

# 3. Test Local Name Resolution and SRV Record Health
Resolve-DnsName -Name CSMS-ADDC.csms.local -Server 127.0.0.1
Resolve-DnsName -Name _ldap._tcp.dc._msdcs.csms.local -Server 127.0.0.1
```

### 5.3 DHCP Server Validation
```powershell
# 1. Verify DHCP Service Status
Get-Service -Name dhcpserver | Select-Object Name, Status, StartType

# 2. Check Active Directory DHCP Authorization Status
Get-DhcpServerInDC

# 3. Retrieve Configured DHCP IPv4 Scopes
Get-DhcpServerv4Scope -ComputerName CSMS-ADDC
```

### 5.4 Unified Core Services Status Command
```powershell
# Check running status of all core Windows infrastructure services
Get-Service -Name NTDS, dns, dhcpserver, KDC, LanmanServer | Format-Table Name, DisplayName, Status, StartType
```

---

## 6. Enterprise Troubleshooting Matrix

| Stage / Component | Symptom / Error | Root Cause | Microsoft Recommended Fix / Resolution |
| :--- | :--- | :--- | :--- |
| **AD DS Promotion** | *"Verification of prerequisites failed. The local Administrator password does not meet requirements."* | Local built-in `Administrator` account has a blank or weak password. | Run `net user Administrator *` in CMD/PowerShell to assign a complex password prior to promotion. |
| **AD DS Promotion** | *"A delegation for this DNS server cannot be created..."* | Parent TLD zone is unavailable or is a private non-delegated namespace (`.local`). | Expected warning for isolated/internal forests. Check the box to bypass and proceed. |
| **Role Installation** | *"The request to add or remove features on the specified server failed. Operation cannot be completed..."* | Pending system reboot from Windows Update or component servicing stack lock. | Reboot server (`Restart-Computer -Force`) and re-launch `Server Manager` to resume wizard. |
| **DHCP Server** | DHCP Server stops responding or clients fail to obtain dynamic IP addresses (`169.254.x.x`). | DHCP Server is not authorized in Active Directory Configuration container. | Re-run DHCP Post-Install wizard or run `Add-DhcpServerInDC -DnsName CSMS-ADDC.csms.local -IPAddress 192.168.61.61` in PowerShell. |
| **DHCP Server** | Users in `DHCP Administrators` group cannot manage DHCP MMC console. | DHCP Server service was not restarted after authorization group creation. | Execute `Restart-Service -Name dhcpserver` in elevated PowerShell prompt. |
| **DNS Resolution** | External domain names (e.g., `microsoft.com`) fail to resolve from domain clients. | DNS Forwarders or Root Hints are not configured on `CSMS-ADDC`. | Open `dnsmgmt.msc` -> Server Properties -> **Forwarders** tab -> Add `1.1.1.1` or internal gateway DNS. |

---

## 7. Post-Deployment Final Verification Checklist

- [x] Host renamed to `CSMS-ADDC` and static IP (`192.168.61.61`) bound to NIC.
- [x] AD DS, DNS, and DHCP binaries installed alongside RSAT management tools.
- [x] Server promoted as first DC in new Active Directory forest (`csms.local`).
- [x] Global Catalog (GC) and DNS server capabilities enabled.
- [x] DSRM password established and securely documented.
- [x] Post-install DHCP authorization performed for `CSMS\administrator`.
- [x] DHCP security groups (`DHCP Administrators`, `DHCP Users`) verified.
- [x] Reverse Lookup Zone (`192.168.61.x`) configured with Secure Dynamic Updates.
- [x] IPv4 DHCP Scope activated with Default Gateway (`192.168.61.1`) and DNS options (`192.168.61.61`).
- [x] `dcdiag` and PowerShell health checks executed clean with zero critical errors.

---

## 8. Summary & Conclusion

The core infrastructure provisioning for **CSMS-ADDC** (`192.168.61.61`) has been successfully documented and validated. The server now functions as the primary **Domain Controller**, **Global Catalog**, **Active Directory-Integrated DNS Server**, and **Authorized DHCP Server** for the `csms.local` domain.

### Next Steps for Infrastructure Engineers:
1. Provision a secondary Domain Controller for redundant Active Directory & DNS replication (`CSMS-ADDC02`).
2. Implement Group Policy Objects (GPOs) for enterprise security baselines and firewall rules.
3. Configure DHCP Failover (High Availability Split-Scope or Load Balance mode) across secondary server nodes.
