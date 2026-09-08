# Windows 11 PowerShell - System Administration & Management Commands

This document contains the most essential PowerShell commands for managing, updating, and troubleshooting Windows 11 and its installed software.

> **Note:** Most of the commands that modify the system require running PowerShell as an Administrator (*Run as Administrator*).

---

## 1. Software Management and Updates (Winget)
Windows 11 includes the built-in Windows Package Manager (`winget`), which is the most powerful way to manage installed software directly from PowerShell.

```powershell
# Search for a specific program to install (e.g., VLC)
winget search "VLC"

# Install a program
winget install "VLC media player"

# Install a program silently in the background (no setup windows)
winget install "Mozilla Firefox" --silent

# Install and automatically accept license agreements (useful for scripts)
winget install "Program Name" --accept-source-agreements --accept-package-agreements

# List all installed programs on the computer
winget list

# Filter the list of installed programs by a specific keyword
winget list "keyword"

# Check for installed programs that have updates available
winget upgrade

# Update a specific program to its latest version
winget upgrade "Program Name"

# UPDATE ALL installed programs to their latest versions at once (Highly recommended!)
winget upgrade --all

# Uninstall a program
winget uninstall "Program Name"
```

---

## 2. Windows Update Management (PSWindowsUpdate)
While Windows Update has a GUI, it can be fully managed via PowerShell by installing the `PSWindowsUpdate` module.

```powershell
# 1. Install the Windows Update module (only required once)
Install-Module -Name PSWindowsUpdate -Force

# 2. Check for available Windows updates
Get-WindowsUpdate

# 3. Download and install all available updates, and allow auto-reboot if necessary
Install-WindowsUpdate -AcceptAll -AutoReboot
```

---

## 3. System Repair and Maintenance
If Windows is crashing or running slow, these commands scan and repair core operating system files.

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
Quick ways to check system status and resource usage.

```powershell
# Get comprehensive system info (OS version, BIOS, RAM, etc.)
Get-ComputerInfo

# Show how long the computer has been running (Uptime)
Get-Uptime

# List the top 10 processes consuming the most Memory (RAM)
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 10

# List the top 10 processes consuming the most CPU
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10
```

---

## 5. Process and Service Management
Manage frozen applications and background services.

```powershell
# Find a running process by name (e.g., Chrome)
Get-Process -Name chrome

# Forcefully close a stuck process
Stop-Process -Name chrome -Force

# List all background services that are currently running
Get-Service | Where-Object Status -eq "Running"

# Restart a specific service (e.g., Print Spooler if printing is stuck)
Restart-Service -Name Spooler -Force
```

---

## 6. Network Management and Troubleshooting
Commands for fixing and analyzing network connections.

```powershell
# Flush the DNS cache (helps if specific websites won't load)
Clear-DnsClientCache

# List all IPv4 addresses assigned to this computer
Get-NetIPAddress -AddressFamily IPv4 | Select-Object IPAddress, InterfaceAlias

# Advanced network connection test (Ping replacement, checks routing and ports)
Test-NetConnection -ComputerName google.com

# Restart a specific network adapter (e.g., Wi-Fi)
Restart-NetAdapter -Name "Wi-Fi"
```

---

## 7. Disk and Storage Management
Check drive health and optimize storage.

```powershell
# List all volumes/drives and their free space
Get-Volume

# Optimize SSD drive (runs the TRIM command for the C: drive)
Optimize-Volume -DriveLetter C -ReTrim -Verbose

# Show the health status of physical disks
Get-PhysicalDisk
```

---

## 8. Local User Account Management
Manage local Windows 11 accounts (Requires Pro or Enterprise editions to work fully).

```powershell
# List all local users
Get-LocalUser

# Create a new local user
New-LocalUser -Name "TestUser" -Description "Temporary account" -NoPassword

# Add a user to the Administrators group
Add-LocalGroupMember -Group "Administrators" -Member "TestUser"
```
