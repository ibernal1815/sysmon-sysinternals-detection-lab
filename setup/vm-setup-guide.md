# Virtual Machine Provisioning & Configuration Guide

This document provides the step-by-step procedures to provision, configure, and validate the three virtual machines comprising the detection laboratory. 

## Prerequisites & Media

Ensure all required media is downloaded prior to beginning the deployment phase.

| Resource | Version / Link | Purpose |
|---|---|---|
| **Hypervisor** | [VirtualBox 7.0+](https://www.virtualbox.org/wiki/Downloads) | Core virtualization platform and Extension Pack. |
| **Windows OS** | [Windows 11 Enterprise Eval](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise) | Target victim endpoint (90-day evaluation). |
| **Kali Linux** | [Kali Offline Installer](https://www.kali.org/get-kali/) | Offensive simulation platform. |
| **Wazuh SIEM** | [Wazuh OVA](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) | Pre-configured log aggregation and SIEM node. |
| **Sysinternals** | [Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) | Execution utilities for telemetry generation. |

---

## Phase 1: Windows 11 Target (Victim)

### 1.1 Virtual Machine Allocation
Provision a new virtual machine in VirtualBox with the following parameters:
*   **Name:** `Win11-Victim`
*   **Type/Version:** Microsoft Windows 11 (64-bit)
*   **CPU / RAM:** 4 vCPUs / 6GB (6144 MB) RAM (Enable PAE/NX)
*   **Storage:** 60GB Dynamically Allocated VDI
*   **Network Adapter 1:** NAT (Temporary for setup/updates)
*   **Network Adapter 2:** Internal Network (`AttackLab`)

### 1.2 OS Installation
Execute the Windows Setup process. When prompted for network connectivity during the Out-of-Box Experience (OOBE), select "I don't have internet" to bypass Microsoft Account requirements. Provision a local administrator account named `LabAdmin` and disable all telemetry/privacy toggles.

### 1.3 Operating System Configuration
Open an elevated PowerShell prompt (Run as Administrator) and execute the following configuration blocks.

**Disable Windows Defender (Permanent via Safe Mode):**
*Note: Standard `Set-MpPreference` commands do not survive reboots in modern Windows 11 builds. You must boot into Safe Mode (`msconfig` > Boot > Safe boot) to execute the following registry modifications.*

~~~cmd
reg add "HKLM\System\CurrentControlSet\Services\WinDefend" /v Start /t REG_DWORD /d 4 /f
reg add "HKLM\System\CurrentControlSet\Services\WdNisSvc" /v Start /t REG_DWORD /d 4 /f
reg add "HKLM\System\CurrentControlSet\Services\Sense" /v Start /t REG_DWORD /d 4 /f
~~~

**Disable Windows Update:**

~~~powershell
Stop-Service wuauserv
Set-Service wuauserv -StartupType Disabled
~~~

**Enable Remote Desktop (Lateral Movement Preparation):**

~~~powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
~~~

**Provision Secondary Test Account:**

~~~powershell
net user labuser Password123! /add
net localgroup administrators labuser /add
~~~

**Static Network Configuration:**
Identify the `AttackLab` adapter index using `Get-NetAdapter`, then assign the static IP:

~~~powershell
New-NetIPAddress -InterfaceAlias "Ethernet 2" -IPAddress 192.168.100.20 -PrefixLength 24
~~~
*Install VirtualBox Guest Additions and perform a system reboot.*

### 1.4 Telemetry & Tooling Deployment
Establish the `C:\Tools` directory and extract the Sysinternals Suite into `C:\Tools\SysinternalsSuite`. Launch `procexp64.exe` and `autoruns64.exe` once to accept the EULAs.

Deploy Sysmon using the provided baseline configuration:

~~~powershell
cd C:\Tools
.\sysmon64.exe -accepteula -i sysmon-config.xml
Get-Service Sysmon64
~~~

### 1.5 Wazuh Agent Registration
1. Download the Wazuh Agent MSI from the Wazuh Manager dashboard (configured in Phase 3).
2. Install the MSI, pointing the agent to the Manager IP (`192.168.100.30`).
3. Start the service:

~~~powershell
Start-Service WazuhSvc
~~~

**Snapshot:** Perform a graceful shutdown and capture a snapshot named `Baseline-Ready`.

---

## Phase 2: Kali Linux (Attacker)

### 2.1 Virtual Machine Allocation
*   **Name:** `Kali-Attacker`
*   **Type/Version:** Linux / Debian (64-bit)
*   **CPU / RAM:** 2 vCPUs / 4GB (4096 MB) RAM
*   **Storage:** 40GB Dynamically Allocated VDI
*   **Network Adapter 1:** Internal Network (`AttackLab`) *Only*

### 2.2 OS Installation
Select the Graphical Install. Assign the hostname `kali-attacker`, configure the primary user, utilize guided full-disk partitioning, and install the GRUB bootloader to `/dev/sda`.

### 2.3 Network Configuration
Configure the static interface via the terminal:

~~~bash
sudo nano /etc/network/interfaces
~~~

Append the following configuration:

~~~text
auto eth0
iface eth0 inet static
    address 192.168.100.10
    netmask 255.255.255.0
~~~

Restart the networking service to apply changes:

~~~bash
sudo systemctl restart networking
ip addr show eth0
~~~

### 2.4 Offensive Tooling Deployment
*(Note: Temporarily attach a NAT adapter if internet access is required to pull packages, then remove it).*

~~~bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y impacket-scripts evil-winrm responder crackmapexec bloodhound neo4j
~~~

**Snapshot:** Perform a graceful shutdown and capture a snapshot named `Kali-Configured`.

---

## Phase 3: Wazuh Server (SIEM)

### 3.1 Appliance Deployment
Import the Wazuh OVA via `File > Import Appliance`. 
Modify the network settings prior to boot:
*   **Adapter 1:** Internal Network (`AttackLab`)
*   **Adapter 2:** NAT (Temporary for initial setup/updates)

### 3.2 Network Configuration
Start the VM and log in using the default credentials (`admin` / `admin` — change these immediately). Modify the Netplan configuration for the static IP:

~~~bash
sudo nano /etc/netplan/00-installer-config.yaml
~~~

Apply the `192.168.100.30/24` schema to the `AttackLab` interface, then apply the configuration:

~~~bash
sudo netplan apply
~~~

**Snapshot:** Disconnect the NAT adapter, ensure the Wazuh dashboard is accessible via `https://192.168.100.30` from the Windows host, and capture a snapshot named `Wazuh-Configured`.

---

## Phase 4: Validation & Troubleshooting

Perform the following checks to ensure environmental integrity before executing attack scenarios.

### 4.1 Connectivity Validation
From the Kali Linux terminal:

~~~bash
ping -c 4 192.168.100.20
ping -c 4 192.168.100.30
~~~

From the Windows 11 PowerShell prompt:

~~~powershell
Test-NetConnection -ComputerName 192.168.100.10
Test-NetConnection -ComputerName 192.168.100.30 -Port 443
~~~

### 4.2 Telemetry Pipeline Validation
1.  On the Windows target, execute `whoami` in a command prompt.
2.  Open Windows Event Viewer (`Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`).
3.  Verify the generation of **Event ID 1** (Process Creation) for `whoami.exe`.
4.  Execute a port scan from Kali (`nmap -sV 192.168.100.20`).
5.  Verify the generation of **Event ID 3** (Network Connection) in both the local Sysmon Event Log and the Wazuh SIEM dashboard.

### 4.3 Common Troubleshooting

| Issue | Resolution |
|---|---|
| **No Network Connectivity** | Verify all VMs are assigned to the exact same Internal Network name (`AttackLab`). Ensure Windows Defender Firewall is not dropping ICMP. |
| **Sysmon Telemetry Missing** | Re-initialize the configuration: `sysmon64.exe -c` followed by `sysmon64.exe -i sysmon-config.xml`. Restart the service: `Restart-Service Sysmon64`. |
| **Wazuh Agent Disconnected** | Verify the Manager IP (`192.168.100.30`) is correct in `C:\Program Files (x86)\ossec-agent\ossec.conf`. Restart the `WazuhSvc` service. |
