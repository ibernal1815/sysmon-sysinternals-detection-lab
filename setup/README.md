# Lab Setup Guide

This directory contains everything you need to build the Sysmon + Sysinternals Detection Lab from scratch.

## 📋 Overview

Setting up this lab takes approximately **2-3 hours** and requires:
- VirtualBox installed on your host machine
- Windows 10/11 ISO (evaluation version available from Microsoft)
- Kali Linux ISO (free download)
- 16GB+ available RAM
- 100GB+ free disk space

## 🗂️ Contents

### `lab-architecture.png`
Visual diagram showing the complete network topology, VM layout, and data flow between components.

### `lab-architecture.md`
Detailed documentation of:
- Network configuration (internal networks, NAT, host-only adapters)
- VM specifications and resource allocation
- Component interactions and data flow
- Security considerations for isolated testing

### `vm-setup-guide.md`
Step-by-step instructions for:
- VirtualBox network configuration
- Windows 10 victim machine setup
- Kali Linux attacker machine setup
- Sysmon installation and configuration
- Sysinternals Suite deployment
- Creating VM snapshots for easy rollback

### `sysmon-config.xml`
Production-ready Sysmon configuration file based on SwiftOnSecurity's config with custom modifications for:
- Process creation monitoring
- Network connection tracking
- Registry modification detection
- File creation events
- Image/DLL load monitoring

## 🚀 Quick Start Path

If you're setting up the lab for the first time, follow this order:

1. **Read `lab-architecture.md`** - Understand the network design
2. **Follow `vm-setup-guide.md`** - Build VMs step-by-step
3. **Deploy `sysmon-config.xml`** - Install monitoring on victim machine
4. **Take snapshots** - Save clean baseline states
5. **Run validation tests** - Verify Sysmon is logging correctly

## ✅ Validation Checklist

Before running attack scenarios, verify:

- [ ] VirtualBox internal network configured
- [ ] Windows 10 VM can access internet (for updates)
- [ ] Kali VM can ping Windows VM on internal network
- [ ] Sysmon is running (`Get-Service Sysmon64`)
- [ ] Sysmon events appearing in Event Viewer (Event ID 1, 3, 13, etc.)
- [ ] Sysinternals Suite extracted to `C:\Tools\`
- [ ] Windows Defender disabled (for controlled testing)
- [ ] Clean baseline snapshot created

## 🔧 Troubleshooting

**Sysmon not logging events:**
```powershell
# Check service status
Get-Service Sysmon64

# Verify configuration loaded
sysmon64.exe -c

# Restart service
Restart-Service Sysmon64
```

**VMs can't communicate:**
- Verify both VMs on same internal network name
- Check Windows Firewall rules
- Test with `ping` and `Test-NetConnection`

**Performance issues:**
- Reduce VM RAM allocation if host struggling
- Disable unnecessary Windows services
- Close other applications on host

## 📚 Additional Resources

- [VirtualBox Documentation](https://www.virtualbox.org/manual/)
- [Sysmon Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config)
- [Windows 10 Evaluation ISOs](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise)

## 💡 Pro Tips

- **Snapshot early, snapshot often** - Take snapshots before each major change
- **Name snapshots descriptively** - "Clean Install", "Sysmon Configured", "Pre-Attack Baseline"
- **Document customizations** - Keep notes on any config changes you make
- **Test incrementally** - Verify each component works before moving to next step
- **Keep ISOs** - Save original installation media for rebuilding if needed

## 🎯 Next Steps

Once your lab is fully set up and validated:
1. Navigate to `../scenarios/01-persistence-registry/`
2. Follow the attack scenario walkthrough
3. Practice detection with Sysinternals tools
4. Compare your findings with documented results

---

**Need help?** Open an issue on GitHub or check the troubleshooting section in the main README.
