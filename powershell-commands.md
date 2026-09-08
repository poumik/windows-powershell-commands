# Windows 11 PowerShell - System Administration & Management Commands

This repository serves as a comprehensive reference guide for essential Windows 11 PowerShell commands, covering system management, troubleshooting, software maintenance, and networking.

> **Note:** Most of the commands that modify system states (like updating software, managing services, or changing firewall rules) require running PowerShell as an Administrator (*Run as Administrator*).

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

---

## 1. Software Management and Updates (Winget)
Windows 11 includes the built-in Windows Package Manager (`winget`).

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
`PSWindowsUpdate` is a third-party module from the PowerShell Gallery, not a built-in cmdlet. It allows full management of Windows Updates via CLI.

```powershell
# Check if the module is already available on your system
Get-Module -ListAvailable PSWindowsUpdate

# Install the Windows Update module (requires Administrator)
Install-Module -Name PSWindowsUpdate -Force

# Check for available Windows updates
Get-WindowsUpdate

# Download and install all available updates
# WARNING: -AutoReboot will restart the computer without prompting!
Install-WindowsUpdate -AcceptAll -AutoReboot
```

---

## 3. System Repair and Maintenance

```powershell
# Scan and repair corrupted system files (System File Checker)
sfc /scannow

# Repair the Windows image from Windows Update if SFC fails (DISM)
DISM /Online /Cleanup-Image /RestoreHealth

# Empty the Recycle Bin without prompting for confirmation
Clear-RecycleBin -Force

# Update PowerShell's built-in help files
Update-Help -Force
```

---

## 4. System Information and Performance

```powershell
# Get comprehensive system info (OS version, BIOS, RAM, etc.)
Get-ComputerInfo

# Show how long the computer has been running (Uptime)
Get-Uptime

# List the top 10 processes consuming the most Memory (RAM)
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# List processes with the highest accumulated CPU time
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

---

## 5. Process and Service Management

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

```powershell
# Test whether a host responds (Ping replacement)
Test-Connection google.com

# Test a specific TCP port (Excellent for troubleshooting firewall/service issues)
Test-NetConnection google.com -Port 443

# Show DNS information for a domain
Resolve-DnsName google.com

# Flush the DNS cache
Clear-DnsClientCache

# List all IPv4 addresses assigned to this computer
Get-NetIPAddress -AddressFamily IPv4 | Select-Object IPAddress, InterfaceAlias

# Show physical and virtual network adapters
Get-NetAdapter

# Show active TCP connections
Get-NetTCPConnection

# Show the routing table
Get-NetRoute
```

---

## 7. Firewall Management

```powershell
# Show active firewall profiles (Domain, Private, Public)
Get-NetFirewallProfile

# Show all currently enabled firewall rules
Get-NetFirewallRule | Where-Object Enabled -eq True
```

---

## 8. Windows Defender & Security

```powershell
# Get Microsoft Defender status and signature versions
Get-MpComputerStatus

# Start a quick malware scan
Start-MpScan -ScanType QuickScan
```

---

## 9. Event Logs
Crucial for troubleshooting system crashes and application errors.

```powershell
# List all available event logs on the system
Get-WinEvent -ListLog *

# Show the latest 50 System events
Get-WinEvent -LogName System -MaxEvents 50

# Show the latest 50 Application events
Get-WinEvent -LogName Application -MaxEvents 50
```

---

## 10. Disk and Storage Management

```powershell
# List all volumes/drives and their free space
Get-Volume

# Run the TRIM command on an SSD to optimize storage blocks
Optimize-Volume -DriveLetter C -ReTrim -Verbose

# Show the health status of physical disks
Get-PhysicalDisk
```

---

## 11. Files & Folders

```powershell
# List files and folders in the current directory
Get-ChildItem

# Include hidden files
Get-ChildItem -Force

# Search recursively for a specific file ignoring access denied errors
Get-ChildItem -Path C:\ -Filter "example.txt" -Recurse -ErrorAction SilentlyContinue

# Copy, Move, and Rename files
Copy-Item "C:\source\file.txt" "C:\destination\"
Move-Item "C:\source\file.txt" "C:\destination\"
Rename-Item "C:\file.txt" "newfile.txt"

# Remove/Delete a file
Remove-Item "C:\file.txt"

# Show the text contents of a file
Get-Content "C:\file.txt"
```

---

## 12. Local User Account Management
*Note: Uses the `Microsoft.PowerShell.LocalAccounts` module. Best run in a 64-bit PowerShell environment.*

```powershell
# List all local users
Get-LocalUser

# Create a new local user
New-LocalUser -Name "TestUser" -Description "Temporary account" -NoPassword

# Add a user to the Administrators group
Add-LocalGroupMember -Group "Administrators" -Member "TestUser"
```

---

## 13. Environment & PowerShell Basics

```powershell
# Show the current PowerShell version
$PSVersionTable

# Show current directory path
Get-Location

# Change directory
Set-Location C:\Temp

# Find a command by name
Get-Command <command_name>

# Find commands related to a specific topic
Get-Command *network*

# Get detailed help for a command (includes examples)
Get-Help <command_name> -Detailed

# Show all available aliases (e.g., 'cd', 'ls', 'dir' equivalents)
Get-Alias
```
