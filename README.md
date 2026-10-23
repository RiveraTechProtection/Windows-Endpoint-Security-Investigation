
# Windows Endpoint Security Investigation

### Windows 11 | Event Viewer | PowerShell | Security Auditing

**Analyst:** Marcos Rivera  
**Portfolio:** Rivera Tech Protection  
**Project Type:** Controlled Windows Endpoint Security Lab

## Project Overview

This project documents a hands-on Windows endpoint investigation performed in a Windows 11 virtual machine running through Parallels on macOS.

I configured Windows security auditing, created a local test account named `FinanceUser`, simulated suspicious authentication and discovery activity, and investigated the resulting telemetry.

A key finding was a visibility gap in PowerShell monitoring. I addressed this by enabling PowerShell Script Block Logging and validating the improvement through Windows Event ID 4104.

> **Lab Disclaimer:** All suspicious activity was generated in a controlled training environment. This project does not represent an actual security breach.

## Environment and Tools

- Windows 11 virtual machine
- Parallels Desktop on macOS
- Windows Event Viewer
- Local Security Policy
- Advanced Audit Policy
- Windows PowerShell
- Command Prompt
- PowerShell Script Block Logging

## Investigation Objectives

1. Generate controlled suspicious authentication activity.
2. Analyze Windows Security Event logs.
3. Reconstruct process creation and parent-child relationships.
4. Identify reconnaissance commands.
5. Investigate PowerShell execution.
6. Identify and remediate endpoint logging limitations.
7. Validate the effectiveness of improved security telemetry.

## Windows Event IDs Investigated

| Event ID | Description |
|---|---|
| 4625 | Failed logon attempt |
| 4648 | Logon attempted using explicit credentials |
| 4688 | New process created |
| 4104 | PowerShell Script Block Logging |

## Phase 1: Authentication Investigation

The investigation began by examining authentication activity associated with the local `FinanceUser` test account.

**Event ID 4625** documented a failed interactive logon attempt (Logon Type 2).

**Event ID 4648** captured the primary Windows account using explicit credentials for `FinanceUser` against localhost.

### Analyst Finding

The evidence demonstrated the controlled use of alternate credentials and allowed authentication activity to be correlated with subsequent process execution.

Different authentication methods can produce different event patterns, making context essential during SOC investigations.

## Phase 2: Process Creation and Discovery

Windows Security Event ID 4688 recorded process execution associated with `FinanceUser`.

Observed discovery commands included:

- `whoami.exe`
- `hostname.exe`
- `ipconfig.exe`

The event logs also confirmed the following process chain:

```text
FinanceUser
    |
    v
cmd.exe
    |
    v
powershell.exe
    |
    v
whoami.exe
```

The PowerShell process was launched at approximately 12:46:46 AM, followed by `whoami.exe` at approximately 12:46:59 AM.

### Analyst Finding

Although these are legitimate Windows utilities, their combined execution can be consistent with post-authentication reconnaissance.

The controlled lab activity was analyzed as suspicious behavior rather than evidence of a real compromise.

## Phase 3: Identifying a PowerShell Logging Gap

Initial Windows Security and PowerShell Operational logs confirmed that PowerShell executed.

However, process creation auditing did not capture every native PowerShell command executed within the existing PowerShell process.

Examples included:

```powershell
Get-Process
Get-Content C:\SecurityLab\Quarterly_Financials.txt
```

These commands did not generate separate Event ID 4688 records because they executed inside the existing PowerShell process.

### Security Finding

**Process creation auditing alone was insufficient to reconstruct all PowerShell activity.**

This represented an endpoint visibility limitation that could affect security investigations.

## Phase 4: Security Logging Improvement

To address the identified visibility gap, I enabled PowerShell Script Block Logging through Group Policy.

I then repeated controlled PowerShell commands and reviewed the resulting events.

**Event ID 4104** successfully captured:

- `Get-Process`
- `Get-Content C:\SecurityLab\Quarterly_Financials.txt`

### Validation Result

The additional telemetry confirmed that the logging improvement increased visibility into native PowerShell commands.

This demonstrated a complete workflow:

**Identify visibility gap → Improve logging → Generate test activity → Verify telemetry**

## Evidence Timeline

| Time | Evidence |
|---|---|
| 12:45:11 AM | Event 4648 recorded explicit credential use |
| 12:46 AM | Event 4688 recorded discovery-related processes |
| 12:46:46 AM | PowerShell launched from Command Prompt |
| 12:46:59 AM | `whoami.exe` launched from PowerShell |
| 12:46 AM | `hostname.exe` and `ipconfig.exe` executed |
| 1:10:44 AM | Event 4625 recorded failed interactive logon |
| 1:42 AM | Event 4104 confirmed script block logging |

## Security Recommendations

- Maintain process creation auditing with command-line capture.
- Enable PowerShell Script Block Logging where appropriate.
- Correlate authentication and process creation activity.
- Investigate unusual discovery commands following suspicious authentication.
- Monitor unexpected alternate-credential usage.
- Centralize endpoint telemetry in a SIEM.

## Skills Demonstrated

- Windows Event Log Analysis
- Security Audit Policy Configuration
- Authentication Event Correlation
- Process Creation Investigation
- Parent-Child Process Reconstruction
- PowerShell Security Monitoring
- Logging Gap Identification
- Endpoint Visibility Improvements
- Security Control Validation
- SOC Incident Documentation

## Conclusion

This project demonstrates the ability to configure a Windows endpoint security lab, generate controlled suspicious activity, and investigate authentication and process execution evidence.

The most significant outcome was identifying a PowerShell monitoring limitation and improving security visibility through Script Block Logging.

By validating Event ID 4104 after making the configuration change, I demonstrated an evidence-based approach to endpoint monitoring and security control verification.

**Prepared by Marcos Rivera | Rivera Tech Protection**
