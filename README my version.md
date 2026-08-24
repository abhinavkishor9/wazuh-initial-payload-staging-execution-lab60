# wazuh-initial-payload-staging-execution-lab60
## Overview

In many real attacks, the suspicious activity does not begin with execution alone. There is often a chain of events before the payload runs.

Payload arrives
      ↓
Payload is written to disk
      ↓
Payload is copied or moved
      ↓
Payload is staged in another location
      ↓
Payload is executed
      ↓
Post-execution activity occurs

A SOC analyst may receive an alert at the execution stage, but the investigation should go backwards and forwards:

Backwards: Where did this file come from?
During execution: Which process launched it and with what command line?
Forwards: What did the executed payload do?
Initial payload

The initial payload is the file or script that begins the activity.

Examples in real investigations include:

A downloaded script.
A file received through email.
A file extracted from an archive.
A script dropped by another process.
A payload copied from a removable drive.

For this lab, we will create a benign PowerShell script that simulates an initial payload.

The important investigation point is not whether the script is malicious. It is learning how to reconstruct the activity.

Payload staging

Staging occurs when a file is placed somewhere before execution.

For example:

C:\Users\User\Downloads\payload.ps1
                ↓
C:\ProgramData\StagedPayload\update.ps1

The attacker may:

Copy the payload.
Rename it.
Move it to another directory.
Hide it among legitimate-looking files.
Prepare it for execution later.

A suspicious file name alone is not enough to determine maliciousness. The analyst should compare:

Original path.
Staging path.
Creation time.
Last write time.
File hash.
File contents.
Process responsible for the staging activity.
Execution

Execution occurs when a process launches the staged payload.

For example:

powershell.exe
      ↓
C:\InitialPayloadLab\Staging\system-update.ps1

The analyst should investigate:

Executable used.
Command line.
Parent process.
Process ID.
Process GUID.
User context.
Execution time.
Why this investigation matters

If an analyst only sees:

powershell.exe executed

that is incomplete.

A stronger investigation reconstructs:

Initial payload created
        ↓
Payload staged
        ↓
Staged file executed
        ↓
Execution artifact created
        ↓
Relevant telemetry reviewed

This gives context to the alert and helps determine whether the activity is:

Benign.
Suspicious.
Malicious.
Insufficient evidence.



This lab investigates a simulated PowerShell payload lifecycle on a Windows 11 endpoint monitored by Wazuh and Sysmon.

The investigation follows the payload through four stages:

1. Initial payload creation
2. Copying the payload into a staging directory
3. Hash-based integrity verification
4. Execution and post-execution artifact collection

The lab is designed as a DFIR and SOC investigation. The payload is intentionally benign and records basic execution information such as the timestamp, hostname, and user.

## Scenario

A SOC analyst identifies a PowerShell script on a Windows endpoint that was created in one directory and later copied to a staging location under a different filename. The staged script is executed using PowerShell with -ExecutionPolicy Bypass, creating a need to understand the file's movement, integrity, and execution context.

The investigation focuses on:

- Tracing the payload from its original location to the staging directory.
- Comparing file metadata and SHA256 hashes.
- Examining the PowerShell execution method.
- Identifying the execution host and user.
- Correlating Wazuh FIM and Sysmon telemetry.
- Determining whether the activity represents malicious behavior or controlled lab activity.


## Investigation Objectives

The investigation aims to:

- Establish a baseline of the Windows endpoint before the simulated activity.
- Track how a suspicious script moves between directories.
- Compare file metadata across the different stages.
- Confirm whether the staged script was altered.
- Examine the use of PowerShell execution-policy bypass.
- Identify the account and system involved in script execution.
- Validate execution through generated artifacts.
- Correlate endpoint activity with Wazuh and Sysmon evidence.
- Distinguish directly related events from unrelated security alerts.
- Build an evidence-based assessment of the activity.

## Environment

| Component | Details |
|---|---|
| Endpoint | `DESKTOP-9MMM37V` |
| Operating System | Windows 11 Pro |
| Wazuh Agent | 4.12.0 |
| Wazuh Agent ID | `001` |
| Wazuh Manager | `localhost.localdomain` |
| PowerShell | 7.6.5 |
| Sysmon | Microsoft-Windows-Sysmon/Operational |
| Investigation Date | 24 August 2026 |
| Lab Directory | `C:\InitialPayloadLab` |

## Directory Structure

```text
C:\InitialPayloadLab\
├── InitialPayload\
│   └── payload.ps1
├── Staging\
│   └── system-update.ps1
├── Output\
│   └── execution-result.txt
├── baseline.txt
└── post-activity-artifacts.txt
```

## Lab Workflow

```text
Host Verification
       ↓
Directory Initialization
       ↓
Baseline Collection
       ↓
Initial Payload Creation
       ↓
Payload Staging
       ↓
Content Inspection
       ↓
SHA256 Integrity Verification
       ↓
Payload Execution
       ↓
Execution Output Collection
       ↓
Post-Activity Snapshot
       ↓
Wazuh / Sysmon Correlation
       ↓
Timeline Reconstruction
```

## Initial Payload

The original PowerShell script was created at:

```text
C:\InitialPayloadLab\InitialPayload\payload.ps1
```

File size:

```text
334 bytes
```

The script collects:

- Execution timestamp
- Hostname
- Current user

It writes the results to:

```text
C:\InitialPayloadLab\Output\execution-result.txt
```

The payload does not establish persistence, perform credential theft, communicate with an external server, or modify system security settings.

## Staging Activity

The original script was copied to:

```text
C:\InitialPayloadLab\Staging\system-update.ps1
```

The filename change is relevant from a DFIR perspective because an analyst should establish whether the staged file is identical to the original or has been modified.

## File Integrity Verification

SHA256 was calculated for both files.

Original payload:

```text
882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

Staged payload:

```text
882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

The hashes are identical.

This indicates that the staged file was byte-for-byte identical to the original payload at the time of verification.

## Payload Execution

The staged script was executed using:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Execution was triggered at approximately:

```text
24 August 2026 07:43:35
```

The use of `-ExecutionPolicy Bypass` is an important PowerShell investigation indicator because it bypasses normal execution-policy restrictions for the launched PowerShell process.

However, this parameter alone does not prove malicious activity.

## Execution Output

The payload created:

```text
C:\InitialPayloadLab\Output\execution-result.txt
```

The file contained:

```text
Payload Execution Time: 08/24/2026 07:43:40
Hostname: DESKTOP-9MMM37V
User: desktop-9mmm37v\dell
```

This confirms that the staged PowerShell script executed successfully.

## Wazuh Evidence

Wazuh identified the endpoint as:

```text
Agent ID: 001
Agent Name: DESKTOP-9MMM37V
Operating System: Microsoft Windows 11 Pro
Client Version: Wazuh v4.12.0
Status: Active
```

A Wazuh Syscheck alert was observed at:

```text
Aug 24, 2026 @ 07:30:09.239
```

Relevant fields included:

```text
decoder.name: syscheck_registry_value_deleted
location: syscheck
rule.description: Registry Value Entry Deleted.
```

The event demonstrates that Wazuh File Integrity Monitoring was active on the endpoint.

The registry deletion should not automatically be attributed to the payload. The available evidence does not establish a direct causal relationship between the registry event and the later PowerShell execution.

## Sysmon Evidence

Sysmon Process Creation telemetry was available through:

```text
Microsoft-Windows-Sysmon/Operational
```

The relevant event type is:

```text
Event ID 1 - Process Create
```

A provided Event ID 1 record showed:

```text
Image: C:\WINDOWS\System32\cmd.exe
ProcessId: 25916
UtcTime: 2026-08-24 02:17:49.125
```

This confirms that process creation telemetry was available.

Further correlation is required to identify the exact Sysmon Event ID 1 associated with the PowerShell execution. The corresponding event should be examined for:

- Image
- CommandLine
- ParentImage
- ParentCommandLine
- ProcessId
- ParentProcessId
- ProcessGuid
- User
- UtcTime

## Investigation Limitations

The provided evidence does not include the complete PowerShell Sysmon Event ID 1 record for the execution command.

The Wazuh registry deletion event also cannot be directly linked to the payload without additional process and registry correlation.

A production investigation would therefore require additional telemetry such as:

- PowerShell Event ID 4104
- Sysmon Event ID 1 with complete command-line data
- Sysmon Event ID 3 for network activity
- Windows Security Event ID 4688
- Microsoft Defender detections
- Parent-child process relationships
- User activity
- File creation and modification telemetry

