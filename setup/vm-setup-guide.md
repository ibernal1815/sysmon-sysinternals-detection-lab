# VM Setup Guide

Step-by-step build instructions for the lab. Plan for 2 to 3 hours start to finish. You need VirtualBox 7+ installed, a Windows 11 Enterprise evaluation ISO, a Kali Linux ISO, and the Wazuh OVA or installer ready before starting.

## Downloads

[VirtualBox 7.0+](https://www.virtualbox.org/wiki/Downloads) and the Extension Pack.

[Windows 11 Enterprise Evaluation ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise) — around 5-6GB, valid for 90 days and renewable.

[Kali Linux Installer ISO](https://www.kali.org/get-kali/) — grab the installer image, not the live image.

[Sysinternals Suite](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) — download and extract somewhere on your host before you start.

[Wazuh OVA](https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html) — the pre-built OVA is the fastest way to get the server up.

## Part 1: Windows 11 Victim

### Create the VM

In VirtualBox Manager click New. Name it Win11-Victim, type Microsoft Windows, version Windows 11 64-bit. Assign 6GB RAM and create a 60GB dynamically allocated VDI disk.

Go into Settings before you start it.

Under System > Processor set it to 4 CPUs and enable PAE/NX. Under Display set video memory to 128MB. Under Storage attach your Windows 11 ISO to the IDE controller. Under Network set Adapter 1 to NAT and enable Adapter 2 as Internal Network named AttackLab.

### Install Windows

Start the VM and run through Windows Setup. When it asks for a network during OOBE click "I don't have internet" and create a local account named LabAdmin. Turn off all privacy settings. Once you're at the desktop you're done with the installer.

### Configure the OS

Open PowerShell as administrator and run through these in order.

Disable Defender:

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisableBehaviorMonitoring $true
New-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Defender" -Name DisableAntiSpyware -Value 1 -PropertyType DWORD -Force
```

Disable Windows Update:

```powershell
Stop-Service wuauserv
Set-Service wuauserv -StartupType Disabled
```

Enable RDP for lateral movement scenarios:

```powershell
Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```

Create a second test user:

```powershell
net user labuser Password123! /add
net localgroup administrators labuser /add
```

Set the static IP on Adapter 2. Open Network and Internet Settings, Change adapter options, right-click the Internal Network adapter, Properties, IPv4, and set it to 192.168.100.20 with subnet 255.255.255.0. Leave gateway and DNS blank.

Install VirtualBox Guest Additions from the Devices menu, then restart.

### Install Sysmon

Create `C:\Tools\` and copy `sysmon-config.xml` from this repo into it. Then run:

```powershell
cd C:\Tools
.\sysmon64.exe -accepteula -i sysmon-config.xml
Get-Service Sysmon64
```

The service should show Running. Open Event Viewer and navigate to Applications and Services Logs > Microsoft > Windows > Sysmon > Operational. Open Notepad on the desktop and refresh — you should see an Event ID 1 for notepad.exe. If you see that, Sysmon is working.

### Install Sysinternals

Extract the Sysinternals Suite to `C:\Tools\SysinternalsSuite\`. Launch procexp64.exe and autoruns64.exe once to accept the EULA so they don't prompt during a scenario.

### Install Wazuh Agent

Download the Wazuh agent MSI from the Wazuh dashboard after you have the server running (Part 3). Install it and point it at 192.168.100.30 during setup. Start the agent service:

```powershell
NET START WazuhSvc
Get-Service WazuhSvc
```

### Take Baseline Snapshot

Shut the VM down cleanly. In VirtualBox Manager go to Snapshots and take one named `Baseline-Ready` with a short description. This is the snapshot you restore to before every scenario.

## Part 2: Kali Linux Attacker

### Create the VM

New VM named Kali-Attacker, type Linux, version Debian 64-bit. 4GB RAM, 2 CPUs, 40GB dynamically allocated VDI. Attach the Kali ISO. Under Network set Adapter 1 to Internal Network AttackLab only — no NAT on this machine.

### Install Kali

Start the VM and select Graphical Install. Hostname kali-attacker, leave domain blank, set up a kali user with a password, guided full disk partitioning, default Xfce software selection. Install GRUB to /dev/sda and reboot.

### Configure the Network

```bash
sudo nano /etc/network/interfaces
```

Add:

```
auto eth0
iface eth0 inet static
    address 192.168.100.10
    netmask 255.255.255.0
```

Restart networking and verify:

```bash
sudo systemctl restart networking
ip addr show eth0
ping 192.168.100.20
```

If the interface name is not eth0, run `ip link show` to find the actual name and update the file accordingly.

### Install Additional Tools

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y impacket-scripts evil-winrm responder crackmapexec bloodhound neo4j
```

### Take Snapshot

Shut Kali down and take a snapshot named `Kali-Configured`.

## Part 3: Wazuh Server

### Deploy the OVA

Import the Wazuh OVA into VirtualBox via File > Import Appliance. After import go into Settings and set the network adapter to Internal Network AttackLab. Add a second adapter on NAT temporarily for the initial setup if you need to pull updates, then disable it once Wazuh is fully configured.

Start the VM. Default credentials are admin/admin — change these immediately. The Wazuh dashboard runs on HTTPS at port 443.

Set a static IP:

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

Set the AttackLab interface to 192.168.100.30/24 and apply:

```bash
sudo netplan apply
```

From the Windows victim, open a browser and navigate to `https://192.168.100.30`. Log in to the dashboard and grab the agent enrollment command for Windows from Agents > Deploy New Agent. Run that on the victim to connect the Wazuh agent to the manager.

### Take Snapshot

Once the agent is showing active in the dashboard, shut the Wazuh VM down and take a snapshot named `Wazuh-Configured`.

## Validation

From Kali, ping both other machines:

```bash
ping 192.168.100.20
ping 192.168.100.30
```

From Windows, test connectivity back:

```powershell
Test-NetConnection 192.168.100.10
Test-NetConnection 192.168.100.30
```

On Windows, open a command prompt and run `whoami`. Refresh Event Viewer under Sysmon/Operational and look for Event ID 1. Check the Wazuh dashboard and confirm events are coming in from the Windows agent.

Run a quick Nmap scan from Kali:

```bash
nmap -sV 192.168.100.20
```

Check Sysmon for Event ID 3 (network connection) entries. If you're seeing those in both Event Viewer and Wazuh, the pipeline is working end to end.

## Troubleshooting

If VMs cannot reach each other, confirm both are on the AttackLab internal network in VirtualBox settings, check the static IP config on each machine, and temporarily disable Windows Firewall to rule it out.

If Sysmon is not logging, check the service and reinstall if needed:

```powershell
Get-Service Sysmon64
Restart-Service Sysmon64
sysmon64.exe -c
sysmon64.exe -u
sysmon64.exe -i sysmon-config.xml
```

If the Wazuh agent is not connecting, verify the manager IP in the agent config at `C:\Program Files (x86)\ossec-agent\ossec.conf` and restart the WazuhSvc service.

If Kali networking is not coming up after a reboot, run `ip link show` to confirm the interface name and make sure it matches what is in `/etc/network/interfaces`.



