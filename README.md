# Memory Forensics Investigation

**Tools:** Volatility 3  
**Type:** Digital Forensics | Incident Response | Memory Analysis  
**Dataset:** DumpMe — CyberDefenders  

---

## Scenario

A memory dump was captured from a compromised Windows 7 host. As the DFIR analyst, the objective was to identify the malicious process, understand its capabilities, trace its execution path, and uncover the attacker's activity at the time of capture.

---

## Investigation Steps

1. Identified system profile using `windows.info`
2. Enumerated all running processes using `windows.pslist`
3. Identified suspicious process and traced parent-child relationship
4. Analyzed loaded DLLs to determine malware capabilities
5. Confirmed execution path via `windows.cmdline`
6. Discovered active C2 connection using `windows.netscan`
7. Identified file and registry access using `windows.handles`

---

## Key Findings

| Finding | Detail |
|---------|--------|
| Compromised host | Windows 7 SP1 64-bit |
| Victim user | Bob |
| Malicious process | UWkpjFjDzM.exe (PID 3496) |
| Execution path | C:\Users\Bob\AppData\Local\Temp\rad93398.tmp\ |
| Dropper | wscript.exe (PID 5116) |
| C2 connection | 10.0.0.106 port 4444 (Metasploit Meterpreter) |
| Capabilities | Network, HTTP, Encryption (WS2_32, WININET, CRYPT32) |
| Persistence | IMAGE FILE EXECUTION OPTIONS registry key |
| File access | Active handle to Bob's Desktop |
| Memory captured | 2019-03-22 05:46 UTC |

---

## Volatility Commands Used

```bash
# System identification
python vol.py -f "Triage-Memory.mem" windows.info

# Process enumeration
python vol.py -f "Triage-Memory.mem" windows.pslist

# DLL analysis
python vol.py -f "Triage-Memory.mem" windows.dlllist | findstr "3496"

# Execution path
python vol.py -f "Triage-Memory.mem" windows.cmdline | findstr "3496"

# Network connections
python vol.py -f "Triage-Memory.mem" windows.netscan | findstr "3496"

# File and registry handles
python vol.py -f "Triage-Memory.mem" windows.handles --pid 3496
```

---

## Attack Chain
wscript.exe (PID 5116)
└── Executed malicious script
└── Dropped UWkpjFjDzM.exe to Temp folder
└── Established Meterpreter session to 10.0.0.106:4444
└── Accessed Bob's Desktop files
└── Modified registry for persistence

---

## Outcome

Produced a DFIR investigation report documenting system identification, investigation methodology, technical findings, full attack chain reconstruction and indicators of compromise.
