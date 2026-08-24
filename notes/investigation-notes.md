# Investigation Notes

## Lab 60: Initial Payload → Staging → Execution Investigation

## 1. Investigation Summary

This investigation examines a controlled PowerShell payload lifecycle on the Windows endpoint `DESKTOP-9MMM37V`.

The activity begins with creation of a PowerShell script, followed by copying the script into a staging directory under a different filename. The staged copy is then inspected, hash-verified, and executed using PowerShell with `-ExecutionPolicy Bypass`.

The investigation focuses on establishing the sequence of events, determining whether the staged file was modified, identifying the execution context, and correlating endpoint artifacts with Wazuh and Sysmon telemetry.

## 2. Host Verification

The laboratory environment was verified before creating the investigation artifacts.

```text
PowerShell: PowerShell 7.6.5
Hostname:   DESKTOP-9MMM37V
User:       desktop-9mmm37v\dell
Date:       24 August 2026
Time:       07:35:40
```

This establishes the endpoint and user context for the investigation.

## 3. Directory Initialization

The investigation workspace was created at:

```text
C:\InitialPayloadLab
```

The following directories were created:

```text
C:\InitialPayloadLab\InitialPayload
C:\InitialPayloadLab\Staging
C:\InitialPayloadLab\Output
```

The directories separate the original payload, staged copy, and execution output.

## 4. Baseline Collection

A directory baseline was collected before the main payload activity.

```powershell
Get-ChildItem "C:\InitialPayloadLab" -Recurse -Force |
Select-Object FullName, Length, CreationTime, LastWriteTime |
Out-File "C:\InitialPayloadLab\baseline.txt"
```

The baseline provides a reference for comparing the directory before and after execution.

## 5. Initial Payload Creation

The original script was created at:

```text
C:\InitialPayloadLab\InitialPayload\payload.ps1
```

File size:

```text
334 bytes
```

The script performs basic host profiling:

```powershell
$timestamp = Get-Date
$hostname = $env:COMPUTERNAME
$user = whoami
```

The collected information is written to:

```text
C:\InitialPayloadLab\Output\execution-result.txt
```

## 6. Payload Staging

The payload was copied using:

```powershell
Copy-Item `
"C:\InitialPayloadLab\InitialPayload\payload.ps1" `
"C:\InitialPayloadLab\Staging\system-update.ps1"
```

The staged file became:

```text
C:\InitialPayloadLab\Staging\system-update.ps1
```

The change in filename is relevant because the investigator must verify whether the renamed file remains identical to the original.

## 7. Staged File Inspection

The staging directory was inspected with:

```powershell
Get-ChildItem "C:\InitialPayloadLab\Staging" -Force |
Select-Object Name, Length, CreationTime, LastWriteTime
```

Observed values:

```text
Name:          system-update.ps1
Length:        334
CreationTime:  24-08-2026 07:40:29
LastWriteTime: 24-08-2026 07:37:55
```

The staged script content was also inspected.

No additional commands were identified compared with the original payload.

## 8. Hash Verification

The original payload hash was:

```text
Algorithm: SHA256
Hash: 882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

The staged payload hash was:

```text
Algorithm: SHA256
Hash: 882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

### Finding

The hashes are identical.

This establishes that the original and staged payloads were byte-for-byte identical when the hashes were calculated.

This is stronger evidence than relying only on filenames, timestamps, or visual content comparison.

## 9. Payload Execution

The execution timestamp was recorded before launching the script:

```text
24 August 2026 07:43:35
```

The execution command was:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\InitialPayloadLab\Staging\system-update.ps1"
```

The script executed successfully.

## 10. Execution Policy Bypass Indicator

The command contains:

```text
-ExecutionPolicy Bypass
```

This is a notable PowerShell execution indicator.

In a real SOC investigation, this should trigger additional analysis of:

- PowerShell command line
- Parent process
- User context
- Script contents
- Script origin
- File reputation
- Network connections
- Persistence mechanisms
- Other endpoint alerts

The parameter itself does not prove that the activity is malicious.

In this laboratory, the command was intentionally used as part of the simulation.

## 11. Execution Output

The payload created:

```text
C:\InitialPayloadLab\Output\execution-result.txt
```

The output contained:

```text
Payload Execution Time: 08/24/2026 07:43:40
Hostname: DESKTOP-9MMM37V
User: desktop-9mmm37v\dell
```

### Finding

The output confirms:

1. The script executed.
2. Execution occurred at approximately 07:43:40.
3. The execution host was `DESKTOP-9MMM37V`.
4. The execution user was `desktop-9mmm37v\dell`.

## 12. Post-Activity Artifact Collection

After execution, the investigation directory was enumerated:

```powershell
Get-ChildItem "C:\InitialPayloadLab" -Recurse -Force |
Select-Object FullName, Length, CreationTime, LastWriteTime |
Sort-Object CreationTime
```

The resulting inventory included:

```text
C:\InitialPayloadLab\InitialPayload\payload.ps1
C:\InitialPayloadLab\Staging\system-update.ps1
C:\InitialPayloadLab\Output\execution-result.txt
C:\InitialPayloadLab\baseline.txt
C:\InitialPayloadLab\post-activity-artifacts.txt
```

The final inventory was saved as:

```text
C:\InitialPayloadLab\post-activity-artifacts.txt
```

## 13. Wazuh Agent Status

The Wazuh manager reported:

```text
Agent ID:        001
Agent Name:      DESKTOP-9MMM37V
Status:           Active
Operating System: Microsoft Windows 11 Pro
Client Version:   Wazuh v4.12.0
```

This confirms that the Windows endpoint was actively connected to Wazuh during the investigation.

## 14. Wazuh FIM Alert

A Wazuh Syscheck alert was observed at:

```text
Aug 24, 2026 @ 07:30:09.239
```

Relevant fields:

```text
decoder.name:     syscheck_registry_value_deleted
location:         syscheck
rule.description: Registry Value Entry Deleted.
agent.id:         001
agent.name:       DESKTOP-9MMM37V
```

The alert indicates that Wazuh detected a registry value deletion.

### Investigation Assessment

The registry event should not automatically be attributed to the PowerShell payload.

The available evidence does not demonstrate that the payload caused the registry deletion.

Additional correlation would be required using:

- Process creation events
- Registry modification telemetry
- Timestamps
- Parent-child process relationships
- User context
- PowerShell logging

## 15. Sysmon Telemetry

Sysmon Process Creation telemetry was available in:

```text
Microsoft-Windows-Sysmon/Operational
```

Event ID:

```text
1 - Process Create
```

One provided event showed:

```text
Image: C:\WINDOWS\System32\cmd.exe
ProcessId: 25916
UtcTime: 2026-08-24 02:17:49.125
```

This confirms that Sysmon process telemetry was available on the endpoint.

However, this specific `cmd.exe` event cannot be conclusively attributed to the payload execution using the supplied information.

The exact PowerShell Event ID 1 should be located and correlated using:

- Image
- CommandLine
- ParentImage
- ParentCommandLine
- ProcessId
- ParentProcessId
- ProcessGuid
- User
- UtcTime

## 16. Evidence Assessment

| Artifact | Observation | Assessment |
|---|---|---|
| `payload.ps1` | 334-byte PowerShell script | Initial payload |
| `system-update.ps1` | Staged copy | Staging activity |
| SHA256 | Matching hashes | No content modification identified |
| `-ExecutionPolicy Bypass` | Present | Notable execution indicator |
| `execution-result.txt` | Created at 07:43:40 | Confirms successful execution |
| Wazuh Syscheck | Registry deletion detected | Separate FIM observation |
| Sysmon Event ID 1 | Process telemetry available | Requires exact process correlation |
| `baseline.txt` | Created before payload execution | Baseline evidence |
| `post-activity-artifacts.txt` | Created after execution | Post-activity evidence |

## 17. Investigation Conclusion

The evidence supports a controlled sequence of payload creation, staging, integrity verification, and execution.

The matching SHA256 values establish that the staged payload was not modified relative to the original payload at the time of verification.

The execution-result artifact confirms successful execution and identifies the endpoint and user context.

The use of `-ExecutionPolicy Bypass` is relevant from a SOC detection perspective, but the available evidence does not establish malicious intent, persistence, credential theft, lateral movement, or command-and-control activity.

The activity should therefore be treated as controlled laboratory simulation activity.
