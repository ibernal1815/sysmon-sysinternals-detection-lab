# Lab Architecture

##  Overview

This detection lab uses an isolated virtual environment to safely simulate attacks and practice detection techniques. The architecture prioritizes security isolation while maintaining flexibility for logging and analysis.

##  Host System Specifications

**Hardware:**
- **CPU:** Intel Core i5-14400F (10 cores, 16 threads)
- **RAM:** 48GB DDR4
- **Storage:** 1TB NVMe SSD
- **Network:** 2.5Gb Ethernet LAN
- **GPU:** AMD Radeon RX 7600 (not used for VMs)

**Host OS:**
- Windows 11 Pro (64-bit)
- VirtualBox 7.0+

**Resource Allocation:**
- VMs: ~20GB RAM total
- Host OS: ~28GB RAM remaining
- Adequate overhead for smooth operation

##  Virtual Machine Layout

### Windows 11 Victim Machine
**Purpose:** Target system instrumented for detection and analysis

**Specifications:**
- **RAM:** 6GB
- **CPU:** 4 cores
- **Disk:** 60GB (dynamically allocated)
- **OS:** Windows 11 Pro 22H2 (or Windows 11 Enterprise)
- **Network Adapters:**
  - Adapter 1: NAT (for internet access during setup)
  - Adapter 2: Internal Network "AttackLab"

**Installed Software:**
- Sysmon 15.0+ with custom configuration
- Sysinternals Suite (full installation)
- .NET Framework 4.8+
- PowerShell 5.1+
- Optional: Chrome, 7-Zip, Notepad++

**Security Posture:**
- Windows Defender **DISABLED** (for controlled attack testing)
- Windows Firewall **ENABLED** (can be toggled per scenario)
- UAC set to default
- Windows Update paused during testing
- Local administrator account enabled

---

### Kali Linux Attacker Machine
**Purpose:** Offensive security platform for attack simulation

**Specifications:**
- **RAM:** 4GB
- **CPU:** 2 cores
- **Disk:** 40GB (dynamically allocated)
- **OS:** Kali Linux 2024.x (latest)
- **Network Adapter:**
  - Adapter 1: Internal Network "AttackLab"

**Installed Tools:**
- Metasploit Framework
- Nmap
- Impacket suite (PsExec, WMIExec, etc.)
- Responder
- Mimikatz (compiled for Windows)
- Custom PowerShell payloads
- netcat / socat
- Evil-WinRM

**Network Configuration:**
- Static IP: 192.168.100.10/24
- No internet access (air-gapped for safety)
- Can reach Windows victim at 192.168.100.20

---

### Optional: Ubuntu ELK Stack Server
**Purpose:** Centralized log collection and analysis

**Specifications:**
- **RAM:** 4GB
- **CPU:** 2 cores
- **Disk:** 50GB
- **OS:** Ubuntu Server 22.04 LTS
- **Network Adapters:**
  - Adapter 1: Internal Network "AttackLab"
  - Adapter 2: NAT (for package downloads)

**Stack Components:**
- Elasticsearch 8.x
- Logstash 8.x
- Kibana 8.x
- Winlogbeat (installed on Windows victim)

**Note:** This VM is optional. You can analyze Sysmon logs directly on the victim machine using Event Viewer or export to your host for analysis.

##  Network Architecture

### Network Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                         Host Machine                            │
│                    Windows 11 - i5-14400F                       │
│                         48GB RAM                                │
└────────────┬────────────────────────────────────────────────────┘
             │
             │ VirtualBox Hypervisor
             │
    ┌────────┴────────┬──────────────────┬──────────────────────┐
    │                 │                  │                      │
┌───▼────────┐   ┌────▼─────────┐   ┌───▼──────────┐   ┌──────▼──────┐
│    NAT     │   │  Internal    │   │ Host-Only    │   │   Bridge    │
│  Network   │   │  "AttackLab" │   │   Adapter    │   │  (unused)   │
└────────────┘   └──────┬───────┘   └──────────────┘   └─────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌────▼─────┐    ┌───▼─────┐    ┌───▼──────┐
   │ Windows  │    │  Kali   │    │   ELK    │
   │  Victim  │    │ Attacker│    │  Stack   │
   │          │    │         │    │ (Optional)│
   │ .100.20  │    │ .100.10 │    │ .100.30  │
   └──────────┘    └─────────┘    └──────────┘
```

### Network Configuration Details

**Internal Network "AttackLab"**
- **Type:** Internal Network (isolated from host and internet)
- **Subnet:** 192.168.100.0/24
- **No DHCP:** Static IPs assigned manually
- **Purpose:** Isolated attack simulation environment

**IP Address Assignments:**
| Host | IP Address | Purpose |
|------|------------|---------|
| Windows Victim | 192.168.100.20 | Target machine |
| Kali Attacker | 192.168.100.10 | Attack platform |
| ELK Stack | 192.168.100.30 | Log aggregation (optional) |

**NAT Network (Temporary)**
- **Purpose:** Initial VM setup, Windows updates, tool downloads
- **When to use:** During VM configuration phase only
- **Security:** Disable NAT adapter before running attack scenarios

##  Data Flow

### Attack Execution Flow
```
Kali Attacker (192.168.100.10)
         │
         │ Execute attack (network/local)
         ▼
Windows Victim (192.168.100.20)
         │
         │ Sysmon captures events
         ▼
Windows Event Log (Microsoft-Windows-Sysmon/Operational)
         │
         │ (Optional) Winlogbeat forwards
         ▼
ELK Stack (192.168.100.30)
         │
         │ Parse, index, visualize
         ▼
Kibana Dashboard
```

### Analysis Workflow
```
1. Execute attack from Kali
2. Observe behavior on Windows (Process Explorer, TCPView)
3. Capture activity with Process Monitor
4. Review Sysmon logs in Event Viewer
5. Correlate events to reconstruct attack timeline
6. Extract IOCs and document findings
7. Restore VM to clean snapshot
8. Repeat with variations
```

##  Security Considerations

### Isolation Strategy
- **No outbound internet** from attack network during active testing
- **Air-gapped environment** prevents accidental malware escape
- **Host-only management** access for VM administration
- **Snapshots** allow instant rollback to clean state

### Attack Containment
- VMs cannot access host filesystem directly
- No shared folders enabled during attack scenarios
- Clipboard sharing disabled
- Drag-and-drop disabled

### Safe Testing Practices
1. **Always work from snapshots** - Never run attacks on production systems
2. **Disconnect NAT** before executing malware or exploits
3. **Verify network isolation** before each attack scenario
4. **Document everything** - Maintain lab notebook with timestamps
5. **Clean up thoroughly** - Restore to baseline between scenarios

##  Resource Monitoring

**Recommended Host Resources During Lab Use:**
- **Available RAM:** 28GB+ for smooth operation
- **CPU Usage:** Monitor with Task Manager; pause VMs if host slows
- **Disk I/O:** NVMe handles VM operations well; avoid HDD if possible
- **Network:** Internal networks have no bandwidth constraints

**VM Performance Tips:**
- Allocate RAM in 2GB increments for efficiency
- Use fixed-size disks if storage isn't constrained (faster I/O)
- Enable VT-x/AMD-V and nested paging in BIOS
- Disable unnecessary Windows services in victim VM

##  Snapshot Strategy

**Recommended Snapshots:**

1. **"Fresh Install"** - Clean OS install before any configuration
2. **"Sysmon Configured"** - Sysmon installed with proper config
3. **"Baseline Ready"** - All tools installed, ready for attacks
4. **"Pre-Scenario-X"** - Before each attack scenario
5. **"Post-Scenario-X"** - After successful attack for comparison

**Snapshot Naming Convention:**
```
YYYY-MM-DD_VMName_Description
Example: 2024-02-15_Win10Victim_BaselineReady
```

##  Scalability Options

### Expanding the Lab

**Active Directory Domain:**
- Add Windows Server 2019/2022 as Domain Controller
- Join victim as domain member
- Practice lateral movement between domain-joined systems

**Multiple Victims:**
- Clone Windows victim for multi-host scenarios
- Simulate network propagation attacks
- Test detection at scale

**SIEM Integration:**
- Forward Sysmon logs to Wazuh
- Create custom detection rules
- Build correlation alerts

**Malware Analysis Environment:**
- Add REMnux VM for malware analysis
- Integrate with Cuckoo Sandbox
- Capture full PCAP for traffic analysis

##  Documentation Standards

Every configuration change should be documented:
- What was changed and why
- Configuration file locations
- Commands executed
- Expected vs actual behavior
- Troubleshooting steps taken

This ensures reproducibility and helps others learn from your work.

---

**Next Step:** Follow `vm-setup-guide.md` to build this architecture from scratch.


