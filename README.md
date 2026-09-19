# Windows 11 PowerShell Commands Guide

A quick-reference guide for essential Windows 11 PowerShell commands, covering system management, troubleshooting, software maintenance, and networking. Aimed at administrators and power users.

📄 **The guide:** [`powershell-commands.md`](powershell-commands.md) — all commands in one categorized document.

## Getting Started

1. **Open PowerShell** — press <kbd>Win</kbd> + <kbd>X</kbd> and choose **Terminal (Admin)**, or search for "PowerShell", right-click, and select *Run as Administrator*.
2. **Check your version** — Windows 11 ships with Windows PowerShell 5.1 by default:
   ```powershell
   $PSVersionTable.PSVersion
   ```
3. **Optional — install PowerShell 7+** for newer cmdlets and improvements:
   ```powershell
   winget install Microsoft.PowerShell
   ```
   > **Note:** `winget` may not be present on every Windows 11 image (e.g., LTSC or debloated builds). Check availability first with `winget --version`.

   Commands marked **(PS 7+)** in the guide only run on PowerShell 7 or newer.

   > **Compatibility note:** Some Windows-only modules (`Get-Net*`, `Get-Mp*`, `Get-BitLocker*`, `Get-ScheduledTask`, `Microsoft.PowerShell.LocalAccounts`, etc.) may not load in PowerShell 7+. If a command is missing in PS 7+, try running it in Windows PowerShell 5.1, or verify the module is available for your PowerShell version.
4. **Mind the permission tags** — each section of the guide is tagged so you know what needs elevation:

   | Tag | Meaning |
   | :--- | :--- |
   | **Administrator** | Run from an elevated PowerShell session |
   | **None (read-only)** | Safe to run unelevated |
   | **Mixed** | Queries are read-only; changes need elevation |

> **Note:** The Windows Update section uses [PSWindowsUpdate](https://www.powershellgallery.com/packages/PSWindowsUpdate), a third-party module from the PowerShell Gallery — the guide includes installation steps.

### Windows Update workflow

For **any** Windows Update (monthly patches or major OS upgrades), follow this order to watch live progress and avoid premature reboots — full details in [`windows-update-in-powershell.md`](windows-update-in-powershell.md):

1. **Install** (from an elevated session):
   - Standard updates — installs everything with live verbose logging, no auto-reboot:
     ```powershell
     Install-WindowsUpdate -AcceptAll -Verbose
     ```
   - Major OS upgrades (e.g. Insider Previews) — trigger the native Windows Update engine directly:
     ```powershell
     usoclient StartInteractiveScan
     ```
     > **Caveat:** `usoclient` is built into Windows but Microsoft does not document it officially. To watch progress, tail the CBS log from a second window:
     > `Get-Content "C:\Windows\Logs\CBS\CBS.log" -Wait -Tail 20`
2. **Verify the reboot status** before restarting — returns plain `$true`/`$false`:
   ```powershell
   Get-WURebootStatus -Silent
   ```
   - `$true` — a reboot is required to complete the installation. Proceed.
   - `$false` — no reboot is pending; the installation is complete without one.
3. **Reboot** once the check confirms it is required:
   ```powershell
   Restart-Computer -Force
   ```

## Contents

| # | Category |
| :-: | :--- |
| 1 | [Software Management and Updates (Winget)](powershell-commands.md#1-software-management-and-updates-winget) |
| 2 | [Windows Update Management (PSWindowsUpdate)](powershell-commands.md#2-windows-update-management-pswindowsupdate) |
| 3 | [System Repair and Maintenance](powershell-commands.md#3-system-repair-and-maintenance) |
| 4 | [System Information and Performance](powershell-commands.md#4-system-information-and-performance) |
| 5 | [Process and Service Management](powershell-commands.md#5-process-and-service-management) |
| 6 | [Networking](powershell-commands.md#6-networking) |
| 7 | [Firewall Management](powershell-commands.md#7-firewall-management) |
| 8 | [Windows Defender & Security](powershell-commands.md#8-windows-defender--security) |
| 9 | [Event Logs](powershell-commands.md#9-event-logs) |
| 10 | [Disk and Storage Management](powershell-commands.md#10-disk-and-storage-management) |
| 11 | [Files & Folders](powershell-commands.md#11-files--folders) |
| 12 | [Local User Account Management](powershell-commands.md#12-local-user-account-management) |
| 13 | [Environment & PowerShell Basics](powershell-commands.md#13-environment--powershell-basics) |
| 14 | [Scheduled Tasks](powershell-commands.md#14-scheduled-tasks) |
| 15 | [BitLocker Management](powershell-commands.md#15-bitlocker-management) |
| 16 | [Archives & File Hashing](powershell-commands.md#16-archives--file-hashing) |
| 17 | [Restart, Shutdown & Power Reports](powershell-commands.md#17-restart-shutdown--power-reports) |
| 18 | [PowerShell Remoting](powershell-commands.md#18-powershell-remoting) |

## Contributing

Found an error or want to add a command? Corrections and additions are welcome — open an issue or submit a pull request. When suggesting changes, please cite the source (official Microsoft Learn documentation preferred) so commands can be verified.

## License

This guide is licensed under the [MIT License](LICENSE).

## Disclaimer

> The commands and scripts provided in this repository can make significant changes to your operating system, hardware configurations, and network settings. Always ensure you fully understand what a command does before executing it in a production environment.
>
> All information and commands are provided "as is" without warranty of any kind. You are solely responsible for any data loss, system instability, or damage that may occur from using these commands. Use them at your own risk.
