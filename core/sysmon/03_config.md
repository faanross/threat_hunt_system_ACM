
## **Configuration Deep Dive**

### Overview


Creating, maintaining, and refining the configuration file is arguably the most important skill when working with Sysmon. While you must obviously be skilled at locating and interpreting logs during investigations, the logs themselves are only as useful as the configuration that generates them.

As mentioned in the previous section, the SwiftOnSecurity config provides an excellent baseline to start with. It will get you up and running quickly and offers a significant improvement over the default configuration - or not running Sysmon at all. However, once Sysmon is operational with that config, you should immediately familiarize yourself with each section and learn to confidently make changes suited to your specific environment and threat hunting approach.

There are two key areas of knowledge required:

- Understanding the language (XML) and logical structure of the config, including rule syntax
- Understanding the 29 specific Event IDs, their relevance to threat detection, and how to write rules that continuously optimize their value

In this section, we'll focus primarily on the former. Following this, we'll cover each Event ID in depth, discussing both its relevance to threat hunting and how to write specific rules for it.





### **Understanding Sysmon Configuration Structure**

Sysmon's behaviour is controlled by an XML configuration file that defines:

1. Which event types to monitor
2. What filters to apply (include or exclude specific events)
3. What additional context to capture (hashes, network details, etc.)

The configuration uses a two-phase filtering model:

**Include Filters**: Only events matching these rules are logged  
**Exclude Filters**: Events matching these rules are discarded

This allows for flexible rule creation. You might include all network connections (Event ID 3) but exclude connections to known-good internal services to reduce noise.





### **Configuration File Anatomy**

A basic Sysmon configuration file structure looks like this:

```xml
<Sysmon schemaversion="4.90">
  <HashAlgorithms>SHA256,IMPHASH</HashAlgorithms>
  <CheckRevocation/>
 
  <EventFiltering>
    <!-- Event ID 1: Process Creation -->
    <RuleGroup name="" groupRelation="or">
      <ProcessCreate onmatch="include">
        <!-- Include rules here -->
      </ProcessCreate>
    </RuleGroup>
    
    <!-- Event ID 3: Network Connection -->
    <RuleGroup name="" groupRelation="or">
      <NetworkConnect onmatch="include">
        <!-- Include rules here -->
      </NetworkConnect>
    </RuleGroup>
    
    <!-- Additional event types... -->
    
  </EventFiltering>
</Sysmon>
```

**Key Configuration Elements:**

**schemaversion**: Specifies which Sysmon configuration schema version is being used (in this example 4.90)

**HashAlgorithms**: Defines which file hash algorithms to use (MD5, SHA1, SHA256, IMPHASH). SHA256 is recommended for security, though it has slightly higher CPU overhead than MD5. I also include the IMPHASH, which is specifically for PE files and tracks the imported functions, which is useful for detecting malware variants. It's a great addition for threat hunting alongside SHA256. Note that they appear in multiple events where files are referenced (primarily Event ID 1, but also 6, 7, etc.) in fields like `Hashes: SHA256=...,IMPHASH=...`

**CheckRevocation**: When present, Sysmon checks the revocation status of code-signing certificates for loaded images. It can thus detect whether compromised certificates were used by attackers (stolen/leaked certs that were later revoked). Result is displayed in `SignatureStatus` field of specific Event IDs - 1, 6, 7 etc.

**EventFiltering**: Container for all rule groups

**RuleGroup**: Groups related rules together, with `groupRelation="or"` meaning any rule in the group triggers a match. Note that the attribute is just metadata for organizing the config - it does not appear in the Sysmon event logs.

**onmatch**: Specifies the action-`include` means "log events matching these rules," `exclude` means "don't log events matching these rules"


This is just a high-level preview, everything mentioned here + more will be unpacked in *much* greater detail shortly.


When on a host and you want a quick overview of config rules, values etc (akin to `man` pages), you can run:

```
Sysmon64.exe -? config
```



### Basic Rule Syntax

Sysmon rules use condition operators to match event properties:

- `is`: Exact match
- `is not`: Exact non-match
- `contains`: Substring match
- `contains any`: Matches if any of the listed values are found
- `contains all`: Matches if all of the listed values are found
- `excludes`: Does not contain substring
- `begin with`: String starts with
- `end with`: String ends with
- `less than`: Numeric comparison
- `more than`: Numeric comparison
- `image`: Matches image path (special case-insensitive comparison)


Let's look at one simplified example per rule type, just to get better insight into how each works, and then we'll look at a few more realistic/representative examples.

#### Sysmon Condition Syntax Examples

##### `is` - Exact match

```xml
<DestinationPort condition="is">443</DestinationPort>
```

Matches only port 443 exactly.

##### `is not` - Exact non-match

```xml
<User condition="is not">SYSTEM</User>
```

Matches everything except when user is "SYSTEM".


##### `contains` - Substring match

```xml
<CommandLine condition="contains">powershell</CommandLine>
```

Matches if "powershell" appears anywhere in command line.


##### `contains any` - Matches if any listed value found

```xml
<CommandLine condition="contains any">cmd;powershell;wscript</CommandLine>
```

Matches if command line contains "cmd" OR "powershell" OR "wscript" (semicolon-separated).


##### `contains all` - Matches if all listed values found

```xml
<CommandLine condition="contains all">-enc;bypass</CommandLine>
```

Matches only if command line contains both "-enc" AND "bypass".


##### `excludes` - Does not contain substring

```xml
<Image condition="excludes">Windows</Image>
```

Matches processes whose path doesn't contain "Windows".




##### `begin with` - String starts with

```xml
<DestinationIp condition="begin with">192.168.</DestinationIp>
```

Matches IPs starting with "192.168."


##### `end with` - String ends with

```xml
<Image condition="end with">.exe</Image>
```

Matches executables ending in ".exe".


##### `less than` - Numeric comparison

```xml
<DestinationPort condition="less than">1024</DestinationPort>
```

Matches ports below 1024 (privileged ports).


##### `more than` - Numeric comparison

```xml
<DestinationPort condition="more than">49152</DestinationPort>
```

Matches ports above 49152 (dynamic/ephemeral range).

##### `image` - Image path comparison (case-insensitive)

```xml
<Image condition="image">C:\Windows\System32\cmd.exe</Image>
```

Matches cmd.exe path, ignoring case differences (special matching for executable paths).



#### More Realistic Examples

In practice we typically don't use a single rule per RuleGroup, rather combine multiple.


##### **Example: Detecting Suspicious PowerShell Execution**

Let's create a rule to log PowerShell executions with suspicious command-line arguments:

```xml
<RuleGroup name="Suspicious PowerShell" groupRelation="or">
  <ProcessCreate onmatch="include">
    <Image condition="end with">powershell.exe</Image>
    <CommandLine condition="contains any">-EncodedCommand;-enc;-ec;-ExecutionPolicy Bypass;-w hidden;-WindowStyle Hidden</CommandLine>
  </ProcessCreate>
</RuleGroup>
```

This rule logs any PowerShell execution where the command line contains common evasion techniques: Base64-encoded commands, execution policy bypasses, or hidden windows.

##### **Example: Detecting Connections to Unusual External Ports**

Create a rule to log network connections to non-standard ports that might indicate C2 communication:

```xml
<RuleGroup name="Unusual Outbound Ports" groupRelation="or">
  <NetworkConnect onmatch="include">
    <DestinationIsIpv6 condition="is">false</DestinationIsIpv6>
    <DestinationIp condition="begin with">172.16.</DestinationIp>
    <DestinationPort condition="is">8443</DestinationPort>
    <DestinationPort condition="is">4444</DestinationPort>
    <DestinationPort condition="is">6666</DestinationPort>
    <DestinationPort condition="is">9999</DestinationPort>
  </NetworkConnect>
</RuleGroup>
```

This logs connections to common C2 ports (8443, 4444, 6666, 9999) destined for external IPs.

##### **Example: Detecting Named Pipe Creation with Suspicious Names**

We'll often craft rules based on specific known malware behaviour/signatures. Here for example we are looking for certain named pipe names due to their association with AdaptixC2.

```xml
<RuleGroup name="Suspicious Named Pipes" groupRelation="or">
  <PipeEvent onmatch="include">
    <PipeName condition="contains any">\agent;\shell;\cmd;\pipe\backdoor;\msagent_</PipeName>
  </PipeEvent>
</RuleGroup>
```



##### Example: **Combining Include and Exclude:**

The examples above combined multiple rules, but only a single Include/Exclude. We can however also combine these in a single RuleGroup for maximum fine-grained control. For example, to log all process creations _except_ from known-good parent processes:

```xml
<RuleGroup name="Process Creation - Filtered" groupRelation="or">
  <ProcessCreate onmatch="exclude">
    <!-- Exclude processes spawned by Windows Update -->
    <ParentImage condition="is">C:\Windows\System32\svchost.exe</ParentImage>
    <ParentCommandLine condition="contains">wuauserv</ParentCommandLine>
  </ProcessCreate>
  
  <ProcessCreate onmatch="include">
    <!-- Include everything else -->
    <Image condition="is not">NOT_MATCHING</Image>
  </ProcessCreate>
</RuleGroup>
```


### **Testing Your Configuration**

#### Invalid Syntax

Sysmon will test and reject an invalid XML file when attempting to apply it, so to "test" the config syntax you can simply attempt to apply it using the command:

```cmd
Sysmon64.exe -c new_config.xml
```

If the config is invalid, Sysmon will report the specific errors, making it easy to know exactly what is wrong and how to go about fixing it.

#### Volume Assessment

Remember, the command above will test whether the XML is structurally correct, but it won't test to make sure your rules make any sense. Sometimes for example we can create rules that have incorrect logic and produce many more events than we intended. Imagine deploying this across 1000s of systems, only to learn a few hours later that you've filled up the host disks and seriously impacting network performance as the log forwarders all get overwhelmed.

So a good rule of thumb is to always first deploy a new config on representative test systems and monitor the volume of events it generates.

##### Selecting Test Systems

Choose test systems that represent your actual environment. At minimum, test on:

- **Standard workstation** - typical user activity patterns
- **Server** - if you monitor servers, test on one with representative service load
- **High-activity system** - developer workstation, build server, or domain controller if applicable

Different system types generate vastly different event volumes (a DC may produce 10x-100x more authentication events than a workstation), so testing on only one type can give misleading results.

##### Understanding Volume Baselines

Before analyzing changes, understand what "normal" looks like:

**Typical Event Rates:**

- **Workstation:** 100-500 events/minute is normal, investigate if sustained >1,000/min
- **Server:** 500-2,000 events/minute is normal, investigate if sustained >5,000/min
- **Domain Controller:** 5,000-10,000+ events/minute can be normal due to authentication volume

**Disk Impact Calculation:**

```
events/minute × average_event_size × 60 × 24 = MB/day

Example: 500 events/min × 2KB × 1,440 min = ~1.4GB/day
```

Ensure your log retention policy and disk space can accommodate the volume. If forwarding to a SIEM, also consider network bandwidth requirements.

##### Monitoring Event Generation Rate

Deploy the config on your test system(s) and monitor for at least 24 hours to capture full business cycle (including overnight and any scheduled tasks). For better coverage, consider testing over a full week to capture weekend vs. weekday differences.

**Preparation:**

```powershell
# Optional but recommended: Clear existing events for clean baseline
wevtutil cl "Microsoft-Windows-Sysmon/Operational"

# If comparing to existing config, skip clearing to measure delta
```

**Quick Volume Check:**

To quickly check current event counts by type:

```powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | 
    Group-Object Id | 
    Sort-Object Count -Descending | 
    Format-Table Name, Count
```

**Automated 24-Hour Monitoring:**

Here is a complete script that monitors Sysmon event generation rate every 15 minutes for 24 hours, tracking new events per interval (not cumulative log totals), calculating percentage changes, and alerting on anomalous spikes:

```powershell
#requires -RunAsAdministrator

<#
.SYNOPSIS
    Monitors Sysmon event generation rate every 15 minutes for 24 hours with spike detection.

.DESCRIPTION
    This script collects Sysmon event statistics for new events generated during each interval,
    calculates percentage changes between intervals, and alerts on significant spikes.
    Results are exported to CSV and displayed in real-time.

.PARAMETER IntervalMinutes
    Collection interval in minutes (default: 15)

.PARAMETER DurationHours
    Total monitoring duration in hours (default: 24)

.PARAMETER SpikeThresholdPercent
    Percentage threshold for spike detection (default: 100)

.PARAMETER SpikeThresholdAbsolute
    Absolute event count threshold for spike detection (default: 500)

.PARAMETER ClearLogFirst
    Clear the Sysmon log before starting monitoring (default: $false)

.EXAMPLE
    .\Monitor-SysmonEvents.ps1
    
.EXAMPLE
    .\Monitor-SysmonEvents.ps1 -IntervalMinutes 10 -DurationHours 12 -ClearLogFirst $true

.EXAMPLE
    .\Monitor-SysmonEvents.ps1 -SpikeThresholdPercent 150 -SpikeThresholdAbsolute 1000
#>

param(
    [int]$IntervalMinutes = 15,
    [int]$DurationHours = 24,
    [int]$SpikeThresholdPercent = 100,
    [int]$SpikeThresholdAbsolute = 500,
    [bool]$ClearLogFirst = $false
)

# Configuration
$LogName = "Microsoft-Windows-Sysmon/Operational"
$OutputPath = "SysmonMonitoring_$(Get-Date -Format 'yyyyMMdd_HHmmss').csv"
$TotalIntervals = ($DurationHours * 60) / $IntervalMinutes
$StartTime = Get-Date

# Clear log if requested
if ($ClearLogFirst) {
    Write-Host "Clearing Sysmon log for clean baseline..." -ForegroundColor Yellow
    wevtutil cl $LogName
    Write-Host "Log cleared. Starting fresh monitoring.`n" -ForegroundColor Green
}

# Initialize tracking variables
$LastCheckTime = Get-Date
$PreviousCounts = @{}
$AllResults = @()

# Function to get event counts since last check (time-based, not cumulative)
function Get-SysmonEventCounts {
    param($SinceTime)
    
    try {
        $events = Get-WinEvent -LogName $LogName -ErrorAction Stop | 
            Where-Object { $_.TimeCreated -ge $SinceTime }
        
        $grouped = $events | Group-Object Id | Select-Object Name, Count
        
        $counts = @{}
        foreach ($group in $grouped) {
            $counts[$group.Name] = $group.Count
        }
        return $counts
    }
    catch {
        if ($_.Exception.Message -like "*No events were found*") {
            return @{}
        }
        Write-Warning "Error reading Sysmon log: $_"
        return @{}
    }
}

# Function to calculate percentage change
function Get-PercentageChange {
    param($Old, $New)
    
    if ($Old -eq 0) {
        if ($New -eq 0) { return 0 }
        return 100  # New events appeared
    }
    return [math]::Round((($New - $Old) / $Old) * 100, 2)
}

# Function to display results
function Show-Results {
    param($Results, $IntervalNum, $TotalInt, $EventsPerMinute)
    
    Write-Host "`n============================================" -ForegroundColor Cyan
    Write-Host "Interval $IntervalNum of $TotalInt - $(Get-Date -Format 'yyyy-MM-dd HH:mm:ss')" -ForegroundColor Cyan
    Write-Host "Generation Rate: $EventsPerMinute events/minute" -ForegroundColor Cyan
    Write-Host "============================================" -ForegroundColor Cyan
    
    $Results | Format-Table EventID, EventName, CurrentCount, PreviousCount, Change, PercentChange, Status -AutoSize
}

# Function to get event name
function Get-EventName {
    param($EventId)
    
    switch ($EventId) {
        "1" { "Process Creation" }
        "2" { "File Creation Time Changed" }
        "3" { "Network Connection" }
        "4" { "Sysmon Service State Changed" }
        "5" { "Process Terminated" }
        "6" { "Driver Loaded" }
        "7" { "Image Loaded" }
        "8" { "CreateRemoteThread" }
        "9" { "RawAccessRead" }
        "10" { "ProcessAccess" }
        "11" { "FileCreate" }
        "12" { "RegistryEvent (Object create/delete)" }
        "13" { "RegistryEvent (Value Set)" }
        "14" { "RegistryEvent (Key/Value Rename)" }
        "15" { "FileCreateStreamHash" }
        "16" { "ServiceConfigurationChange" }
        "17" { "PipeEvent (Pipe Created)" }
        "18" { "PipeEvent (Pipe Connected)" }
        "19" { "WmiEvent (WmiEventFilter)" }
        "20" { "WmiEvent (WmiEventConsumer)" }
        "21" { "WmiEvent (WmiEventConsumerToFilter)" }
        "22" { "DNSEvent (DNS query)" }
        "23" { "FileDelete" }
        "24" { "ClipboardChange" }
        "25" { "ProcessTampering" }
        "26" { "FileDeleteDetected" }
        "27" { "FileBlockExecutable" }
        "28" { "FileBlockShredding" }
        "29" { "FileExecutableDetected" }
        default { "Event ID $EventId" }
    }
}

# Main monitoring loop
Write-Host "Starting Sysmon Event Generation Rate Monitoring" -ForegroundColor Green
Write-Host "Duration: $DurationHours hours" -ForegroundColor Green
Write-Host "Interval: $IntervalMinutes minutes" -ForegroundColor Green
Write-Host "Total Intervals: $TotalIntervals" -ForegroundColor Green
Write-Host "Spike Threshold: $SpikeThresholdPercent% AND $SpikeThresholdAbsolute absolute events" -ForegroundColor Green
Write-Host "Output File: $OutputPath" -ForegroundColor Green
Write-Host "Start Time: $StartTime`n" -ForegroundColor Green

for ($i = 1; $i -le $TotalIntervals; $i++) {
    # Wait for the interval (except first iteration)
    if ($i -gt 1) {
        Write-Host "Waiting $IntervalMinutes minutes until next collection..." -ForegroundColor Gray
        Start-Sleep -Seconds ($IntervalMinutes * 60)
    }
    
    # Collect current counts (events since last check)
    $CheckTime = Get-Date
    Write-Host "Collecting data for interval $i..." -ForegroundColor Yellow
    $CurrentCounts = Get-SysmonEventCounts -SinceTime $LastCheckTime
    
    # Calculate total events in this interval
    $TotalIntervalEvents = ($CurrentCounts.Values | Measure-Object -Sum).Sum
    $EventsPerMinute = [math]::Round($TotalIntervalEvents / $IntervalMinutes, 2)
    
    # Get all unique event IDs from current and previous intervals
    $AllEventIds = ($PreviousCounts.Keys + $CurrentCounts.Keys) | Select-Object -Unique
    
    $IntervalResults = @()
    
    foreach ($EventId in $AllEventIds) {
        $Previous = if ($PreviousCounts.ContainsKey($EventId)) { $PreviousCounts[$EventId] } else { 0 }
        $Current = if ($CurrentCounts.ContainsKey($EventId)) { $CurrentCounts[$EventId] } else { 0 }
        $Change = $Current - $Previous
        $PercentChange = Get-PercentageChange -Old $Previous -New $Current
        
        # Determine status with combined threshold (percentage AND absolute)
        $Status = "Normal"
        
        if ($Previous -gt 0) {
            # Require BOTH percentage threshold AND absolute threshold to trigger alert
            if ($PercentChange -ge $SpikeThresholdPercent -and $Change -ge $SpikeThresholdAbsolute) {
                $Status = "⚠ SPIKE"
            }
            elseif ($PercentChange -le -$SpikeThresholdPercent -and [math]::Abs($Change) -ge $SpikeThresholdAbsolute) {
                $Status = "⚠ DROP"
            }
        }
        elseif ($Current -ge $SpikeThresholdAbsolute) {
            # New event type with significant volume
            $Status = "⚠ NEW"
        }
        
        $EventName = Get-EventName -EventId $EventId
        
        $Result = [PSCustomObject]@{
            Timestamp = Get-Date -Format 'yyyy-MM-dd HH:mm:ss'
            Interval = $i
            EventID = $EventId
            EventName = $EventName
            PreviousCount = $Previous
            CurrentCount = $Current
            Change = $Change
            PercentChange = "$PercentChange%"
            EventsPerMin = [math]::Round($Current / $IntervalMinutes, 2)
            Status = $Status
        }
        
        $IntervalResults += $Result
        $AllResults += $Result
    }
    
    # Display results
    Show-Results -Results $IntervalResults -IntervalNum $i -TotalInt $TotalIntervals -EventsPerMinute $EventsPerMinute
    
    # Highlight anomalies
    $Anomalies = $IntervalResults | Where-Object { $_.Status -ne "Normal" }
    if ($Anomalies) {
        Write-Host "`n⚠ ANOMALIES DETECTED:" -ForegroundColor Red
        $Anomalies | ForEach-Object {
            $color = if ($_.Status -like "*SPIKE*" -or $_.Status -like "*NEW*") { "Red" } else { "Yellow" }
            Write-Host "  - $($_.EventName): $($_.Change) events ($($_.PercentChange)) - $($_.EventsPerMin) events/min" -ForegroundColor $color
        }
    }
    
    # Update tracking variables
    $PreviousCounts = $CurrentCounts
    $LastCheckTime = $CheckTime
    
    # Save to CSV
    $AllResults | Export-Csv -Path $OutputPath -NoTypeInformation -Force
}

# Final Summary
Write-Host "`n============================================" -ForegroundColor Green
Write-Host "Monitoring Complete!" -ForegroundColor Green
Write-Host "============================================" -ForegroundColor Green
Write-Host "Duration: $DurationHours hours" -ForegroundColor White
Write-Host "Total Intervals: $TotalIntervals" -ForegroundColor White
Write-Host "Results saved to: $OutputPath" -ForegroundColor White

# Generate summary statistics
Write-Host "`nSummary Statistics:" -ForegroundColor Cyan

$SummaryByEvent = $AllResults | Group-Object EventName | ForEach-Object {
    $eventData = $_.Group
    $avgPerInterval = [math]::Round(($eventData | Measure-Object -Property CurrentCount -Average).Average, 2)
    $avgPerMinute = [math]::Round($avgPerInterval / $IntervalMinutes, 2)
    
    [PSCustomObject]@{
        EventName = $_.Name
        TotalEvents = ($eventData | Measure-Object -Property CurrentCount -Sum).Sum
        AvgPerInterval = $avgPerInterval
        AvgPerMinute = $avgPerMinute
        MaxSpike = ($eventData | Measure-Object -Property CurrentCount -Maximum).Maximum
        SpikeCount = ($eventData | Where-Object { $_.Status -like "*SPIKE*" -or $_.Status -like "*NEW*" }).Count
    }
} | Sort-Object TotalEvents -Descending

$SummaryByEvent | Format-Table -AutoSize

$TotalAnomalies = ($AllResults | Where-Object { $_.Status -ne "Normal" }).Count
$TotalEvents = ($AllResults | Measure-Object -Property CurrentCount -Sum).Sum
$OverallAvgPerMin = [math]::Round($TotalEvents / ($DurationHours * 60), 2)

Write-Host "`nOverall Statistics:" -ForegroundColor Cyan
Write-Host "  Total Events Generated: $TotalEvents" -ForegroundColor White
Write-Host "  Average Rate: $OverallAvgPerMin events/minute" -ForegroundColor White
Write-Host "  Estimated Daily Volume: $([math]::Round($OverallAvgPerMin * 1440, 0)) events" -ForegroundColor White
Write-Host "  Estimated Daily Disk Usage: ~$([math]::Round(($OverallAvgPerMin * 1440 * 2) / 1024, 2)) MB (assuming 2KB/event)" -ForegroundColor White
Write-Host "  Total Anomalies Detected: $TotalAnomalies" -ForegroundColor $(if ($TotalAnomalies -gt 0) { "Red" } else { "Green" })

Write-Host "`nScript execution completed at $(Get-Date)" -ForegroundColor Green
```

**Usage Examples:**

```powershell
# Default: 15-minute intervals for 24 hours
.\Monitor-SysmonEvents.ps1

# Clear logs first for clean baseline
.\Monitor-SysmonEvents.ps1 -ClearLogFirst $true

# Shorter test: 10-minute intervals for 12 hours
.\Monitor-SysmonEvents.ps1 -IntervalMinutes 10 -DurationHours 12

# More sensitive detection (lower thresholds)
.\Monitor-SysmonEvents.ps1 -SpikeThresholdPercent 75 -SpikeThresholdAbsolute 250

# Week-long monitoring with 30-minute intervals
.\Monitor-SysmonEvents.ps1 -IntervalMinutes 30 -DurationHours 168
```

**Key Script Features:**

- **Time-based tracking** - Measures new events per interval, not cumulative log totals
- **Generation rate** - Shows events/minute for capacity planning
- **Dual spike detection** - Requires both percentage AND absolute thresholds to reduce false positives
- **Disk projection** - Estimates daily disk usage based on observed rates
- **CSV export** - Full data for analysis in Excel, Splunk, or other tools

**What to Look For:**

- **Sustained high rates** - If consistently above baseline thresholds for your system type
- **Unusual spikes** - 2x-3x increases during specific times (investigate what triggered it)
- **Unexpected event types** - Events firing that you didn't anticipate
- **Low/no volume** - Event types you expected to see but aren't appearing

#### False Positive Review

After the 24-hour volume assessment, spend time examining actual logged events to ensure your config logic is working as intended. Focus your review efficiently:

**Structured Review Process:**

1. **Prioritize high-volume events** - Start with event types generating the most volume (typically Event IDs 1, 3, 7, 13)
2. **Sample size** - Review at least 50-100 events per major event type, more if you see patterns
3. **Look for common patterns:**
    - Same source process/binary appearing repeatedly
    - Legitimate software behaving unexpectedly (updaters, management agents etc)
    - Overly broad rules catching benign activity
    - Rules not catching what you intended
4. **Common false positive sources to check:**
    - **Event ID 3 (Network):** Localhost connections, known management tools
    - **Event ID 11 (FileCreate):** Temp file churn, application caches
    - **Event ID 13 (Registry):** Windows Update, legitimate software settings
    - **Event ID 1 (Process):** Scheduled tasks, system maintenance
5. **Document findings** - Note which rules need exclusions and why

**Example Review Command:**

```powershell
# Review last 100 Process Creation events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 100 | 
    Where-Object { $_.Id -eq 1 } | 
    Format-List TimeCreated, Message

# Find most common source images for an event type
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" | 
    Where-Object { $_.Id -eq 3 } |
    ForEach-Object { ([xml]$_.ToXml()).Event.EventData.Data | 
        Where-Object { $_.Name -eq 'Image' } | 
        Select-Object -ExpandProperty '#text' } |
    Group-Object | Sort-Object Count -Descending | Select-Object -First 20
```

#### Intentional Event Triggering

If you have the capability, deliberately trigger events you expect your config to detect and verify they appear in the log. This validates your detection logic.

**Basic Event Triggers:**

```powershell
# Event ID 1 - Process Creation
notepad.exe

# Event ID 11 - File Creation (if monitoring that path)
New-Item -Path "C:\temp\test.txt" -ItemType File

# Event ID 13 - Registry Modification (if monitoring that key)
Set-ItemProperty -Path "HKCU:\Software\TestKey" -Name "TestValue" -Value "Test"

# Event ID 22 - DNS Query
nslookup google.com
```

**MITRE ATT&CK Technique Testing:**

If your config is designed to detect specific techniques, for example T1055 Process Injection, T1059 Command Line scripting, T1547 Persistence, consider using frameworks like Atomic Red Team to safely trigger and validate detection of those specific behaviours.

Document timestamps as you trigger events for easy cross-referencing in the logs.

#### Deployment Considerations

Finally, once you've assessed everything and are satisfied the new config works as it should, you should still have contingency plans in place should something go awry.

**Rollback Planning:**

```powershell
# Save current config before applying new one
Sysmon64.exe -c > backup_config_$(Get-Date -Format 'yyyyMMdd').xml

# If issues occur, revert:
Sysmon64.exe -c backup_config_20241010.xml
```

**Phased Deployment:**

Don't deploy to your entire environment at once. Consider a phased approach:

1. Test systems (1-5 per type) - 24-48 hours
2. Pilot group (10-50 systems) - 1 week
3. Staged rollout - 25% → 50% → 75% → 100% over 2-4 weeks

Monitor each phase for unexpected volume spikes or performance issues before expanding.

**Performance Monitoring:**

While Sysmon is efficient, overly aggressive configs can impact system performance. During testing, monitor:

```powershell
# Check Sysmon CPU usage
Get-Process Sysmon* | Select-Object Name, CPU, WorkingSet

# Typical: <2% CPU on workstation, may be higher on busy servers
```

If you see sustained high CPU usage (>5%), your config may be too aggressive and needs optimization.




## **Updating and Managing Configurations**

### **Updating an Existing Configuration**

As I already mentioned above, to update Sysmon's configuration after installation:

```cmd
Sysmon64.exe -c new_config.xml
```

The `-c` flag applies a new configuration file to the running Sysmon service. The service does not need to be restarted -configuration changes take effect immediately.

### **Confirming Configuration Has Been Applied**

To verify the current configuration:

```cmd
Sysmon64.exe -c
```

Running `Sysmon64.exe -c` with no argument displays the current active configuration, dumped to the console in XML format. You can redirect this to a file if you'd like to open it with another application:

```cmd
Sysmon64.exe -c > current_config.xml
```

Alternatively, query the Sysmon service configuration from the registry:

```powershell
Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\SysmonDrv\Parameters" -Name "Rules"
```

The `Rules` registry value contains the currently applied configuration as a binary blob.

### **Configuration Version Control**

Treat your Sysmon configuration as code:

1. Store it in version control (Git)
2. Document changes in commit messages
3. Test updates in a lab before production deployment
4. Maintain environment-specific configs if needed (dev, prod, etc.)



