# Windows 11 PowerShell - System Administration & Management Commands

A comprehensive quick-reference guide for essential Windows 11 system administration commands, covering system management, troubleshooting, software maintenance, and networking.

> **Scope note:** This guide includes both PowerShell cmdlets and native Windows tools used in system administration (`DISM`, `sfc`, `powercfg`, `winget`, `net`, `usoclient`). Not every example is a PowerShell-native cmdlet — native tools are called from PowerShell the same way, but they are separate programs with their own syntax.

> **Note:** Most of the commands that modify system states (like updating software, managing services, or changing firewall rules) require running PowerShell as an Administrator (*Run as Administrator*).

> **Caution:** Commands marked **WARNING** in this guide can permanently change system state, alter networking, reduce security coverage, or destroy data. Use with caution, and preview high-risk changes with `-WhatIf` where available.

> **Environment note:** This guide is intended for supported Windows 11 systems. Some commands require administrator rights, network access, optional modules, or specific Windows builds. Managed/enterprise images, LTSC builds, and stripped-down or offline installations may differ from consumer installs — `winget`, WinRM, Defender cmdlets, or `PSWindowsUpdate` may be absent or blocked on such systems.

> **Placeholder note:** In examples, `<name>`, `<command_name>`, and similar angle-bracket tokens are placeholders. Do not type the angle brackets literally in PowerShell. Use a real command name or a variable.

## Compatibility: PowerShell 5.1 vs 7+
Windows 11 ships with **Windows PowerShell 5.1** as the default. **PowerShell 7+** is a separate install (`winget install Microsoft.PowerShell`) that adds newer cmdlets and improvements. Some commands are only available in PowerShell 6/7+, while others depend on Windows-only modules or features that may not be present in all builds or managed environments.

- Commands marked **(PS 6+)** or **(PS 7+)** will not run in 5.1 (e.g. `Get-Uptime` was introduced in PowerShell 6.0).
- Check your version with `$PSVersionTable.PSVersion`.
- Each section heading is followed by a **Permission** tag: **Administrator**, **None (read-only)**, or **Mixed** (queries are read-only, changes need elevation).
- Some Windows-only modules (`Get-Net*`, `Get-Mp*`, `Get-BitLocker*`, `Get-ScheduledTask`, `Microsoft.PowerShell.LocalAccounts`, etc.) may be unavailable, behave differently, or require compatibility loading (`Import-Module ... -UseWindowsPowerShell`) in PowerShell 7+. If a cmdlet is missing, verify the module is available, or run the command in Windows PowerShell 5.1.

## Contents
1. [Software Management and Updates (Winget)](#1-software-management-and-updates-winget)
2. [Windows Update Management (PSWindowsUpdate)](#2-windows-update-management-pswindowsupdate)
3. [System Repair and Maintenance](#3-system-repair-and-maintenance)
4. [System Information and Performance](#4-system-information-and-performance)
5. [Process and Service Management](#5-process-and-service-management)
6. [Networking](#6-networking)
7. [Firewall Management](#7-firewall-management)
8. [Windows Defender & Security](#8-windows-defender--security)
9. [Event Logs](#9-event-logs)
10. [Disk and Storage Management](#10-disk-and-storage-management)
11. [Files & Folders](#11-files--folders)
12. [Local User Account Management](#12-local-user-account-management)
13. [Environment & PowerShell Basics](#13-environment--powershell-basics)
14. [Scheduled Tasks](#14-scheduled-tasks)
15. [BitLocker Management](#15-bitlocker-management)
16. [Archives & File Hashing](#16-archives--file-hashing)
17. [Restart, Shutdown & Power Reports](#17-restart-shutdown--power-reports)
18. [PowerShell Remoting](#18-powershell-remoting)

## Quick Reference
| Purpose | Command |
| :--- | :--- |
| **PowerShell version** | `$PSVersionTable` |
| **Find command** | `Get-Command <name>` |
| **Get help** | `Get-Help <name>` |
| **Processes** | `Get-Process` |
| **Services** | `Get-Service` |
| **IP addresses** | `Get-NetIPAddress` |
| **Network test** | `Test-NetConnection` |
| **Event logs** | `Get-WinEvent` |
| **Installed software** | `winget list` |
| **Windows updates** | `Get-WindowsUpdate` |
| **System info** | `Get-ComputerInfo` |
| **Scheduled tasks** | `Get-ScheduledTask` |
| **BitLocker status** | `Get-BitLockerVolume` |
| **Create ZIP archive** | `Compress-Archive` |
| **Remote session** | `Enter-PSSession` |

---

## 1. Software Management and Updates (Winget)

> **Permission:** None — some installers may request elevation themselves.

Windows 11 includes the built-in Windows Package Manager (`winget`). Check availability first:

```powershell
# Check that winget is available
winget --version
```

```powershell
# Search for a specific program to install
winget search "VLC"

# Install a program
winget install "VLC media player"

# Install a program silently in the background
winget install "Mozilla Firefox" --silent

# Install and automatically accept license agreements
winget install "Program Name" --accept-source-agreements --accept-package-agreements

# List all installed programs on the computer
winget list

# Filter the list of installed programs by a keyword
winget list "keyword"

# Check for installed programs that have updates available
winget upgrade

# Update a specific program
winget upgrade "Program Name"

# Update all packages that have updates available
winget upgrade --all

# Uninstall a program
winget uninstall "Program Name"
```

---

## 2. Windows Update Management (PSWindowsUpdate)

> **Permission:** Administrator

`PSWindowsUpdate` is a third-party module from the PowerShell Gallery, not a built-in cmdlet. It allows full management of Windows Updates via CLI. Installing it requires internet access, and it may not be available — or installation may be blocked — on managed, offline, or locked-down systems. For a complete install → verify → reboot workflow, see [windows-update-in-powershell.md](windows-update-in-powershell.md).

```powershell
# Check if the module is already available on your system
Get-Module -ListAvailable PSWindowsUpdate

# If Install-Module fails on PowerShell 5.1, enable TLS 1.2 first, then bootstrap NuGet
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Install-PackageProvider -Name NuGet -Force

# Install the Windows Update module (requires Administrator)
Install-Module -Name PSWindowsUpdate -Force

# Import the module into the current session
Import-Module PSWindowsUpdate

# Check for available Windows updates
Get-WindowsUpdate

# Download and install all available updates
# WARNING: -AutoReboot will restart the computer without prompting!
Install-WindowsUpdate -AcceptAll -AutoReboot

# Install updates but do not auto-reboot
Install-WindowsUpdate -AcceptAll -IgnoreReboot

# Show Windows Update history
Get-WUHistory

# Check Windows Update installer status
Get-WUInstallerStatus

# Check if a reboot is required to complete installed updates (-Silent returns plain $true/$false)
# PSWindowsUpdate cmdlet — requires the module installed and imported
Get-WURebootStatus -Silent

# Hide an update by KB article ID
Hide-WindowsUpdate -KBArticleID KB123456

# Uninstall an update by KB article ID (PSWindowsUpdate cmdlet; canonical name; alias: Get-WUUninstall)
Remove-WindowsUpdate -KBArticleID KB123456
```

---

## 3. System Repair and Maintenance

> **Permission:** Mixed — `DISM` and `sfc` require an elevated session; `Clear-RecycleBin` clears the current user's Recycle Bin and works unelevated; `Update-Help` usually works unelevated but may need elevation and internet access.

```powershell
# Repair the Windows image from Windows Update first (DISM - recommended before SFC)
# Docs: https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/repair-a-windows-image
DISM /Online /Cleanup-Image /RestoreHealth

# Scan and repair corrupted system files (System File Checker - run after DISM)
sfc /scannow

# PowerShell-native equivalent of the DISM command above
# Note: Repair-WindowsImage is a Windows PowerShell 5.1 cmdlet; it is not available in PowerShell 7+.
Repair-WindowsImage -Online -RestoreHealth

# Empty the Recycle Bin without prompting for confirmation
Clear-RecycleBin -Force

# Update PowerShell's built-in help files
# Note: often requires elevation and internet access; may fail if help files are in use.
Update-Help -Force
```

---

## 4. System Information and Performance

> **Permission:** None (read-only)

```powershell
# Get comprehensive system info (OS version, BIOS, RAM, etc.)
# Note: Get-ComputerInfo can be slow. For targeted info, use Get-CimInstance.
Get-ComputerInfo

# Faster targeted alternatives
Get-CimInstance Win32_OperatingSystem
Get-CimInstance Win32_BIOS
Get-CimInstance Win32_Processor
Get-CimInstance Win32_LogicalDisk -Filter "DriveType=3"
Get-CimInstance Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True"
Get-CimInstance Win32_QuickFixEngineering

# Show installed hotfixes
Get-HotFix

# Show how long the computer has been running (Uptime) - PowerShell 6 or newer
Get-Uptime

# Show only the last boot time (works in Windows PowerShell 5.1)
(Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# Uptime as a duration - also works in Windows PowerShell 5.1
(Get-Date) - (Get-CimInstance Win32_OperatingSystem).LastBootUpTime

# List the top 10 processes consuming the most Memory (RAM)
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# List processes with the highest accumulated CPU time
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Performance counters
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 1 -MaxSamples 5
Get-Counter '\Memory\Available MBytes'
Get-Counter '\PhysicalDisk(_Total)\% Disk Time'
```

---

## 5. Process and Service Management

> **Permission:** Mixed — queries are read-only; `Stop-*`, `Start-*`, `Restart-*`, and `Set-*` require Administrator.

```powershell
# Find a running process by name
Get-Process -Name chrome

# Forcefully close a stuck process
Stop-Process -Name chrome -Force

# Find a service by name
Get-Service -Name Spooler

# List all background services that are currently running
Get-Service | Where-Object Status -eq "Running"

# Stop, Start, or Restart a service
Stop-Service -Name Spooler
Start-Service -Name Spooler
Restart-Service -Name Spooler -Force

# Set a service to start automatically on boot
Set-Service -Name Spooler -StartupType Automatic
```

---

## 6. Networking

> **Permission:** Mixed — queries and `Clear-DnsClientCache` (≈ `ipconfig /flushdns`) work unelevated; `Set-DnsClientServerAddress` requires Administrator. Confirm the interface alias with `Get-NetAdapter` before changing it.

```powershell
# Test whether a host responds (Ping replacement)
Test-Connection google.com

# Test a specific TCP port (Excellent for troubleshooting firewall/service issues)
Test-NetConnection google.com -Port 443

# Show DNS information for a domain
Resolve-DnsName google.com

# Query specific DNS record types
Resolve-DnsName google.com -Type MX

# Flush the DNS cache
Clear-DnsClientCache

# List all IPv4 addresses assigned to this computer
Get-NetIPAddress -AddressFamily IPv4 | Select-Object IPAddress, InterfaceAlias

# Show IP configuration
Get-NetIPConfiguration

# Show DNS server addresses
Get-DnsClientServerAddress

# Set DNS server addresses for an interface (requires Administrator; verify the interface alias
# with Get-NetAdapter first — changing the wrong interface can interrupt connectivity)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 1.1.1.1,8.8.8.8

# Restore DNS server addresses to automatic (DHCP)
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ResetServerAddresses

# Show physical and virtual network adapters
Get-NetAdapter

# Show active TCP connections
Get-NetTCPConnection

# Show listening TCP ports
Get-NetTCPConnection -State Listen

# Show UDP endpoints
Get-NetUDPEndpoint

# Show the routing table
Get-NetRoute

# Trace a route
Test-NetConnection google.com -TraceRoute
```

---

## 7. Firewall Management

> **Permission:** Mixed — queries are read-only; modifying rules requires Administrator.

```powershell
# Show active firewall profiles (Domain, Private, Public)
Get-NetFirewallProfile

# Show all currently enabled firewall rules (native parameter - faster than Where-Object)
Get-NetFirewallRule -Enabled True

# Create an inbound firewall rule (changes security posture — verify port and scope before opening)
New-NetFirewallRule -DisplayName "Allow HTTP" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow

# Remove a firewall rule by display name (can disconnect services relying on it)
Remove-NetFirewallRule -DisplayName "Allow HTTP"

# Enable or disable firewall profiles (disabling a profile reduces protection on that network)
Set-NetFirewallProfile -Profile Domain,Private,Public -Enabled True
```

---

## 8. Windows Defender & Security

> **Permission:** Mixed — status queries are read-only; scans, signature updates, and exclusions require Administrator.

```powershell
# Get Microsoft Defender status and signature versions
Get-MpComputerStatus

# Get Defender preferences and exclusions
Get-MpPreference

# Run a quick Defender scan
Start-MpScan -ScanType QuickScan

# Show recent threat detections
Get-MpThreatDetection

# Add a Defender exclusion folder (requires Administrator)
# WARNING: use exclusions sparingly — an excluded path is no longer scanned, which reduces
# protection if the location is shared or untrusted content is written there.
Add-MpPreference -ExclusionPath "C:\Tools"

# Update the Defender malware signature database
Update-MpSignature
```

---

## 9. Event Logs
Crucial for troubleshooting system crashes and application errors.

> **Permission:** None for most logs (some logs like Security require Administrator).

```powershell
# List all available event logs on the system
Get-WinEvent -ListLog *

# Show the latest 50 System events
Get-WinEvent -LogName System -MaxEvents 50

# Show the latest 50 Application events
Get-WinEvent -LogName Application -MaxEvents 50

# Filter System log for critical and error events (Level 1 = Critical, 2 = Error)
Get-WinEvent -FilterHashtable @{ LogName = 'System'; Level = 1, 2 } -MaxEvents 50

# Show recent critical/error/warning events from the last 24 hours
Get-WinEvent -FilterHashtable @{ LogName = 'System'; Level = 1, 2, 3; StartTime = (Get-Date).AddDays(-1) }

# Filter by specific event IDs (IDs are only unique within a log/provider)
Get-WinEvent -FilterHashtable @{ LogName = 'System'; Id = 41, 6008 } -MaxEvents 50

# Filter by event provider (useful because IDs collide across sources)
Get-WinEvent -FilterHashtable @{ LogName = 'System'; ProviderName = 'Microsoft-Windows-Kernel-Power'; Id = 41 } -MaxEvents 20
```

Common event IDs worth checking:

| Event ID | Meaning |
| :--- | :--- |
| `6008` | Unexpected shutdown |
| `41` | Kernel power |
| `1001` | Bugcheck |
| `19`, `20` | Windows Update |
| `7000`, `7009`, `7031`, `7034` | Service failures |
| `7`, `11`, `15`, `51`, `55`, `98`, `129`, `153` | Disk errors |
| `1000`, `1002`, `1026` | Application errors |

---

## 10. Disk and Storage Management

> **Permission:** Administrator for `Optimize-Volume` and `Repair-Volume`; queries are read-only.

```powershell
# List all volumes/drives and their free space
Get-Volume

# List physical disks
Get-Disk

# List partitions
Get-Partition

# Run the TRIM command on an SSD to optimize storage blocks
Optimize-Volume -DriveLetter C -ReTrim -Verbose

# Show the health status of physical disks
Get-PhysicalDisk

# Show storage reliability counters
Get-StorageReliabilityCounter

# Scan a volume for file system errors (read-only check, no repair)
Repair-Volume -DriveLetter C -Scan

# WARNING: the following commands destroy data. Before continuing, confirm the intended disk,
# partition, and volume with Get-Disk, Get-Partition, and Get-Volume — a wrong number wipes the wrong disk.

# Initialize, partition, and format a new disk
# Note: -AssignDriveLetter picks the next free letter (not guaranteed to be D:),
# so pipe the new partition into Format-Volume instead of formatting a guessed drive letter.
# Initialize a new blank disk — wipes the target disk's existing partition table and data
Initialize-Disk -Number <disk-number>
New-Partition -DiskNumber <disk-number> -UseMaximumSize -AssignDriveLetter | Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data"

# Resize a partition
# Warning: -Size is the resulting size — a value smaller than the current size shrinks the
# partition (incorrect use risks data loss); verify the target partition first.
Resize-Partition -DriveLetter C -Size 100GB
```

---

## 11. Files & Folders

> **Permission:** None for most operations (system paths and deleting in-use files require elevation).

```powershell
# List files and folders in the current directory
Get-ChildItem

# Include hidden files
Get-ChildItem -Force

# Test whether a path exists
Test-Path "C:\Temp"

# Create a new folder or file
New-Item -ItemType Directory -Path "C:\Temp\NewFolder"
New-Item -ItemType File -Path "C:\Temp\newfile.txt"

# Search recursively for a specific file ignoring access denied errors
# Warning: searching all of C:\ can be slow.
Get-ChildItem -Path C:\ -Filter "example.txt" -Recurse -ErrorAction SilentlyContinue

# Copy, Move, and Rename files
Copy-Item "C:\source\file.txt" "C:\destination\"
Move-Item "C:\source\file.txt" "C:\destination\"
Rename-Item "C:\file.txt" "newfile.txt"

# Remove/Delete a file or folder
Remove-Item "C:\file.txt"

# Destructive: permanently removes a folder and everything in it, without prompting
Remove-Item "C:\Temp\Folder" -Recurse -Force

# Preview a destructive operation before running it (-WhatIf shows what would happen)
Remove-Item "C:\Temp\Folder" -Recurse -Force -WhatIf

# Show the text contents of a file
Get-Content "C:\file.txt"

# Sum the size of files in a folder tree
Get-ChildItem -Path C:\Temp -Recurse -File | Measure-Object -Property Length -Sum

# List files larger than 1 GB on a drive (find disk space hogs)
Get-ChildItem -Path C:\ -Recurse -File -ErrorAction SilentlyContinue |
    Where-Object Length -gt 1GB |
    Sort-Object Length -Descending |
    Select-Object -First 10 -Property FullName, @{ Name = 'SizeGB'; Expression = { [math]::Round($_.Length / 1GB, 2) } }
```

---

## 12. Local User Account Management

> **Permission:** Administrator for changes; queries are read-only.
>
> The `Microsoft.PowerShell.LocalAccounts` cmdlets (`Get-LocalUser`, `New-LocalUser`, `Add-LocalGroupMember`, ...) are built into Windows 11, including 24H2 and later. They are reliable in **Windows PowerShell 5.1**. In **PowerShell 7+**, they may not be available depending on install. If missing, use the classic `net` commands or CIM alternatives.

```powershell
# List all local users
Get-LocalUser

# Create a new local user (lab/testing only: -NoPassword creates a passwordless account, which is a security risk)
New-LocalUser -Name "TestUser" -Description "Temporary account" -NoPassword

# Create a local user with a password (normal use)
$password = Read-Host "Enter password" -AsSecureString
New-LocalUser -Name "ExampleUser" -Password $password

# Add a user to the Administrators group (grants full admin rights — verify the account first)
Add-LocalGroupMember -Group "Administrators" -Member "TestUser"

# CIM alternative that works in more PowerShell versions
Get-CimInstance Win32_UserAccount
```

Classic `net` command equivalents (work on any Windows version):

```powershell
# List all local users
net user

# Create a new local user (prompts for password)
net user TestUser * /add

# Add a user to the Administrators group (grants full admin rights)
net localgroup Administrators TestUser /add
```

---

## 13. Environment & PowerShell Basics

> **Permission:** None (read-only), except `Set-ExecutionPolicy` which may require Administrator or affect the current user only.

```powershell
# Show the current PowerShell version
$PSVersionTable

# Show current directory path
Get-Location

# Change directory
Set-Location C:\Temp

# Find a command by name
Get-Command -Name "Get-Process"

# Find commands related to a specific topic
Get-Command *network*

# Get detailed help for a command (includes examples)
Get-Help -Name "Get-Process" -Detailed

# Open help online
Get-Help -Name "Get-Process" -Online

# Explore object properties and methods
Get-Process | Get-Member

# Show all available aliases (e.g., 'cd', 'ls', 'dir' equivalents)
Get-Alias

# List environment variables (PATH, TEMP, etc.)
Get-ChildItem Env:

# Show a single environment variable
$env:PATH

# Check execution policy
Get-ExecutionPolicy -List

# Set execution policy for the current user
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser

# Export output
Get-Process | Export-Csv -Path "C:\Temp\processes.csv" -NoTypeInformation
Get-Process | ConvertTo-Json -Depth 3 | Out-File "C:\Temp\processes.json"
Get-Process | Out-GridView

# Tee output to both console and file
Get-Process | Tee-Object -FilePath "C:\Temp\processes.txt"
```

---

## 14. Scheduled Tasks

> **Permission:** Mixed — queries are read-only; changing or running tasks usually requires Administrator.

```powershell
# List all scheduled tasks
Get-ScheduledTask

# List tasks that are currently in a running state
Get-ScheduledTask | Where-Object State -eq "Running"

# Find a task by name
Get-ScheduledTask -TaskName "MyTask"

# Show the last run time and result of a task
Get-ScheduledTaskInfo -TaskName "MyTask"

# Run a scheduled task manually
Start-ScheduledTask -TaskName "MyTask"

# Disable a task so it no longer triggers automatically
Disable-ScheduledTask -TaskName "MyTask"

# Re-enable a disabled task
Enable-ScheduledTask -TaskName "MyTask"

# Create and register a scheduled task
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" -Argument "-NoProfile -Command Get-Date"
$trigger = New-ScheduledTaskTrigger -Daily -At 3am
Register-ScheduledTask -TaskName "DailyDate" -Action $action -Trigger $trigger -Description "Runs daily at 3 AM"
```

---

## 15. BitLocker Management

> **Permission:** Administrator (enabling/modifying) — status queries work unelevated.

```powershell
# Show BitLocker status and protection state for all volumes
Get-BitLockerVolume

# Show detailed key protectors for the OS drive
(Get-BitLockerVolume -MountPoint C).KeyProtector

# Enable BitLocker on the C: drive with a recovery password
# WARNING: always back up the recovery key in a secure location (print, USB, or cloud) BEFORE
# enabling BitLocker. If the key is lost, the data on the encrypted drive is unrecoverable.
# Do not keep the only copy on the encrypted computer. In organizational environments, key
# escrow may be configured through Microsoft Entra ID or management policy — verify the key
# is actually backed up before relying on it.
Enable-BitLocker -MountPoint C -RecoveryPasswordProtector

# Back up the recovery password to Microsoft Entra ID (work/school accounts only — this cmdlet
# does not upload to a personal Microsoft account; personal-account escrow applies to the
# automatic "device encryption" feature, not to Enable-BitLocker).
# Note: requires an existing recovery-password protector on the volume — if $rp is empty,
# the backup cannot run.
$BLV = Get-BitLockerVolume -MountPoint C
$rp = $BLV.KeyProtector | Where-Object KeyProtectorType -eq 'RecoveryPassword'
BackupToAAD-BitLockerKeyProtector -MountPoint C -KeyProtectorId $rp.KeyProtectorId

# Pause BitLocker protection temporarily (e.g., for a BIOS update)
Suspend-BitLocker -MountPoint C -RebootCount 1

# Resume BitLocker protection
Resume-BitLocker -MountPoint C
```

---

## 16. Archives & File Hashing

> **Permission:** None for your own files.

> **Note:** `Compress-Archive` also ignores hidden files and folders. For hidden files or archives with very large content, use a dedicated archiving tool or the .NET `System.IO.Compression` API instead.

```powershell
# Compress a folder into a ZIP archive
# Note: Compress-Archive uses System.IO.Compression.ZipArchive, which limits files to 2 GB
# (applies to Windows PowerShell 5.1 and PowerShell 7+ alike).
Compress-Archive -Path "C:\source\folder" -DestinationPath "C:\backup\archive.zip" -Force

# Add/refresh files in an existing archive
Compress-Archive -Path "C:\source\folder\*" -Update -DestinationPath "C:\backup\archive.zip"

# Extract a ZIP archive
Expand-Archive -Path "C:\backup\archive.zip" -DestinationPath "C:\extracted" -Force

# Compute the SHA256 hash of a file (verify downloads)
Get-FileHash "C:\downloads\setup.exe" -Algorithm SHA256

# Compare a downloaded file against a known checksum
(Get-FileHash "C:\downloads\setup.exe" -Algorithm SHA256).Hash -eq "EXPECTED_HASH"
```

---

## 17. Restart, Shutdown & Power Reports

> **Permission:** Mixed — local restart/shutdown works for standard users; remote restarts require appropriate permissions (DCOM by default in Windows PowerShell 5.1, WSMan in PowerShell 7+); the `powercfg` reports shown (`/batteryreport`, `/energy`, `/list`) are not documented by Microsoft as requiring elevation — run an elevated session if a report fails.

```powershell
# Restart the computer (add -Force to force close applications without saving — unsaved work is lost)
Restart-Computer -Force

- Shut down the computer
Stop-Computer -Force

# Restart or shut down a remote computer
# Note: uses DCOM in Windows PowerShell 5.1 (remoting not required); uses WSMan in PowerShell 7+ (WinRM required)
Restart-Computer -ComputerName "PC01" -Force

# Generate a battery health report (laptops) - writes an HTML file
powercfg /batteryreport /output "C:\Temp\battery-report.html"

# Analyze energy efficiency issues (runs a 60-second trace; run when the computer is idle)
# If the report fails unelevated, run from an elevated session.
powercfg /energy /output "C:\Temp\energy-report.html"

# Show all configured power plans and the active one
powercfg /list
```

---

## 18. PowerShell Remoting

> **Permission:** Administrator (to enable remoting and for most remote operations)

> **Caveat:** Remote administration depends on more than the command itself — it may require WinRM service configuration, matching firewall rules (WinRM HTTP/HTTPS inbound), TrustedHosts settings, and network profile configuration, especially on workgroup systems or enterprise-managed networks where domain policies or firewalls can block it. `Enable-PSRemoting -Force` does not guarantee remoting works in every environment. Requirements: [about_Remote_Requirements](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_remote_requirements).

```powershell
# Enable PowerShell Remoting on the local machine (run on the target computer)
Enable-PSRemoting -Force

# Same, but also for machines on a Public network profile (e.g. client SKUs)
Enable-PSRemoting -SkipNetworkProfileCheck -Force

# Show the TrustedHosts list (needed for workgroup remoting, where Kerberos is unavailable)
Get-Item WSMan:\localhost\Client\TrustedHosts

# Add a computer to TrustedHosts (warning: reduces authentication security; never set to "*")
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "PC01" -Force

# Open an interactive remote session on another computer
Enter-PSSession -ComputerName "PC01"

# Exit the remote session
Exit-PSSession

# Create a persistent remote session
$session = New-PSSession -ComputerName "PC01"

# List active remote sessions
Get-PSSession

# Run a command remotely on one or more computers (no interactive session)
Invoke-Command -ComputerName "PC01", "PC02" -ScriptBlock { Get-Volume }

# Run commands on many computers in parallel
Invoke-Command -ComputerName (Get-Content "C:\Temp\computers.txt") -ScriptBlock { Get-Volume }

# Run remote command as a background job
$job = Invoke-Command -ComputerName "PC01" -ScriptBlock { Get-Process } -AsJob

# Retrieve the results of a remote background job
Receive-Job -Job $job

# Copy files to and from a remote session
Copy-Item -Path "C:\Local\file.txt" -Destination "C:\Remote\" -ToSession $session
Copy-Item -Path "C:\Remote\file.txt" -Destination "C:\Local\" -FromSession $session

# Remove a remote session
Remove-PSSession -Session $session
```
