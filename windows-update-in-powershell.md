# Universal Windows Update Procedure via PowerShell

The following is a practical workflow for common Windows Update operations — monthly patches and driver updates. The correct procedure can vary for feature upgrades, Insider builds, enterprise-managed devices, Windows Update for Business, WSUS, and other managed environments.

> **Prerequisite:** Option A and Step 2 use [PSWindowsUpdate](https://www.powershellgallery.com/packages/PSWindowsUpdate), a **third-party module** from the PowerShell Gallery — `Install-WindowsUpdate` and `Get-WURebootStatus` are provided by that module, not by the base PowerShell installation. Installing it requires internet access, package-provider setup, and administrator rights, and it may be unavailable or blocked on managed, offline, or locked-down systems. Option B (`usoclient`) and `Get-WindowsUpdateLog` are built into Windows and work without the module.

Check for and import the module before starting:

```powershell
Get-Module -ListAvailable PSWindowsUpdate
Install-Module -Name PSWindowsUpdate -Force
Import-Module PSWindowsUpdate
```

## Step 1: Start the Installation (Choose A or B)

Open **PowerShell as Administrator** and choose the correct command based on your update type:

### Option A: For Standard Updates (Monthly Patches / Drivers)
This command includes `-Verbose` so you can watch the live installation logs directly in your current window, and `-IgnoreReboot` so Windows never reboots on its own (it also skips the reboot prompt). Do **not** use `-AutoReboot`.

```powershell
Install-WindowsUpdate -AcceptAll -IgnoreReboot -Verbose
```

### Option B: For Major OS Upgrades (Windows 11 Insider Previews)
Insider upgrades use specialized deployment channels that standard PowerShell modules can miss. Trigger the native Windows engine directly:

```powershell
usoclient StartInteractiveScan
```
*(Optional: If using Option B, open a second window and run `Get-Content "C:\Windows\Logs\CBS\CBS.log" -Wait -Tail 20` to watch the live background installation logs).*

> **Caveat:** `usoclient` is built into Windows but Microsoft does not document or support it — it is build-dependent, triggers a *scan* only, and does not guarantee that updates will download or install. It reportedly does nothing on some newer Windows 11 builds. Test it on your build before relying on it; do not treat it as a reliable installation mechanism.

For supported troubleshooting, generate a readable Windows Update log from the event traces (see [Windows Update log files](https://learn.microsoft.com/en-us/windows/deployment/update/windows-update-logs)):

```powershell
# Generate a readable WindowsUpdate.log snapshot on the Desktop (from ETW traces).
# This is a static snapshot — it does not update continuously; re-run the cmdlet to refresh it.
Get-WindowsUpdateLog
```

---

## Step 2: Verify the Reboot Status
Before restarting, check whether the Windows Update engine requires a reboot to finish staging the installed updates (PSWindowsUpdate cmdlet — requires the module installed and imported):

```powershell
Get-WURebootStatus -Silent
```

* **If `$true`:** A reboot is required to complete the installation. Proceed to Step 3.
* **If `$false`:** No reboot is pending — the installation is complete without one.

---

## Step 3: Trigger the Final Reboot
Once the previous step returns `$true`, restart the computer:

```powershell
Restart-Computer
```

`-Force` (as in `Restart-Computer -Force`) closes running applications without saving — use it only when you accept that.
