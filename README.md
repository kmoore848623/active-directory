<p align="center">
  <img src="https://i.imgur.com/pU5A58S.png" alt="Microsoft Active Directory Logo" width="200"/>
</p>

# On-Premises Active Directory in Azure

This project walks through deploying an on-premises-style Active Directory environment using Microsoft Azure. You'll simulate a traditional domain controller (DC) setup entirely in the cloud using Azure Virtual Machines (VMs).

---

## 🧰 Technologies Used

- Microsoft Azure (Virtual Machines)
- Remote Desktop Protocol (RDP)
- Active Directory Domain Services (AD DS)
- PowerShell

---

## 🖥️ Operating Systems

- Windows Server 2022 (for the Domain Controller)
- Windows 10 (21H2) (for the client machine)

---


## 🧩 Deployment Summary

### Step 1: Create the Virtual Machines

- Provision two VMs:
  - **DC-1**: Windows Server 2022
  - **Client-1**: Windows 10
- Ensure both are in the **same virtual network and region**.

---

### Step 2: Configure a Static IP Address

- On DC-1, navigate to:  
  `Azure Portal → Virtual Machines → DC-1 → Networking → IP Configurations → Set to Static`

> 🔒 Disable the Windows Firewall temporarily on DC-1 for easier connectivity during setup.

---

### Step 3: Set Up DNS

- On **Client-1**, update the DNS setting to point to DC-1's **private IP**:  
  `VM Settings → Networking → DNS Servers → Custom → <DC-1 IP>`

- Validate settings with:
```powershell
ipconfig /all
ping <DC-1-IP>
```

---

### Step 4: Install Active Directory Domain Services

- On DC-1:
  - Open **Server Manager → Add Roles and Features**
  - Select **Active Directory Domain Services**
  - After installation, click the flag to **promote the server** to a domain controller.
  - Choose **Add New Forest**, and set your root domain (e.g., `mydomain.com`).

---

### Step 5: Create Organizational Units & Admin User

- Open **Active Directory Users and Computers**
- Create two OUs: `_EMPLOYEES` and `_ADMINS`
- Create a new user under `_ADMINS` (e.g., `Jane_admin`)
- Add that user to the `Domain Admins` group via:
  - `User Properties → Member Of → Add → Domain Admins`

---

### Step 6: Join Client-1 to the Domain

- Log in to Client-1
- Go to:  
  `Settings → System → About → Rename this PC → Change → Join Domain → mydomain.com`

- After restarting, confirm domain join on DC-1 via AD Users & Computers → **Computers** folder

---

### Step 7: Enable Remote Desktop for Domain Users

- On Client-1 (as Jane_admin):
  - Go to:  
    `Settings → Remote Desktop → Select Users → Add Domain Users`

---

### Step 8: Create Bulk Users with PowerShell

- On DC-1, run the PowerShell script:  
  `scripts/create_users.ps1`

- This creates 100 users in the `_EMPLOYEES` OU with the default password `Password1`.

- To test, use RDP to connect as one of the generated users:  
  `mydomain.com\keith.moore` with password `Password1`

---

## 💡 Notes

- Ensure port **3389** (RDP) is open on your network security group (NSG).
- Rename or update the script as needed for different user counts or naming schemes.
- For production, secure passwords and firewall rules appropriately.
