# Timeline

## Lab 60: Initial Payload → Staging → Execution Investigation

## Investigation Timeline

| Time | Source | Event | Significance |
|---|---|---|---|
| 07:35:40 | PowerShell | `hostname`, `whoami`, and `Get-Date` executed | Establishes host, user, and investigation start time |
| 07:36:22 | PowerShell | `C:\InitialPayloadLab` directory structure created | Initializes investigation workspace |
| 07:37:17 | PowerShell | `baseline.txt` created | Establishes pre-activity directory baseline |
| 07:37:55 | PowerShell | `payload.ps1` created | Establishes initial payload artifact |
| 07:37:55 | File Metadata | Original payload creation/last-write timestamp | Establishes initial file timestamp |
| 07:37:55 | PowerShell | `payload.ps1` copied to `Staging\system-update.ps1` | Establishes staging activity |
| 07:40:29 | File Metadata | `system-update.ps1` creation time | Records staged file creation timestamp |
| 07:40:29 | PowerShell | Staged file content inspected | Confirms staged script contents |
| 07:40:29 | PowerShell | SHA256 calculated for original and staged files | Begins integrity verification |
| 07:40:29 | File Hash | Both files produce identical SHA256 | Confirms no content modification was identified |
| 07:43:35 | PowerShell | Staged script executed with `-ExecutionPolicy Bypass` | Payload execution trigger |
| 07:43:40 | Payload Output | `execution-result.txt` created | Confirms successful execution |
| 07:43:40 | Payload Output | Host recorded as `DESKTOP-9MMM37V` | Confirms execution endpoint |
| 07:43:40 | Payload Output | User recorded as `desktop-9mmm37v\dell` | Confirms execution context |
| After 07:43:40 | PowerShell | Post-activity directory snapshot collected | Captures resulting artifacts |
| After 07:43:40 | PowerShell | `post-activity-artifacts.txt` created | Preserves post-execution artifact inventory |

## Wazuh and Windows Telemetry

| Time | Source | Event | Significance |
|---|---|---|---|
| 07:30:09.239 | Wazuh | `syscheck_registry_value_deleted` | Registry value deletion detected by FIM |
| 07:30:09.239 | Wazuh | `Registry Value Entry Deleted.` | Demonstrates active Wazuh FIM telemetry |
| 07:47:49 | Sysmon | Event ID 1 - Process Create | Process creation telemetry available after documented payload execution |
| 02:17:49 UTC | Sysmon | `cmd.exe` process creation | Requires additional correlation before attribution to payload |

## Important File Timestamps

### Initial Payload

```text
File:
C:\InitialPayloadLab\InitialPayload\payload.ps1

Length:
334 bytes

Creation Time:
24-08-2026 07:37:55

Last Write Time:
24-08-2026 07:37:55
```

### Staged Payload

```text
File:
C:\InitialPayloadLab\Staging\system-update.ps1

Length:
334 bytes

Creation Time:
24-08-2026 07:40:29

Last Write Time:
24-08-2026 07:37:55
```

### Execution Output

```text
File:
C:\InitialPayloadLab\Output\execution-result.txt

Length:
202 bytes

Creation Time:
24-08-2026 07:43:40

Last Write Time:
24-08-2026 07:43:40
```

## Hash Timeline

The original payload was hashed and produced:

```text
882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

The staged payload produced the same hash:

```text
882C88FFA9B650FFA820A2A190BA5CEE199D08A75BE79EDCA1F7A5E4C2983CFC
```

### Integrity Finding

The matching SHA256 values establish that the original and staged files were byte-for-byte identical at the time of verification.

## Execution Timeline

The documented execution command was:

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File "C:\InitialPayloadLab\Staging\system-update.ps1"
```

Execution was triggered at approximately:

```text
24 August 2026 07:43:35
```

The payload output was created approximately five seconds later:

```text
24 August 2026 07:43:40
```

The output contained:

```text
Payload Execution Time: 08/24/2026 07:43:40
Hostname: DESKTOP-9MMM37V
User: desktop-9mmm37v\dell
```

## Reconstructed Activity Flow

```text
07:35:40
Host Verification
        ↓
07:36:22
Lab Directory Creation
        ↓
07:37:17
Baseline Collection
        ↓
07:37:55
Initial Payload Creation
        ↓
07:37:55
Payload Staging
        ↓
07:40:29
Staged File Inspection
        ↓
07:40:29
SHA256 Integrity Verification
        ↓
07:43:35
PowerShell Execution
        ↓
07:43:40
Execution Output Created
        ↓
After 07:43:40
Post-Activity Artifact Collection
```

## Wazuh Timeline Assessment

The Wazuh registry deletion occurred at:

```text
Aug 24, 2026 @ 07:30:09.239
```

This is before the documented payload execution at approximately 07:43:35.

The event was:

```text
syscheck_registry_value_deleted
```

with the rule:

```text
Registry Value Entry Deleted.
```

The available evidence does not establish that this registry activity was caused by the payload.

It should therefore remain a separate Wazuh observation unless additional telemetry provides a causal relationship.

## Sysmon Timeline Assessment

A Sysmon Event ID 1 record was observed at:

```text
24-08-2026 07:47:49
```

The provided event showed:

```text
Image:
C:\WINDOWS\System32\cmd.exe

ProcessId:
25916

UtcTime:
2026-08-24 02:17:49.125
```

The event confirms process-creation telemetry but does not contain enough information in the provided extraction to establish that it represents the documented PowerShell execution.

The exact PowerShell Event ID 1 should be located for complete correlation.

## Final Timeline Assessment

The strongest confirmed sequence is:

```text
Initial Payload
      ↓
Staging
      ↓
Content Inspection
      ↓
SHA256 Verification
      ↓
PowerShell Execution
      ↓
Execution Output
      ↓
Post-Activity Collection
```

The matching hashes establish payload integrity between the original and staged files.

The execution output establishes successful execution and identifies the host and user.

The use of `-ExecutionPolicy Bypass` is a significant SOC investigation indicator but does not independently prove malicious intent.

The Wazuh registry alert and the provided Sysmon `cmd.exe` event require additional correlation and should not be attributed to the payload without supporting evidence.
