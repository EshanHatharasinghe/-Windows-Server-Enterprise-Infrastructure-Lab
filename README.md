# -Windows-Server-Enterprise-Infrastructure-Lab (Zero to Hero)
An industrial-grade Windows Server 2022 infrastructure lab designed and implemented using Virtualization. This repository documents the deployment of core identity services, role separation concepts, centralized storage, network management, and security policies following enterprise IT standards.

## 1. Conceptual Enterprise Architecture

" In a production enterprise environment, workloads are isolated across dedicated Virtual Machines to guarantee high availability, fault tolerance, and security (Separation of Roles). " 


```
┌────────────────────────────────────────────────────────────────────────┐
│                        Physical Hyper-V / VMware Host                  │
│  ┌───────────────────────┐  ┌───────────────────────┐  ┌────────────┐  │
│  │         DC01          │  │         FS01          │  │   SQL01    │  │
│  │  - Active Directory   │  │  - File & Storage     │  │  - Database│  │
│  │  - DNS / DHCP         │  │  - Shared Folders     │  │  - Engine  │  │
│  └───────────┬───────────┘  └───────────┬───────────┘  └─────┬──────┘  │
└──────────────┼──────────────────────────┼────────────────────┼─────────┘
               │                          │                    │
               └──────────────────┬───────┴────────────────────┘
                                  │ (Internal Virtual Switch)
                                  ▼
                     [ Client Workstation (CL01) ]
```
                     
Architecture Note: While this proof-of-concept consolidates core roles within a virtualized server instance to optimize hardware resources during local testing, the blueprint adheres to enterprise role separation principles.

## 2. Network Topology & IP Addressing Scheme

| Operating System | IP Address |	Subnet Mask |	Preferred DNS |	Network Adapter |
_________________________________________________________________________________
DC01	Primary DC, DNS, DHCP, File Services	Windows Server 2022	192.168.10.10	255.255.255.0	127.0.0.1	Dual NIC (NAT + Internal CorporateLab)
CL01	Domain Client Workstation	Windows 10/11 Pro	Dynamic (DHCP)	255.255.255.0	192.168.10.10	Internal (CorporateLab)


## 3. Master Project Roadmap (Phases)


### Phase 1: Lab & Network Architecture Setup
- **Dual NICs Configuration on Windows Server 2022:**
  - `NIC 1 (External/NAT):` For internet connectivity and software updates.
  - `NIC 2 (Internal Network - CorporateLab):` For isolated domain traffic.
- **Static IP Configuration:** `192.168.10.10/24`, Subnet Mask, Gateway, and Loopback DNS (`127.0.0.1`).
- **System Preparation:** Hostname renaming to `DC01`.

### Phase 2: Core Infrastructure (AD DS & DNS)
- Installing Active Directory Domain Services (AD DS) and DNS Server roles.
- Promoting `DC01` to Primary Domain Controller for domain `corp.local`.
- Verifying DNS Forward & Reverse Lookup Zones, Active Directory SRV records, and SYSVOL/NETLOGON shares.

### Phase 3: Network Automation & IP Management (DHCP)
- Installing and Authorizing the DHCP Server role in Active Directory.
- Configuring DHCP Scope (`192.168.10.100 - 192.168.10.200`).
- Setting Scope Options:
  - `Option 003` (Router/Gateway)
  - `Option 006` (DNS Server - `192.168.10.10`)
  - `Option 015` (Domain Name - `corp.local`)
- Setting up static IP Reservations for critical hosts.

### Phase 4: Identity & Organizational Structure (OUs, Users & Security Groups)
- **OU Hierarchy Creation:** `Corporate_OUs` $\rightarrow$ `IT_Dept`, `HR_Dept`, `Finance_Dept`, `Workstations`, `Servers`.
- **Automated Bulk User Provisioning:** Creating 10+ Domain Users automatically via PowerShell scripts using a CSV data source.
- **Access Control:** Security Group Creation (Global vs Domain Local Groups) & Role-Based Access Control (RBAC) delegation.

### Phase 5: Centralized Storage & Access Control (File Services)
- Partitioning a dedicated Virtual Disk (`D:\DataDrive`).
- Setting up Shared Folders (`IT_Share`, `HR_Share`, `Company_Public`).
- Configuring Share Permissions vs NTFS Permissions following the **Least Privilege Principle**.
- Enabling **Access-Based Enumeration (ABE)** so users only see folders they have permission to access.

### Phase 6: Enterprise Security & Policy Management (GPO)
- Creating and linking Group Policy Objects (GPOs) to specific OUs:
  - **Automated Network Drive Mapping:** Auto-mapping `D:\DataDrive\Shares` as `Z:` Drive upon user login.
  - **Folder Redirection:** Redirecting Desktop & Documents to the File Server.
  - **Workstation Security Policies:** USB Drive Storage Block, Custom Corporate Desktop Wallpaper Enforcement, and Account Lockout Policy (5 failed attempts).
- Policy verification using `gpresult /r` and `gpupdate /force` on the client side.

### Phase 7: Client Integration & Domain Join
- Setting up Windows 10/11 Client Workstation (`CL01`) attached to the `CorporateLab` Internal Network.
- Obtaining dynamic IP configuration via DHCP.
- Joining `CL01` to the `corp.local` domain using Domain Admin credentials.
- Testing domain user logins (`corp\Kasun`) and verifying GPO/Drive Map enforcement.

### Phase 8: Block-Level SAN Storage Integration (iSCSI Target & Initiator) *(Pro-Level)*
- Installing iSCSI Target Server Role on Windows Server.
- Creating an iSCSI Virtual Disk (`.vhdx`) and assigning an iSCSI Target IQN.
- Connecting a Client/Member Server using iSCSI Initiator over the network, initializing, formatting, and mounting it as a raw local volume (`E:` Drive).

### Phase 9: Enterprise Maintenance, Patching & Deployment (WDS/WSUS Overview) *(Pro-Level)*
- **Windows Server Update Services (WSUS):** Configuring Update Approval Workflows & GPO-based Windows Update Redirection to local WSUS.
- **Windows Deployment Services (WDS):** Configuring PXE Boot environment with `boot.wim` and `install.wim` for automated network OS deployment.

---

## 🛠️ Tech Stack & Environment
- **Hypervisor:** Oracle VirtualBox / VMware Workstation
- **Server OS:** Windows Server 2022 Datacenter / Standard (Desktop Experience)
- **Client OS:** Windows 10 / 11 Pro
- **Automation:** PowerShell


 # 🚀 Phase 1: Lab & Network Architecture Setup

 In this initial phase, we configure the isolated virtual networking environment inside Oracle VirtualBox, assign dual network interfaces (Dual NICs), set the corporate Static IP schema, and configure the base server system properties.

---

## 🛠️ Step-by-Step Practical Implementation

### 1. VirtualBox Dual-NIC Network Configuration

1. Open **Oracle VirtualBox** and select the **Windows Server 2022** VM (`DC01`).
2. Navigate to **Settings** $\rightarrow$ **Network**.
3. **Adapter 1 (External / Internet Access):**
   * Tick **Enable Network Adapter**.
   * **Attached to:** `NAT` *(Used for Windows Updates and downloading necessary software/tools)*.
4. **Adapter 2 (Isolated Corporate Network):**
   * Tick **Enable Network Adapter**.
   * **Attached to:** `Internal Network`.
   * **Name:** `CorporateLab` *(Isolates Domain traffic between Domain Controllers and Client Workstations)*.
5. Click **OK** to save settings.

---

#### 2. System Hostname Renaming
1. Power on the **Windows Server 2022 VM** and log in as the **Local Administrator**.
2. Open **Server Manager** and click on **Local Server** from the left navigation pane.
3. Click on the current randomly generated **Computer Name**.
4. In the System Properties window, click **Change...**.
5. Enter **`DC01`** under **Computer Name** and click **OK**.
6. Click **Restart Now** to apply the hostname changes.

---

#### 3. Static IP Configuration (Internal Adapter - `CorporateLab`)
1. Log back into `DC01` after reboot.
2. Press `Win + R`, type **`ncpa.cpl`**, and press **Enter** to open Network Connections.
3. Identify the Internal Network Adapter (typically named **Ethernet 2**) and rename it to **`CorporateLab-NIC`** for clarity.
4. Right-click **`CorporateLab-NIC`** $\rightarrow$ **Properties**.
5. Select **Internet Protocol Version 4 (TCP/IPv4)** and click **Properties**.
6. Select **Use the following IP address** and enter the following enterprise IP schema:
   * **IP address:** `192.168.10.10`
   * **Subnet mask:** `255.255.255.0`
   * **Default gateway:** *(Leave blank – Internal isolated network)*
   * **Preferred DNS server:** `127.0.0.1` *(Loopback address; points to its own local Active Directory-Integrated DNS)*
7. Click **OK** and then **Close**.

---

### 🔍 Verification & Testing

Open **Command Prompt (`cmd`)** on `DC01` and execute the following commands to verify the baseline network setup:

```cmd
:: Verify system hostname
hostname

:: Verify static IP allocation and adapter options
ipconfig /all


 
