# Troubleshooting Notes

## 1. Directory Does Not Exist

### Symptom

PowerShell reports that `C:\InitialPayloadLab` or one of its subdirectories cannot be found.

### Resolution

Create the directory structure:

```powershell
New-Item -Path "C:\InitialPayloadLab" -ItemType Directory -Force
New-Item -Path "C:\InitialPayloadLab\InitialPayload" -ItemType Directory -Force
New-Item -Path "C:\InitialPayloadLab\Staging" -ItemType Directory -Force
New-Item -Path "C:\InitialPayloadLab\Output" -ItemType Directory -Force
```

Verify the structure:

```powershell
Get-ChildItem "C:\InitialPayloadLab" -Recurse
```

## 2. Payload File Not Found

### Symptom

PowerShell reports that:

```text
C:\InitialPayloadLab\InitialPayload\payload.ps1
```

cannot be found.

### Resolution

Check whether the file exists:

```powershell
Get-Item "C:\InitialPayloadLab\InitialPayload\payload.ps1"
```

If the file does not exist, recreate the payload before performing the staging step.

## 3. Staged File Not Found

### Symptom

The staged script cannot be located.

### Resolution

Inspect the staging directory:

```powershell
Get-ChildItem "C:\InitialPayloadLab\Staging" -Force
```

The expected file is:

```text
system-update.ps1
```

Expected path:

```text
C:\InitialPayloadLab\Staging\system-update.ps1
```

## 4. Copy Operation Fails

### Symptom

The `Copy-Item` command fails or the staged file is not created.

### Resolution

Verify the source:

```powershell
Get-Item "C:\InitialPayloadLab\InitialPayload\payload.ps1"
```

Then perform the copy:

```powershell
Copy-Item `
"C:\InitialPayloadLab\InitialPayload\payload.ps1" `
"C:\InitialPayloadLab\Staging\system-update.ps1"
```

Verify:

```powershell
Get-Item "C:\InitialPayloadLab\Staging\system-update.ps1"
```

## 5. Hashes Do Not Match

### Symptom

The SHA256 hash of the original and staged files differs.

### Resolution

Calculate both hashes:

```powershell
Get-FileHash "C:\InitialPayloadLab\InitialPayload\payload.ps1"

Get-FileHash "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Inspect the contents:

```powershell
Get-Content "C:\InitialPayloadLab\InitialPayload\payload.ps1"

Get-Content "C:\InitialPayloadLab\Staging\system-update.ps1"
```

The expected SHA256 value for both files in this lab is:

```text
882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

A different hash indicates that the files are not byte-for-byte identical.

## 6. Timestamp Differences

### Observation

The staged file contains:

```text
Creation Time:  24-08-2026 07:40:29
Last Write Time: 24-08-2026 07:37:55
```

### Explanation

File timestamps should not be interpreted independently.

A file can have different creation and modification timestamps because copying, file-system behavior, and metadata preservation can affect the resulting timestamps.

Use:

```powershell
Get-Item "C:\InitialPayloadLab\Staging\system-update.ps1" |
Select-Object FullName, Length, CreationTime, LastWriteTime
```

Correlate timestamps with:

- PowerShell activity
- Sysmon
- Wazuh
- File hashes
- Other endpoint telemetry

## 7. Script Does Not Execute

### Symptom

PowerShell runs but the expected output file does not appear.

### Resolution

Verify the staged file:

```powershell
Get-Item "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Inspect the content:

```powershell
Get-Content "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Verify the output directory:

```powershell
Get-Item "C:\InitialPayloadLab\Output"
```

Execute the script:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Check the output:

```powershell
Get-Item "C:\InitialPayloadLab\Output\execution-result.txt"

Get-Content "C:\InitialPayloadLab\Output\execution-result.txt"
```

## 8. Execution Policy Bypass

### Observation

The execution command contains:

```text
-ExecutionPolicy Bypass
```

### Investigation Note

This is a notable PowerShell detection indicator because the process explicitly uses an execution-policy bypass.

It should not automatically be classified as malicious.

The investigator should examine:

- Parent process
- Command line
- User
- Script contents
- File origin
- Network activity
- Persistence
- Security-product alerts
- Expected administrative activity

In this lab, the use of `-ExecutionPolicy Bypass` is intentional.

## 9. Wazuh Agent Not Active

### Verification

From the Wazuh manager:

```bash
sudo /var/ossec/bin/agent_control -i 001
```

Expected:

```text
Agent ID:   001
Agent Name: DESKTOP-9MMM37V
Status:     Active
Operating system: Microsoft Windows 11 Pro
Client version:  Wazuh v4.12.0
```

If the agent is not active, endpoint telemetry may not reach the Wazuh manager.

## 10. Wazuh FIM Alert Timing

### Observation

The Wazuh agent reported:

```text
Syscheck last started at:
Sun Aug 23 19:24:19 2026

Syscheck last ended at:
Mon Aug 24 07:30:09 2026
```

FIM events may therefore depend on the configured scan behavior and timing.

A file created after a scan begins may not necessarily appear immediately in the Wazuh dashboard.

When investigating FIM results, consider:

- Syscheck schedule
- Real-time monitoring configuration
- File creation time
- File modification time
- Agent status
- Manager ingestion time

## 11. Sysmon Event ID 1 Correlation

### Observation

Sysmon Event ID 1 telemetry was available.

One provided event showed:

```text
Image: C:\WINDOWS\System32\cmd.exe
ProcessId: 25916
UtcTime: 2026-08-24 02:17:49.125
```

### Investigation Guidance

This event should not automatically be considered the payload execution event.

Locate the Sysmon Event ID 1 record corresponding to:

```text
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Review:

- Image
- CommandLine
- ParentImage
- ParentCommandLine
- ProcessId
- ParentProcessId
- ProcessGuid
- User
- UtcTime

## 12. Wazuh Registry Alert Correlation

### Observation

Wazuh generated:

```text
decoder.name: syscheck_registry_value_deleted
rule.description: Registry Value Entry Deleted.
```

The event occurred at:

```text
Aug 24, 2026 @ 07:30:09.239
```

### Investigation Guidance

Do not automatically connect this event to the PowerShell payload.

The registry alert occurred before the documented payload execution at approximately 07:43.

Additional evidence would be required to establish a relationship.

Useful correlation sources include:

- Sysmon process events
- Sysmon registry events
- Windows Security logs
- PowerShell logs
- User activity
- Process parent-child relationships

## 13. Baseline File Appears in Baseline

### Observation

The baseline file itself appears in the directory listing:

```text
C:\InitialPayloadLab\baseline.txt
```

### Explanation

The baseline command writes its output into the same directory being enumerated.

Therefore, the output file can appear in the resulting inventory.

This is expected and is not itself suspicious.

## 14. Post-Activity Snapshot

The post-activity inventory was written to:

```text
C:\InitialPayloadLab\post-activity-artifacts.txt
```

This is an investigator-generated artifact.

It should not be confused with a file created by the payload.

## 15. Missing PowerShell Telemetry

If the exact PowerShell execution event cannot be found, check whether PowerShell logging is enabled.

Useful telemetry includes:

```text
PowerShell Operational
Event ID 4104 - Script Block Logging
```

Other useful sources include:

```text
Windows Security
Event ID 4688 - Process Creation
```

and:

```text
Sysmon
Event ID 1 - Process Create
```

The absence of a specific telemetry source should be documented as an investigation limitation rather than filled with assumptions.

