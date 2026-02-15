# Virtual Machine Setup Guide

Complete step-by-step instructions for building the Sysmon + Sysinternals Detection Lab.

**Estimated Time:** 2-3 hours  
**Difficulty:** Intermediate  
**Prerequisites:** VirtualBox installed, ISO files downloaded

---

##  Required Downloads

Before starting, download these files:

1. **VirtualBox** (if not already installed)
   - Download: https://www.virtualbox.org/wiki/Downloads
   - Version: 7.0 or newer
   - Install VirtualBox Extension Pack (for better USB support)

2. **Windows 10/11 Evaluation ISO**
   - Download: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise
   - Version: Windows 11 Enterprise 22H2 (or Windows 11)
   - Size: ~5-6GB
   - Valid for 90 days (renewable)

3. **Kali Linux ISO**
   - Download: https://www.kali.org/get-kali/
   - Version: Latest (2024.x)
   - Variant: Installer (not live image)
   - Size: ~3.5GB

4. **Sysinternals Suite**
   - Download: https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite
   - Extract to save for later deployment

5. **Sysmon**
   - Download: https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon
   - Alternatively, included in Sysinternals Suite

---

##  Part 1: VirtualBox Network Configuration

### Step 1: Create Internal Network

1. Open **VirtualBox Manager**
2. Go to **File > Preferences > Network**
3. Click **NAT Networks** tab
4. Click the **+** icon to add a new NAT network (optional, for internet access)
5. We'll primarily use **Internal Network** which doesn't need pre-configuration

**Note:** Internal Networks are created automatically when you assign them to VMs. We'll use the name **"AttackLab"**.

---

##  Part 2: Windows 11 Victim Machine Setup

### Step 1: Create the Virtual Machine

1. Click **New** in VirtualBox Manager
2. Configure basic settings:
   - **Name:** Win11-Victim
   - **Type:** Microsoft Windows
   - **Version:** Windows 11 (64-bit)
   - Click **Next**

3. **Memory Size:** 6144 MB (6GB)
   - Adjust based on your available RAM
   - Minimum: 4GB, Recommended: 6GB

4. **Hard Disk:**
   - Select **Create a virtual hard disk now**
   - Click **Create**

5. **Hard Disk File Type:**
   - Select **VDI (VirtualBox Disk Image)**
   - Click **Next**

6. **Storage on Physical Hard Disk:**
   - Select **Dynamically allocated**
   - Click **Next**

7. **File Location and Size:**
   - Size: **60 GB**
   - Click **Create**

### Step 2: Configure VM Settings

1. Select **Win10-Victim** and click **Settings**

2. **System > Motherboard:**
   - Boot Order: Optical, Hard Disk (uncheck Floppy)
   - Extended Features: Enable I/O APIC ✓

3. **System > Processor:**
   - Processor(s): **4 CPUs**
   - Extended Features: Enable PAE/NX ✓

4. **Display:**
   - Video Memory: **128 MB**
   - Graphics Controller: VMSVGA
   - Enable 3D Acceleration: ✓ (optional)

5. **Storage:**
   - Click the **Empty** CD icon under Controller: IDE
   - Click the CD icon on the right > **Choose a disk file**
   - Select your **Windows 11 ISO**

6. **Network:**
   - **Adapter 1:**
     - Enable Network Adapter: ✓
     - Attached to: **NAT**
     - (We'll use this temporarily for Windows updates, then disable)
   
   - **Adapter 2:**
     - Enable Network Adapter: ✓
     - Attached to: **Internal Network**
     - Name: **AttackLab**

7. Click **OK** to save settings

### Step 3: Install Windows 10

1. Start the **Win11-Victim** VM
2. Windows Setup will begin:
   - Language: English (or your preference)
   - Click **Install Now**
   - Select **Windows 11 Enterprise Evaluation** (or Pro)
   - Accept license terms
   - Choose **Custom: Install Windows only (advanced)**
   - Select the virtual hard disk and click **Next**

3. Wait for installation (~10-15 minutes)
4. VM will restart automatically

5. **Out of Box Experience (OOBE):**
   - Region: Select your region
   - Keyboard: Select your keyboard layout
   - **Network:** Click "I don't have internet" (skip for now)
   - **Account:** Create local account "LabAdmin" with password
   - Security questions: Answer as desired
   - Privacy settings: Turn off all (for lab purposes)

6. Wait for Windows to complete setup

### Step 4: Initial Windows Configuration

Once logged in:

1. **Disable Windows Defender** (for attack testing):
   ```powershell
   # Open PowerShell as Administrator
   Set-MpPreference -DisableRealtimeMonitoring $true
   Set-MpPreference -DisableIOAVProtection $true
   Set-MpPreference -DisableBehaviorMonitoring $true
   New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name DisableAntiSpyware -Value 1 -PropertyType DWORD -Force
   ```

2. **Configure Network (Adapter 2 - Internal Network):**
   - Open **Network & Internet Settings**
   - Click **Change adapter options**
   - Right-click **Ethernet 2** (Internal Network) > **Properties**
   - Select **Internet Protocol Version 4 (TCP/IPv4)** > **Properties**
   - Configure:
     - IP address: **192.168.100.20**
     - Subnet mask: **255.255.255.0**
     - Default gateway: (leave blank)
     - DNS: (leave blank for now)
   - Click **OK**

3. **Disable Windows Firewall** (optional, for easier testing):
   ```powershell
   Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
   ```

4. **Enable File Sharing** (for certain attack scenarios):
   - Control Panel > Network and Sharing Center
   - Advanced sharing settings
   - Turn on file and printer sharing
   - Turn on network discovery

5. **Install VirtualBox Guest Additions:**
   - VM menu: **Devices > Insert Guest Additions CD image**
   - Open File Explorer > CD Drive > Run **VBoxWindowsAdditions.exe**
   - Restart after installation

### Step 5: Install Sysmon

1. **Download Sysmon config from this repo:**
   - Copy `sysmon-config.xml` to the VM (use shared clipboard or download)
   - Save to `C:\Tools\sysmon-config.xml`

2. **Install Sysmon:**
   ```powershell
   # Download Sysmon (or use from Sysinternals Suite)
   # Extract to C:\Tools\
   
   cd C:\Tools
   
   # Install Sysmon with config
   .\sysmon64.exe -accepteula -i sysmon-config.xml
   ```

3. **Verify Sysmon is running:**
   ```powershell
   Get-Service Sysmon64
   
   # Should show "Running"
   ```

4. **Check Sysmon logs:**
   - Open **Event Viewer**
   - Navigate to: **Applications and Services Logs > Microsoft > Windows > Sysmon > Operational**
   - You should see Event ID 1 (Process Creation) events

### Step 6: Install Sysinternals Suite

1. **Extract Sysinternals Suite:**
   - Create folder: `C:\Tools\SysinternalsSuite\`
   - Extract all tools to this folder

2. **Add to PATH** (optional):
   ```powershell
   $env:Path += ";C:\Tools\SysinternalsSuite"
   [Environment]::SetEnvironmentVariable("Path", $env:Path, [System.EnvironmentVariableTarget]::Machine)
   ```

3. **Test tools:**
   - Run `C:\Tools\SysinternalsSuite\procexp64.exe` (Process Explorer)
   - Run `C:\Tools\SysinternalsSuite\autoruns64.exe` (Autoruns)

### Step 7: Additional Configurations

1. **Disable Windows Update** (prevents interruptions):
   ```powershell
   Stop-Service wuauserv
   Set-Service wuauserv -StartupType Disabled
   ```

2. **Enable RDP** (for lateral movement scenarios):
   ```powershell
   Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
   Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
   ```

3. **Create additional test user:**
   ```powershell
   net user labuser Password123! /add
   net localgroup administrators labuser /add
   ```

### Step 8: Create Baseline Snapshot

1. **Shut down the VM** (Start > Shutdown)
2. In VirtualBox Manager, select **Win11-Victim**
3. Click **Snapshots** (top-right)
4. Click **Take** (camera icon)
5. Name: **"Baseline-Sysmon-Configured"**
6. Description: "Clean Windows 11 with Sysmon and Sysinternals installed"
7. Click **OK**

**Important:** This snapshot is your clean baseline. Restore to this before each new attack scenario.

---

##  Part 3: Kali Linux Attacker Machine Setup

### Step 1: Create the Virtual Machine

1. Click **New** in VirtualBox Manager
2. Configure:
   - **Name:** Kali-Attacker
   - **Type:** Linux
   - **Version:** Debian (64-bit)
   - **Memory:** 4096 MB (4GB)
   - **Hard Disk:** Create new, VDI, Dynamically allocated, **40GB**

### Step 2: Configure VM Settings

1. Select **Kali-Attacker** > **Settings**

2. **System > Processor:**
   - Processor(s): **2 CPUs**

3. **Display:**
   - Video Memory: **128 MB**

4. **Storage:**
   - Attach Kali Linux ISO to CD/DVD

5. **Network:**
   - **Adapter 1:**
     - Enable Network Adapter: ✓
     - Attached to: **Internal Network**
     - Name: **AttackLab**

6. Click **OK**

### Step 3: Install Kali Linux

1. Start **Kali-Attacker** VM
2. Select **Graphical Install**
3. Follow installation:
   - Language: English
   - Location: Your location
   - Keyboard: Your keyboard
   - Hostname: **kali-attacker**
   - Domain: (leave blank)
   - Root password: Create strong password
   - User: **kali** / password of choice
   - Partitioning: **Guided - use entire disk**
   - Write changes: **Yes**
   - Proxy: (leave blank)
   - Software selection: Default (Xfce desktop)

4. Wait for installation (~15-20 minutes)
5. Install GRUB: **Yes**, select **/dev/sda**
6. Reboot and log in

### Step 4: Configure Kali Network

1. Open terminal and become root:
   ```bash
   sudo su -
   ```

2. **Configure static IP:**
   ```bash
   nano /etc/network/interfaces
   ```

3. Add this configuration:
   ```
   auto eth0
   iface eth0 inet static
       address 192.168.100.10
       netmask 255.255.255.0
   ```

4. Save (Ctrl+O, Enter, Ctrl+X)

5. **Restart networking:**
   ```bash
   systemctl restart networking
   ```

6. **Verify IP:**
   ```bash
   ip addr show eth0
   # Should show 192.168.100.10
   ```

7. **Test connectivity to Windows VM:**
   ```bash
   ping 192.168.100.20
   # Should get responses
   ```

### Step 5: Install Additional Tools

1. **Update Kali:**
   ```bash
   apt update && apt upgrade -y
   ```

2. **Install useful tools:**
   ```bash
   apt install -y impacket-scripts evil-winrm powershell-empire \
                  responder crackmapexec bloodhound neo4j
   ```

3. **Download Mimikatz** (for Windows victim):
   ```bash
   cd /opt
   git clone https://github.com/gentilkiwi/mimikatz.git
   ```

### Step 6: Create Snapshot

1. Shut down Kali VM
2. Take snapshot: **"Kali-Configured"**

---

##  Part 4: Validation & Testing

### Test 1: Network Connectivity

**From Kali:**
```bash
ping 192.168.100.20
# Should get replies from Windows VM
```

**From Windows:**
```powershell
Test-NetConnection 192.168.100.10
# Should show TcpTestSucceeded: True
```

### Test 2: Sysmon Logging

**On Windows VM:**
1. Open **Event Viewer**
2. Navigate to **Sysmon/Operational**
3. Open Command Prompt and run: `notepad.exe`
4. Refresh Event Viewer
5. Look for Event ID 1 (Process Create) for `notepad.exe`

**Expected:** You should see Sysmon logging the notepad.exe process creation

### Test 3: Sysinternals Tools

**On Windows VM:**
1. Run **Process Explorer** (`procexp64.exe`)
   - Should show all running processes
2. Run **Autoruns** (`autoruns64.exe`)
   - Should show startup items
3. Run **TCPView** (`tcpview64.exe`)
   - Should show network connections

### Test 4: Simple Attack Simulation

**From Kali, test Nmap scan:**
```bash
nmap -sV 192.168.100.20
```

**On Windows, verify Sysmon captured:**
- Event ID 3 (Network Connection) events in Sysmon logs
- TCPView should show incoming connections

---

##  Snapshot Management Strategy

### Recommended Snapshots

Create these snapshots as you progress:

| VM | Snapshot Name | Description | When to Take |
|----|---------------|-------------|--------------|
| Windows | Fresh-Install | Clean Windows install | After OS install |
| Windows | Sysmon-Installed | Sysmon configured | After Sysmon setup |
| Windows | Baseline-Ready | All tools, ready for attacks | After all tools installed |
| Kali | Kali-Installed | Fresh Kali install | After OS install |
| Kali | Kali-Configured | Network and tools configured | After setup complete |

### Snapshot Workflow

**Before each attack scenario:**
1. Restore both VMs to baseline snapshots
2. Start VMs
3. Run attack scenario
4. Document findings
5. Optional: Take "Post-Attack" snapshot for comparison

**To restore:**
1. Select VM in VirtualBox Manager
2. Click **Snapshots**
3. Right-click snapshot > **Restore**
4. Choose whether to create checkpoint (recommended)

---

##  Troubleshooting

### Issue: VMs can't ping each other

**Solution:**
1. Verify both VMs on "AttackLab" internal network
2. Check static IP configuration
3. Disable Windows Firewall temporarily
4. Restart both VMs

### Issue: Sysmon not logging events

**Solution:**
```powershell
# Check service
Get-Service Sysmon64

# Restart service
Restart-Service Sysmon64

# Verify config
sysmon64.exe -c

# Reinstall if needed
sysmon64.exe -u
sysmon64.exe -i sysmon-config.xml
```

### Issue: Windows VM slow/unresponsive

**Solution:**
- Increase RAM allocation (try 8GB)
- Add more CPU cores
- Disable Windows Search service
- Disable Windows Update service

### Issue: Kali Linux network not working

**Solution:**
```bash
# Check interface name
ip link show

# If not eth0, update /etc/network/interfaces
# Restart networking
systemctl restart networking

# Or use NetworkManager
nmcli device
nmcli con add type ethernet ifname eth0 ip4 192.168.100.10/24
```

---

##  Setup Complete Checklist

Before proceeding to attack scenarios, verify:

- [ ] Windows 11 VM fully installed and updated
- [ ] Sysmon installed and logging events
- [ ] Sysinternals Suite extracted to C:\Tools\
- [ ] Windows Defender disabled
- [ ] Windows VM IP: 192.168.100.20
- [ ] Kali Linux VM fully installed
- [ ] Kali VM IP: 192.168.100.10
- [ ] Both VMs can ping each other
- [ ] Baseline snapshots created for both VMs
- [ ] Event Viewer showing Sysmon logs
- [ ] Process Explorer launches successfully

---

##  Next Steps

Your lab is now ready! Navigate to:
- `../scenarios/01-persistence-registry/` - Start your first attack scenario
- Test your detection skills with real attack techniques
- Practice using Sysinternals tools for investigation

**Remember:** Always restore to baseline snapshots between scenarios for consistent testing!

---

**Questions or issues?** Open an issue on the GitHub repository.


