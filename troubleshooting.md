# Troubleshooting Notes

## Issue: Splunk Universal Forwarder was connected, but Sysmon logs were not visible

### Validation steps performed

1. Confirmed Sysmon was generating local Windows Event Log entries:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
```

2. Confirmed the Splunk Universal Forwarder was reading the Sysmon stanza:

```powershell
cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe btool inputs list --debug | findstr /i sysmon
```

3. Confirmed Sysmon events in Splunk using broader searches:

```spl
index=* EventCode=1
index=* EventCode=3 source="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

## Lesson learned

If Windows Event Logs are ingesting but Sysmon is not, validate the telemetry pipeline in this order:

1. Confirm Sysmon generates local events.
2. Confirm the forwarder is running.
3. Confirm `inputs.conf` exists and is parsed by Splunk.
4. Confirm the Splunk receiving port is active.
5. Search broadly by `source`, `sourcetype`, and `EventCode`.
