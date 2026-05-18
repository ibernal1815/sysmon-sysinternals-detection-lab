# sysmon-detection-lab

A hands-on Windows detection lab built to practice identifying attacker TTPs at the event level using Sysmon and the Sysinternals Suite. Every scenario is mapped to MITRE ATT&CK and documented with real logs, the specific event IDs triggered, and the detection logic that catches it.

This is a work in progress. Nothing gets added to the scenario list until it is fully documented with actual log output and a working detection rule.

## Background

Reading about Sysmon event IDs is one thing. Watching a credential dump happen and then finding exactly which events fired and why is another. I wanted a lab where I could run real techniques, capture the actual logs, and build detections from what I saw rather than from documentation.

The goal is not to collect attack scripts. It is to understand what each technique looks like from the defender side well enough to write detection logic that catches it.

## Environment

Three VMs on an isolated VirtualBox internal network.

The victim is Windows 11 Enterprise with Sysmon deployed using the SwiftOnSecurity config as a baseline. The attacker is Kali Linux with standard offensive tooling. There is an optional Ubuntu ELK Stack VM for log aggregation, but most analysis is done directly in Event Viewer and Sysinternals tools to stay close to what a SOC analyst actually works with day to day.

Host specs are an Intel i5-14400F, 48GB DDR4, and 1TB NVMe running Windows 11 with VirtualBox 7.

## Setup

The setup/ folder has the Sysmon configuration XML and the VM networking guide. Start there before running any scenario.

```bash
git clone https://github.com/ibernal1815/sysmon-detection-lab.git
cd sysmon-detection-lab
```

Follow setup/vm-setup-guide.md to configure the VirtualBox internal network and provision the VMs, then deploy Sysmon on the victim using setup/sysmon-config.xml.

## Scenarios

| ID | Technique | MITRE | Status |
|----|-----------|-------|--------|
| 01 | Persistence via Registry Run Keys | T1547.001 | In progress |
| 02 | Credential Dumping via LSASS | T1003.001 | Planned |
| 03 | Lateral Movement via PsExec | T1570 | Planned |
| 04 | DLL Injection | T1055.001 | Planned |

Each scenario folder contains the attack steps, the Sysmon event IDs triggered, raw log output, a Sysinternals detection walkthrough, and a Sigma rule.

## Project Layout

```
setup/          lab setup guides and Sysmon config
scenarios/      attack scenarios with detection walkthroughs
detections/     Sigma rules and Sysmon filter logic per scenario
```

## Tools

Detection: Sysmon, Process Explorer, Process Monitor, Autoruns, TCPView, ProcDump, Windows Event Viewer, Elastic Stack optional for log search.

Attack simulation: Metasploit, Mimikatz, custom PowerShell scripts.

## References

[SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
[MITRE ATT&CK](https://attack.mitre.org/)
[Sysinternals Documentation](https://docs.microsoft.com/en-us/sysinternals/)
[Ultimate Windows Security Event Encyclopedia](https://www.ultimatewindowssecurity.com/)

## Stack

Sysmon · Sysinternals Suite · Windows Event Viewer · Elastic Stack · Sigma · Kali Linux · VirtualBox

## Legal

All techniques in this lab are executed in an isolated virtual environment for educational purposes only. Do not use any of this against systems you do not own or have explicit written permission to test.
