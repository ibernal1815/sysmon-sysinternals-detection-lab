# Sysmon + Sysinternals Detection Lab

A practical threat detection lab demonstrating Windows attack techniques and their detection using Sysmon and Sysinternals Suite. This project showcases real-world incident response and threat hunting methodologies in a controlled virtualized environment.

## 🎯 Project Overview

This repository documents a hands-on security lab where common attack techniques are executed and detected using:
- **Sysmon** - Advanced system activity monitoring
- **Sysinternals Suite** - Process Explorer, Process Monitor, Autoruns, TCPView, and more
- **MITRE ATT&CK Framework** - Industry-standard attack technique mapping

Each scenario includes step-by-step attack execution, detection methodology, and forensic analysis showing exactly how defenders can identify and respond to threats.

## 🏗️ Lab Architecture

**Host System:**
- CPU: Intel i5-14400F
- RAM: 48GB DDR4
- Storage: 1TB NVMe
- Network: 2.5Gb Ethernet
- Hypervisor: VirtualBox on Windows 11

**Virtual Machines:**
- **Windows 10 Victim** (6GB RAM) - Instrumented with Sysmon, target for attacks
- **Kali Linux Attacker** (4GB RAM) - Offensive tooling and exploit execution
- **Ubuntu ELK Stack** (4GB RAM) - *Optional* - Log aggregation and analysis

All VMs run on an isolated internal network for safe attack simulation.

## 🔬 Attack Scenarios

### Implemented
1. **Persistence via Registry Run Keys** - MITRE T1547.001
2. **Credential Dumping (LSASS)** - MITRE T1003.001
3. **Lateral Movement via PsExec** - MITRE T1570
4. **Process Injection (DLL)** - MITRE T1055.001

Each scenario includes:
- ✅ Attack execution steps
- ✅ Sysmon event analysis
- ✅ Sysinternals detection walkthrough
- ✅ Screenshots and event logs
- ✅ IOCs (Indicators of Compromise)

## 🛠️ Tools & Technologies

**Detection & Analysis:**
- Sysmon (SwiftOnSecurity config baseline)
- Process Explorer
- Process Monitor
- Autoruns
- TCPView
- ProcDump

**Attack Simulation:**
- Metasploit Framework
- PowerShell Empire
- Mimikatz
- Custom PowerShell scripts

**Log Analysis:**
- Windows Event Viewer
- Elastic Stack (optional)
- Custom PowerShell parsing scripts

## 🚀 Quick Start

### Prerequisites
- VirtualBox 7.0+
- Windows 10/11 ISO
- Kali Linux ISO
- 16GB+ RAM available for VMs
- Basic PowerShell knowledge

### Setup Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/sysmon-sysinternals-detection-lab.git
   cd sysmon-sysinternals-detection-lab
   ```

2. **Build the lab environment**
   - Follow the guide in `setup/vm-setup-guide.md`
   - Configure internal network in VirtualBox
   - Install Sysmon using `setup/sysmon-config.xml`

3. **Run a scenario**
   - Navigate to any scenario folder (e.g., `scenarios/01-persistence-registry/`)
   - Follow the attack steps in the README
   - Use Sysinternals tools to detect the activity
   - Compare your findings with documented results

4. **Create baseline (recommended)**
   ```powershell
   .\tools\baseline-autoruns.ps1
   ```

## 📚 Learning Outcomes

By working through this lab, you'll gain practical experience with:

- **Threat Detection:** Identifying malicious activity using Sysmon event correlation
- **Forensic Analysis:** Using Sysinternals to investigate running processes, network connections, and persistence mechanisms
- **Incident Response:** Building investigation timelines from attack artifacts
- **PowerShell Automation:** Creating scripts for baseline comparison and threat hunting
- **MITRE ATT&CK Mapping:** Understanding how real attacks map to the framework

## 📂 Repository Structure

```
├── setup/               # Lab setup guides and Sysmon configuration
├── scenarios/           # Attack scenarios with detection walkthroughs
├── tools/               # PowerShell scripts for automation and hunting
└── resources/           # Cheat sheets and reference materials
```

## 🎓 Use Cases

- **Portfolio Project** - Demonstrate practical security skills to employers
- **Interview Preparation** - Hands-on experience discussing real attack scenarios
- **SOC Training** - Learn detection techniques before starting analyst roles
- **Certification Study** - Practical lab for CySA+, Security+, or GCFA preparation
- **Home Lab Learning** - Self-paced security research and skill development

## ⚠️ Legal Disclaimer

This lab is for **educational purposes only**. All attack techniques are performed in an isolated virtual environment. Do not use these techniques against systems you do not own or have explicit permission to test.

## 🤝 Contributing

Found a new detection technique or want to add a scenario? Contributions are welcome!

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-scenario`)
3. Commit your changes (`git commit -m 'Add new lateral movement scenario'`)
4. Push to the branch (`git push origin feature/new-scenario`)
5. Open a Pull Request

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Connect

**Isaiah** - Cloud Windows Systems Administrator & Security Researcher

- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your LinkedIn](https://linkedin.com/in/yourprofile)
- Portfolio: [yourportfolio.com](https://yourportfolio.com)

## 📖 References & Further Reading

- [Sysinternals Documentation](https://docs.microsoft.com/en-us/sysinternals/)
- [Sysmon Configuration Reference](https://github.com/SwiftOnSecurity/sysmon-config)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [Windows Security Log Encyclopedia](https://www.ultimatewindowssecurity.com/)

---

⭐ **If you find this project helpful, please consider giving it a star!**