
# Tempest DFIR Investigation

A hands-on **Digital Forensics & Incident Response (DFIR)** case study documenting the investigation of a simulated Windows compromise.

The objective of this investigation was to reconstruct the complete attack chain from the provided forensic artifacts, identify the attacker's tactics, techniques, and procedures (TTPs), and produce a structured Incident Response report following DFIR best practices.

> **Note**
> 
> This repository contains a laboratory investigation performed in a controlled training environment. It does **not** represent a real-world security incident.

---

## Investigation Objectives

- Reconstruct the complete attack timeline.
    
- Identify the initial infection vector.
    
- Analyze the exploitation and execution chain.
    
- Track payload delivery and Command & Control (C2) activity.
    
- Identify persistence mechanisms.
    
- Investigate privilege escalation and post-exploitation activity.
    
- Extract Indicators of Compromise (IoCs).
    
- Map attacker behavior to the MITRE ATT&CK framework.
    

---

## Investigation Artifacts

The investigation was performed using the following artifacts:

- Sysmon Event Logs (`Sysmon.evtx`)
    
- Parsed Sysmon Logs (`Sysmon.csv`)
    
- Windows Event Logs (`Windows.evtx`)
    
- Network Packet Capture (`traffic.pcap`)


---

## Tools Used

- Timeline Explorer
- SysmonView
- Wireshark
- Brim
- EvtxECmd
- VirusTotal

---

## Investigation Workflow

```Mathematica
User
 ↓
Downloads malicious Word document
 ↓
Word executes PowerShell
 ↓
PowerShell downloads Stage 2 payload
 ↓
Stage 2 establishes persistence
 ↓
Stage 2 contacts C2 server
 ↓
Attacker executes discovery commands
 ↓
Attacker deploys reverse SOCKS proxy
 ↓
Privilege Escalation occurs
 ↓
Attacker gains SYSTEM
 ↓
Additional persistence mechanisms created
 ↓
Machine fully compromised
```

---

## Report Contents

- Incident Scenario
    
- Incident Architecture
    
- Investigation Methodology
    
- Initial Access Analysis
    
- Execution Analysis
    
- Exploitation Analysis
    
- Payload Delivery
    
- Stage 2 Activity
    
- Network Analysis
    
- Discovery
    
- Persistence
    
- Remote Access
    
- Privilege Escalation
    
- Post-Exploitation
    
- Impact & Scope
    
- Indicators of Compromise (IoCs)
    
- MITRE ATT&CK Mapping
    
- Attack Timeline
    
- Incident Conclusion
    

---

## Attack Summary

The investigation determined that the attacker compromised the workstation through a malicious Microsoft Word document exploiting **CVE-2022-30190 (Follina)**. The exploit executed an embedded PowerShell command that downloaded additional payloads, established persistence through the Startup folder and a Windows Service, deployed a reverse SOCKS proxy for remote access, escalated privileges to **NT AUTHORITY\SYSTEM**, and created additional administrative accounts to maintain long-term access.

---

## MITRE ATT&CK Highlights

- Initial Access
    
- User Execution
    
- Exploitation for Client Execution
    
- PowerShell
    
- Ingress Tool Transfer
    
- Startup Folder Persistence
    
- Windows Service Persistence
    
- System Discovery
    
- Account Discovery
    
- Proxy
    
- Privilege Escalation
    
- Windows Remote Management (WinRM)
    
- Account Manipulation
    

---

## Repository Structure

```text
.
├── Tempest2_Report.pdf
├── README.md
├── Artifacts/
│   ├── Sysmon.evtx
│   ├── Sysmon.csv
│   ├── Windows.evtx
│   └── traffic.pcap
├── IOC_List.md
├── Timeline/
└── Screenshots/
```

---

## Skills Demonstrated

- Digital Forensics
    
- Incident Response
    
- Windows Event Log Analysis
    
- Sysmon Analysis
    
- PCAP Analysis
    
- Process Tree Reconstruction
    
- Attack Timeline Reconstruction
    
- Threat Hunting
    
- MITRE ATT&CK Mapping
    
- IOC Extraction
    
- DFIR Reporting
    

---

## License

This repository is published for **educational and portfolio purposes only**. All artifacts originate from a controlled DFIR laboratory environment and should be used exclusively for learning and research.
