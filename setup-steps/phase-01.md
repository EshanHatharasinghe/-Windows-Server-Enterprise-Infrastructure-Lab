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

