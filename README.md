# Sysmon & Sysinternals Detection Lab

![Build Status](https://img.shields.io/badge/Status-Active_Development-blue)
![Focus](https://img.shields.io/badge/Focus-Detection_Engineering-red)
![Platform](https://img.shields.io/badge/Environment-VirtualBox-lightgrey)

## Project Objective
This repository serves as a dedicated detection engineering laboratory focused on generating, capturing, and analyzing telemetry from Windows Sysinternals tools. The primary goal is to establish baseline benign activity, simulate the malicious misuse of administrative utilities, and develop high-fidelity detection logic utilizing Sysmon and centralized log aggregation.

## Logical Architecture
The laboratory is built on a type-2 hypervisor with isolated networking to ensure a secure and controlled testing environment. 

* **Hypervisor:** Oracle VM VirtualBox
* **Host Operating System:** Windows 11 Pro
* **Network Configuration:** Host-Only Adapter (Isolated subnet)
* **Target Endpoints:** Windows Client / Windows Server VMs

## Technologies Utilized
* **Endpoint Telemetry:** Sysmon (System Monitor)
* **Execution Tools:** Windows Sysinternals Suite (PsExec, ProcDump, Autoruns, SDelete)
* **Log Aggregation:** [Insert SIEM, e.g., Elastic Stack / Wazuh / Windows Event Viewer]
* **Automation:** PowerShell

## MITRE ATT&CK Mapping
The simulation scenarios developed in this laboratory map directly to the MITRE ATT&CK framework, focusing specifically on how adversaries leverage legitimate system administration tools ("Living off the Land") for post-exploitation operations.

| Tactic | Technique ID | Technique Name | Tool Analyzed |
|---|---|---|---|
| Credential Access | T1003.001 | OS Credential Dumping: LSASS Memory | ProcDump |
| Execution | T1569.002 | System Services: Service Execution | PsExec |
| Persistence | T1547.001 | Boot or Logon Autostart Execution | Autoruns |
| Defense Evasion | T1070.001 | Indicator Removal: Clear Windows Event Logs | SDelete / PsExec |

## Repository Structure
* `README.md` - Project overview, objectives, and architecture topology.
* `SETUP.md` - Environment provisioning, network isolation configuration, and Sysmon baseline deployment.
* `SCENARIOS.md` - Detailed attack execution syntax, expected Sysmon Event IDs, and subsequent detection queries.
* `assets/` - Network topology diagrams and cropped telemetry evidence.

## Project Roadmap
- [x] Establish initial VirtualBox lab environment and network isolation.
- [ ] Deploy and configure Sysmon with baseline XML ruleset.
- [ ] Document configuration parameters in `SETUP.md`.
- [ ] Execute baseline ProcDump LSASS extraction and document Event ID 10 / Event ID 11 telemetry.
- [ ] Execute baseline PsExec lateral movement and document Event ID 1 / Event ID 3 telemetry.
- [ ] Develop and validate SIEM detection queries for ProcDump and PsExec scenarios.
- [ ] Expand testing repository to include Autoruns and Procexp misuse.
