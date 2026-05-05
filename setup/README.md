# Setup

This folder contains everything needed to build the lab from scratch. Work through the files in order and take a snapshot after each major step so you have a clean rollback point.

## Contents

`sysmon-config.xml` is the Sysmon configuration deployed on the victim machine. It's based on the SwiftOnSecurity baseline with modifications to capture process creation, network connections, registry changes, file creation events, and DLL loads.

`lab-architecture.md` covers the full network design, VM resource allocation, and how the components talk to each other.

`vm-setup-guide.md` is the step-by-step build guide covering VirtualBox network setup, Windows 11 victim configuration, Kali attacker configuration, Sysmon installation, Sysinternals deployment, and snapshot strategy.

`lab-architecture.png` is a diagram of the network topology and data flow. *(planned)*

## Requirements

You need VirtualBox 7+, a Windows 11 Enterprise ISO (evaluation builds are free from Microsoft), a Kali Linux ISO, at least 16GB of RAM available for VMs, and around 100GB of free disk space. Plan for 2 to 3 hours for a full build.

## Build Order

Follow `lab-architecture.md` first to understand the network design, then `vm-setup-guide.md` to build the VMs, then deploy `sysmon-config.xml` on the victim machine. Take a clean baseline snapshot before running any scenarios.

## Validation

Before touching any scenario, confirm the following:

The VirtualBox internal network is configured and both VMs are on the same network name. The Kali VM can ping the Windows VM. Sysmon is running (`Get-Service Sysmon64`) and events are showing up in Event Viewer, at minimum Event IDs 1, 3, and 13. Sysinternals Suite is extracted to `C:\Tools\`. Windows Defender is disabled for controlled testing. A clean baseline snapshot exists.

## Troubleshooting

If Sysmon is not logging events, check the service status, verify the config loaded, and restart if needed:

```powershell
Get-Service Sysmon64
sysmon64.exe -c
Restart-Service Sysmon64
```

If the VMs cannot communicate, make sure both are assigned to the same internal network name in VirtualBox, check Windows Firewall rules on the victim, and test with `ping` and `Test-NetConnection`.

If the host is struggling with performance, pull back the RAM allocation on one of the VMs and close anything unnecessary on the host.

## References

[VirtualBox Documentation](https://www.virtualbox.org/manual/)
[Sysmon Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
[SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
[Windows 11 Evaluation ISOs](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-11-enterprise)


