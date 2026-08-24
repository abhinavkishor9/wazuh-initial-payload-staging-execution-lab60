# Lab 60: Initial Payload → Staging → Execution Investigation

## Overview

This lab investigates a simulated PowerShell payload lifecycle on a Windows 11 endpoint monitored by Wazuh and Sysmon.

The investigation follows the payload through four stages:

1. Initial payload creation
2. Copying the payload into a staging directory
3. Hash-based integrity verification
4. Execution and post-execution artifact collection

The lab is designed as a DFIR and SOC investigation. The payload is intentionally benign and records basic execution information such as the timestamp, hostname, and user.

## Scenario

A PowerShell script is created on a Windows endpoint and subsequently copied into a staging directory under a different filename. The staged script is then executed using PowerShell with `-NoProfile` and `-ExecutionPolicy Bypass`.

The SOC investigator must reconstruct the activity and determine whether the staged file was modified, how it was executed, what artifacts were produced, and what telemetry is available from Wazuh and Sysmon.

## Investigation Objectives

The investigation aims to determine:

- The complete initial payload-to-execution sequence.
- Where the original payload was created.
- When the payload was moved into the staging directory.
- Whether the staged file was modified.
- Whether the original and staged files have identical hashes.
- How the staged PowerShell script was executed.
- What user and host executed the script.
- What artifacts were created after execution.
- What evidence is available through Wazuh FIM and Sysmon.
- Whether the observed activity provides sufficient evidence of malicious behavior.

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

## Key Findings

- A PowerShell payload was created under the `InitialPayload` directory.
- The payload was copied to the `Staging` directory.
- The staged file was renamed `system-update.ps1`.
- Both payload files were 334 bytes.
- Both files produced the same SHA256 hash.
- No modification of the staged payload was identified.
- The staged payload was executed using PowerShell.
- `-ExecutionPolicy Bypass` was used during execution.
- The payload successfully generated an output file.
- The output identified `DESKTOP-9MMM37V` as the execution host.
- The output identified `desktop-9mmm37v\dell` as the executing user.
- Wazuh FIM generated registry monitoring telemetry.
- Sysmon generated process-creation telemetry.
- The available evidence represents a controlled simulation and does not independently demonstrate compromise.

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

## Conclusion

The investigation successfully reconstructed the simulated payload lifecycle from initial creation through staging, integrity verification, and execution. SHA256 verification confirmed that the staged script matched the original payload, while the generated output confirmed successful execution on the Windows endpoint.

The use of `-ExecutionPolicy Bypass` is a noteworthy SOC detection indicator, but it should be evaluated together with script content, execution context, parent process, file origin, and other telemetry. Based on the available evidence and controlled laboratory context, the activity is best classified as authorized simulation activity rather than confirmed malicious behavior.

## Skills Demonstrated

- Windows DFIR
- PowerShell investigation
- Wazuh FIM analysis
- Sysmon investigation
- Process creation analysis
- File timeline analysis
- SHA256 integrity verification
- Artifact collection
- SOC alert correlation
- PowerShell execution-policy analysis
- Evidence-based incident classification
