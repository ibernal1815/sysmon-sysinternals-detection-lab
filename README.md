# Sysmon + Sysinternals Detection Lab

A hands-on Windows detection lab built to practice identifying attacker TTPs using Sysmon and the Sysinternals Suite, with each scenario mapped to MITRE ATT&CK. The goal is to document not just what the attack looks like, but exactly how a defender catches it at the event level.

This is a work-in-progress lab. Scenarios will be added as they are fully documented with real logs, event IDs, and detection logic rather than placeholder content.

## Lab Environment

The lab runs on VirtualBox with three VMs on an isolated internal network.

The victim machine is a Windows 11 Enterprise VM with Sysmon installed using the SwiftOnSecurity config as a baseline. The attacker machine is Kali Linux with standard offensive tooling. There is an optional Ubuntu ELK Stack VM for log aggregation, but most analysis is done directly in Event Viewer and Sysinternals tools to keep things close to what a SOC analyst actually works with.

Host specs are an Intel i5-14400F, 48GB DDR4, and 1TB NVMe running Windows 11 with VirtualBox 7.

## Setup

The `setup/` folder contains the Sysmon configuration XML and the VM networking guide. Start there before running any scenario.

1. Clone the repo and navigate into it.
2. Follow `setup/vm-setup-guide.md` to configure the VirtualBox internal network and provision the VMs.
3. Deploy Sysmon on the victim VM using `setup/sysmon-config.xml`.

## Scenarios

Each scenario folder will contain the attack steps, the specific Sysmon event IDs triggered, raw log output, Sysinternals detection walkthrough, and a detection rule. Nothing gets added to this list until it is fully documented.

| ID | Technique | MITRE | Status |
|----|-----------|-------|--------|
| 01 | Persistence via Registry Run Keys | T1547.001 | In progress |
| 02 | Credential Dumping via LSASS | T1003.001 | Planned |
| 03 | Lateral Movement via PsExec | T1570 | Planned |
| 04 | DLL Injection | T1055.001 | Planned |

## Repository Structure

```
setup/          lab setup guides and Sysmon config
scenarios/      attack scenarios with detection walkthroughs (in progress)
detections/     Sigma rules and Sysmon filter logic per scenario
```

## Tools Used

Detection side: Sysmon, Process Explorer, Process Monitor, Autoruns, TCPView, ProcDump, Windows Event Viewer, and Elastic Stack optionally for log search.

Attack simulation: Metasploit, Mimikatz, and custom PowerShell scripts.

## References

[Sysinternals Documentation](https://docs.microsoft.com/en-us/sysinternals/)
[SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
[MITRE ATT&CK](https://attack.mitre.org/)
[Ultimate Windows Security Event Encyclopedia](https://www.ultimatewindowssecurity.com/)

## Legal

All techniques in this lab are executed in an isolated virtual environment for educational purposes only. Do not use any of this against systems you do not own or have explicit written permission to test.
