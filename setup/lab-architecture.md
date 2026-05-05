# Lab Architecture

## Overview

The lab runs three VMs on an isolated internal network inside VirtualBox. The Windows 11 machine is the victim, Kali is the attacker, and Wazuh handles log collection, alerting, and SIEM functionality. Nothing on the attack network touches the internet during active testing.

## Host Machine

Intel Core i5-14400F, 48GB DDR4, 1TB NVMe, Windows 11 Pro with VirtualBox 7. The three VMs consume around 14GB RAM combined, leaving plenty of headroom on the host.

## Virtual Machines

### Windows 11 Victim (192.168.100.20)

This is the target machine. It runs Sysmon with the SwiftOnSecurity config as a baseline, the full Sysinternals Suite, and Winlogbeat to forward logs to Wazuh.

4 cores, 6GB RAM, 60GB dynamically allocated disk. Two network adapters: NAT for initial setup and updates, and Internal Network "AttackLab" for lab traffic. Disable the NAT adapter before running any attack scenarios.

Windows Defender is disabled for controlled testing. Firewall stays enabled and gets toggled per scenario. UAC is at default. Windows Update is paused during active use. A local administrator account is enabled.

### Kali Linux Attacker (192.168.100.10)

The offensive platform. Standard Kali installation with Metasploit, Nmap, Impacket, Responder, Evil-WinRM, netcat, and custom PowerShell payloads staged here for transfer to the victim.

2 cores, 4GB RAM, 40GB dynamically allocated disk. One network adapter on the internal AttackLab network only. No internet access during testing.

### Wazuh Server (192.168.100.30)

Wazuh handles log ingestion from the Windows victim via the Wazuh agent (replacing Winlogbeat), runs detection rules including MITRE ATT&CK mapped alerts out of the box, and exposes a web dashboard for alert triage and hunting. It runs on Ubuntu Server 22.04 LTS on top of the Elastic stack.

2 cores, 4GB RAM, 50GB disk. Two adapters: Internal Network "AttackLab" for receiving agent logs, and NAT temporarily during initial package installation. Pull the NAT adapter once Wazuh is fully configured.

## Network

All three VMs sit on an internal VirtualBox network named "AttackLab" with subnet 192.168.100.0/24. No DHCP — IPs are assigned statically. This network is fully isolated from the host and the internet.

| Machine | IP |
|---|---|
| Kali Attacker | 192.168.100.10 |
| Windows Victim | 192.168.100.20 |
| Wazuh Server | 192.168.100.30 |

NAT adapters exist on the Windows and Wazuh VMs for initial setup only. Both get disabled before any scenario runs.

## Network Diagram

```
                        Host Machine
                   Windows 11 / VirtualBox
                        48GB RAM

                   Internal Network "AttackLab"
                      192.168.100.0/24

        .100.10              .100.20              .100.30
      Kali Linux          Windows 11            Wazuh Server
       Attacker             Victim              Ubuntu 22.04
                               |                    |
                        Wazuh Agent            Wazuh Manager
                        (log shipping)         (detection + UI)
```

## Data Flow

Kali executes an attack against the Windows victim over the internal network. Sysmon captures the activity and writes to the Windows Event Log. The Wazuh agent on the victim ships those logs to the Wazuh manager on the Ubuntu server in near real time. Wazuh correlates events against its ruleset and fires alerts, which show up in the dashboard. You can also hunt manually through raw log data in the Discover view.

For scenarios where you want to work closer to the raw events, Event Viewer and Sysinternals tools on the victim are still the primary analysis surface. Wazuh is there for correlation and triage practice.

## Analysis Workflow

Run the attack from Kali. Watch live behavior on the victim using Process Explorer and TCPView. Capture detailed activity with Process Monitor. Review Sysmon event IDs in Event Viewer. Cross-reference what Wazuh fired and what it missed. Extract IOCs, document findings, restore the victim snapshot, repeat.

## Isolation and Safety

VMs cannot access the host filesystem. No shared folders, no clipboard sharing, no drag-and-drop during attack scenarios. NAT adapters are disabled before executing anything offensive. Snapshots give you instant rollback on the victim so you always have a clean baseline to return to.

## Snapshot Strategy

Take snapshots at the following points and name them with a date and description so the timeline is clear.

Fresh OS install before any configuration. Sysmon and Sysinternals installed and validated. Wazuh agent connected and shipping logs. Full baseline ready with all tools in place. Pre-scenario snapshot before each attack run.

A consistent naming pattern like `2025-06-01_Win11Victim_BaselineReady` keeps things readable when you have a dozen snapshots stacked up.

## Expanding the Lab Later

When you're ready to go beyond the current three VMs, the most useful additions in order are a Windows Server domain controller to practice lateral movement in an AD environment, a REMnux VM for malware analysis alongside Digital Forensics work, and additional victim clones to simulate multi-host propagation scenarios.
