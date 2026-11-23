# ⚙️ Phase 2: Core Infrastructure (AD DS & DNS)

In this phase, we transform the standalone Windows Server 2022 instance (**DC01**) into the primary identity and name resolution authority for the enterprise environment. We install the **Active Directory Domain Services (AD DS)** and **DNS Server** roles, promote `DC01` to the Primary Domain Controller for a new forest (`corp.local`), configure DNS Reverse Lookup Zones, and verify core directory services.

---

## 🎯 Phase Objectives
- Install **AD DS** and **DNS Server** roles via Server Manager.
- Promote `DC01` to a Primary Domain Controller for the root domain **`corp.local`**.
- Configure an Active Directory-Integrated **Reverse Lookup Zone** (`192.168.10.x`) for PTR record resolution.
- Verify `SYSVOL` and `NETLOGON` administrative shares, DNS forward/reverse lookups, and domain health via CLI.

---

## 🛠️ Step-by-Step Practical Implementation

### 1. Installing AD DS & DNS Server Roles
1. Log in to **DC01** as the Local Administrator and launch **Server Manager**.
2. Navigate to **Manage** → **Add Roles and Features**.
3. Select **Role-based or feature-based installation** and click **Next**.
4. Confirm that **DC01 (`192.168.10.10`)** is selected from the server pool and click **Next**.
5. Under **Server Roles**, select the following roles (click **Add Features** on the pop-up prompts for management tools):
   - **Active Directory Domain Services**
   - **DNS Server**
6. Click **Next** through the Features and role descriptions, then click **Install**.
7. Wait for the role binaries to finish installing before proceeding to promotion.

![AD DS and DNS Role Installation](../snapshots/p2-adds-install.png)

---

### 2. Promoting DC01 to Primary Domain Controller (`corp.local`)
1. In **Server Manager**, click the **Notifications Flag (⚠️)** at the top-right and select **Promote this server to a domain controller**.
2. **Deployment Configuration:**
   - Select **Add a new forest**.
   - Set **Root domain name** to **`corp.local`** and click **Next**.
3. **Domain Controller Options:**
   - Keep **Forest functional level** and **Domain functional level** at **Windows Server 2016** (default).
   - Ensure **Domain Name System (DNS) server** and **Global Catalog (GC)** are checked.
   - Enter a strong **Directory Services Restore Mode (DSRM)** password and click **Next**.
4. **DNS Options:** Acknowledge the DNS delegation warning (normal for a new root forest) and click **Next**.
5. **Additional Options:** Verify the NetBIOS domain name auto-populates as **`CORP`** and click **Next**.
6. **Paths:** Retain the default locations for the AD DS database (`NTDS`), log files, and `SYSVOL` folder (`C:\Windows\...`), then click **Next**.
7. **Review Options & Prerequisites Check:** Verify the configuration summary and click **Next**. Once *"All prerequisite checks passed successfully"* appears, click **Install**.

![Domain Controller Promotion Prerequisites](../snapshots/p2-dc-promotion.png)

8. The server will automatically reboot upon completion. Log back in using the new domain administrator credentials: **`CORP\Administrator`**.

---

### 3. Configuring DNS Reverse Lookup Zone (PTR Records)
To support bidirectional name resolution (IP-to-Hostname), configure an AD-integrated Reverse Lookup Zone:

1. Open **Server Manager** → **Tools** → **DNS**.
2. Expand **DC01**, right-click **Reverse Lookup Zones**, and select **New Zone...**.
3. Follow the New Zone Wizard with these enterprise parameters:
   - **Zone Type:** Primary zone (Ensure *"Store the zone in Active Directory"* is checked).
   - **AD Zone Replication Scope:** To all DNS servers running on domain controllers in this domain: `corp.local`.
   - **Reverse Lookup Zone Name:** IPv4 Reverse Lookup Zone.
   - **Network ID:** **`192.168.10`** (auto-populates as `10.168.192.in-addr.arpa`).
   - **Dynamic Update:** Allow only secure dynamic updates (recommended for Active Directory).
4. Click **Finish** to create the zone.
5. Navigate to **Forward Lookup Zones** → **`corp.local`**, double-click the **`DC01`** Host (`A`) record, check **Update associated pointer (PTR) record**, and click **OK**.

![DNS Forward and Reverse Lookup Zones](../snapshots/p2-dns-zones.png)

---

## 🔍 Verification & Testing

Open **Command Prompt (`cmd`)** as Administrator on `DC01` and run the following diagnostic commands to confirm AD DS and DNS health:

```cmd
:: 1. Verify SYSVOL and NETLOGON domain shares are active
net share

:: 2. Verify Forward and Reverse DNS name resolution
nslookup dc01.corp.local
nslookup 192.168.10.10

:: 3. Run Domain Controller DNS diagnostics
dcdiag /test:dns
