# Timeline

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

