# Splunk SOC Detection Lab

## Project Overview

This project documents the buildout of a beginner-to-intermediate SOC detection lab using **Splunk Enterprise**, **Sysmon**, and the **Splunk Universal Forwarder** on a Windows endpoint.

The objective of this lab is to gain hands-on experience with endpoint telemetry, Windows Event Logs, SIEM ingestion, Splunk search, and basic threat hunting workflows.

## Lab Goals

- Build a working local SIEM lab using Splunk Enterprise.
- Collect Windows Security, System, Application, and Sysmon logs.
- Configure Splunk Universal Forwarder to send endpoint telemetry to Splunk.
- Validate Sysmon process creation and network connection events.
- Practice basic SOC analyst and detection engineering workflows.
- Document the build for portfolio and resume use.

## Environment

| Component | Description |
|---|---|
| Operating System | Windows 11 |
| SIEM | Splunk Enterprise |
| Log Forwarder | Splunk Universal Forwarder |
| Endpoint Telemetry | Sysmon |
| Hostname | `DESKTOP-EQBN02I` |
| Local IP | `10.0.0.129` |
| Splunk Receiving Port | `9997` |

## Architecture

```text
Windows Endpoint
   |
   |  Sysmon + Windows Event Logs
   v
Splunk Universal Forwarder
   |
   |  TCP 9997
   v
Splunk Enterprise
   |
   v
Search, Detection, Threat Hunting
```

## Step 1 – Network Verification

I verified the host network configuration using `ipconfig`. This confirmed the endpoint IP address and helped validate local connectivity for Splunk forwarding.

```powershell
ipconfig
```

![IP configuration](screenshots/01-ipconfig.png)

Key result:

```text
IPv4 Address: 10.0.0.129
Default Gateway: 10.0.0.1
```

## Step 2 – Splunk Universal Forwarder Configuration

The Splunk Universal Forwarder was configured to forward logs to the local Splunk Enterprise instance using port `9997`.

Validation command:

```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe list forward-server
```

Expected result:

```text
Active forwards:
127.0.0.1:9997
```

## Step 3 – Windows Event Log Inputs

The following file was created and configured:

```text
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf
```

Current lab configuration:

```ini
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0
index = wineventlog

[WinEventLog://Security]
disabled = 0
index = wineventlog

[WinEventLog://System]
disabled = 0
index = wineventlog

[WinEventLog://Application]
disabled = 0
index = wineventlog
```

![inputs.conf configuration](screenshots/02-inputs-conf.png)

> Note: Sysmon is currently being ingested into `wineventlog` for simplicity during this phase. A separate `sysmon` index can be created later for cleaner separation.

## Step 4 – Sysmon Local Verification

Before troubleshooting Splunk ingestion, I confirmed that Sysmon was generating events locally on the endpoint.

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

![Sysmon local verification](screenshots/03-sysmon-local-verification.png)

This confirmed Sysmon events such as:

- Event ID 1 – Process Create
- Event ID 11 – File Created

## Step 5 – Splunk Input Validation with btool

To confirm that the Splunk Universal Forwarder was reading the Sysmon input stanza, I used `btool`.

```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe btool inputs list --debug | findstr /i sysmon
```

![btool Sysmon validation](screenshots/04-btool-input-validation.png)

This confirmed Splunk was parsing the local `inputs.conf` file and recognized:

```text
[WinEventLog://Microsoft-Windows-Sysmon/Operational]
```

## Step 6 – Successful Sysmon Ingestion in Splunk

After troubleshooting, Sysmon logs were successfully visible in Splunk.

Example search for Sysmon network connection events:

```spl
index=* EventCode=3 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

![Sysmon network connection events](screenshots/05-sysmon-network-events-splunk.png)

This confirmed ingestion of Event ID 3 network connection telemetry, including fields such as:

- `EventCode`
- `source`
- `sourcetype`
- `Computer`
- `Image`
- `DestinationIp`
- `DestinationPort`

## Step 7 – Threat Hunting Query Validation

I tested a basic threat hunting query looking for Command Prompt process execution using Sysmon Event ID 1.

```spl
index=* EventCode=1 cmd.exe
```

![Threat hunting command line query](screenshots/06-threat-hunting-cmd-query.png)

This search returned command-line activity and confirmed that fields such as `CommandLine` were available for investigation.

## Starter Hunt Queries

### Process Creation

```spl
index=* EventCode=1
```

### Network Connections

```spl
index=* EventCode=3 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

### Command Prompt Execution

```spl
index=* EventCode=1 cmd.exe
```

### PowerShell Execution

```spl
index=* EventCode=1 powershell
```

### Encoded PowerShell Indicator

```spl
index=* EventCode=1 CommandLine="*enc*"
```

### Failed Logons

```spl
index=* EventCode=4625
```

### RDP Logons

```spl
index=* EventCode=4624 Logon_Type=10
```

## Troubleshooting Summary

During the build, I encountered several common SIEM lab issues:

| Issue | Resolution |
|---|---|
| Forwarder connected but no Sysmon logs visible | Validated Sysmon locally and checked `inputs.conf` with `btool` |
| `inputs.conf` file did not initially exist | Created the file manually under `etc\system\local` |
| Sysmon events not obvious in Splunk | Searched broadly using `index=*`, `EventCode`, and `source` |
| Raw XML-style events | Confirmed fields such as `EventCode`, `CommandLine`, and `source` were still searchable |

## Skills Demonstrated

- Splunk Enterprise setup
- Splunk Universal Forwarder configuration
- Windows Event Log collection
- Sysmon deployment and validation
- SIEM ingestion troubleshooting
- `inputs.conf` configuration
- Splunk `btool` validation
- SPL search basics
- Endpoint telemetry analysis
- Basic threat hunting methodology
- Build a dashboard for process creation, network connections, and authentication events.
- Add Atomic Red Team simulations.
- Map detections to MITRE ATT&CK.
- Write detection rules for suspicious PowerShell, LOLBins, and persistence techniques.
