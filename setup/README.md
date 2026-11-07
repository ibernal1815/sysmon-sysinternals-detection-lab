# Environment Provisioning & Setup

This directory contains the necessary documentation and configuration files to provision the detection laboratory. It is highly recommended to capture hypervisor snapshots at each major configuration milestone to establish clean rollback baselines.

## Hardware & Software Requirements

| Component | Specification |
|---|---|
| **Hypervisor** | Oracle VM VirtualBox 7.0+ |
| **Host Resource Allocation** | ≥ 16GB RAM available, 100GB Free Disk Space |
| **Target OS (Victim)** | Windows 11 Enterprise (Evaluation ISO) |
| **Adversary OS (Attacker)** | Kali Linux ISO |
| **Estimated Build Time** | 2 - 3 Hours |

## Directory Contents

*   **`lab-architecture.md`** - Details the logical network topology, virtual machine resource allocation, and inter-node communication paths.
*   **`vm-setup-guide.md`** - Step-by-step provisioning instructions covering VirtualBox network configuration, OS deployment, Sysmon installation, and Sysinternals staging.
*   **`sysmon-config.xml`** - Custom Sysmon configuration file deployed on the Windows target. Based on the SwiftOnSecurity baseline, optimized for Process Creation (ID 1), Network Connections (ID 3), Registry Events (ID 12-14), File Creation (ID 11), and Image Loads (ID 7).
*   **`lab-architecture.png`** - Visual diagram of the network topology and data flow *(Pending)*.

## Provisioning Sequence

1.  **Architecture Review:** Review `lab-architecture.md` to establish the required virtual network adapters and routing configuration.
2.  **OS Deployment:** Execute the build instructions in `vm-setup-guide.md` to deploy both the Windows 11 and Kali Linux virtual machines.
3.  **Telemetry Configuration:** Deploy Sysmon on the Windows 11 target using the provided `sysmon-config.xml` file.
4.  **Baseline Snapshot:** Capture a clean hypervisor snapshot prior to executing any simulated scenarios.

## Pre-Flight Validation Criteria

Before initiating any detection scenarios, verify the following baseline conditions are met:

*   [ ] The VirtualBox internal network is active and both VMs reside on the same network interface name.
*   [ ] ICMP Echo Requests (ping) succeed from the Kali VM to the Windows VM.
*   [ ] Windows Defender is permanently disabled via Safe Mode registry modifications (Start keys for `WinDefend`, `WdNisSvc`, and `Sense` set to `4`) to prevent automated quarantine of testing tools.
*   [ ] Windows Sysinternals Suite is extracted to `C:\Tools\`.
*   [ ] The Sysmon service is in a running state.
*   [ ] Windows Event Viewer (`Applications and Services Logs > Microsoft > Windows > Sysmon > Operational`) is actively populating with Event IDs 1, 3, and 13.

## Troubleshooting Support

**Sysmon Telemetry Missing**
If Sysmon fails to log events, verify the service status, reload the configuration XML, and restart the service via an elevated PowerShell prompt:

```powershell
Get-Service Sysmon64
sysmon64.exe -c
Restart-Service Sysmon64
```

**Network Isolation Issues**
If the virtual machines cannot communicate:
*   Verify both VMs are assigned to the identical internal network name within VirtualBox adapter settings.
*   Verify Windows Defender Firewall inbound rules on the Windows target are not dropping ICMP traffic. 
*   Validate connectivity using `Test-NetConnection -ComputerName <Target_IP>`.

**Host Performance Degradation**
If the physical host experiences latency, reduce the RAM allocation on the Kali VM or suspend unnecessary host background processes during simulation runs.

## References

*   [VirtualBox Documentation](https://www.virtualbox.org/manual/)
*   [Windows 11 Evaluation Media](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)
*   [Sysmon Official Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
*   [SwiftOnSecurity Sysmon Baseline](https://github.com/SwiftOnSecurity/sysmon-config)
