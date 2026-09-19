# Universal Windows Update Procedure via PowerShell

Follow this exact order for **any** Windows Update (small monthly patches or massive Insider Preview upgrades) to view live progress and avoid premature reboots.

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

> **Caveat:** `usoclient` is built into Windows but Microsoft does not document it officially — it is unsupported and build-dependent: it triggers a *scan* only (it does not install updates by itself) and reportedly does nothing on some newer Windows 11 builds. Test it on your build before relying on it.

For supported troubleshooting, generate a readable Windows Update log from the event traces (see [Windows Update log files](https://learn.microsoft.com/en-us/windows/deployment/update/windows-update-logs)):

```powershell
# Generate a readable WindowsUpdate.log on the Desktop (from ETW traces)
Get-WindowsUpdateLog
```

---

## Step 2: Verify the Reboot Status
Before restarting, check whether the Windows Update engine requires a reboot to finish staging the installed updates:

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
