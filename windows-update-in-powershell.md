# Universal Windows Update Procedure via PowerShell

Follow this exact order for **any** Windows Update (small monthly patches or massive Insider Preview upgrades) to view live progress and avoid premature reboots.

## Step 1: Start the Installation (Choose A or B)

Open **PowerShell as Administrator** and choose the correct command based on your update type:

### Option A: For Standard Updates (Monthly Patches / Drivers)
This command includes `-Verbose` so you can watch the live installation logs directly in your current window. Do **not** use the auto-reboot switch.

```powershell
Install-WindowsUpdate -AcceptAll -Verbose
```

### Option B: For Major OS Upgrades (Windows 11 Insider Previews)
Insider upgrades use specialized deployment channels that standard PowerShell modules can miss. Trigger the native Windows engine directly:

```powershell
usoclient StartInteractiveScan
```
*(Optional: If using Option B, open a second window and run `Get-Content "C:\Windows\Logs\CBS\CBS.log" -Wait -Tail 20` to watch the live background installation logs).*

---

## Step 2: Verify the Reboot Status
Before restarting, verify that the system has completely finished writing the update files to your drive. 

```powershell
Get-WUIsRebootRequired
```

* **If `$false`:** The background installation engine is still working. **Do not reboot.**
* **If `$true`:** The installation is 100% complete and safely staged. Proceed to Step 3.

---

## Step 3: Trigger the Final Reboot
Once the previous step confirms `$true`, safely force the system restart:

```powershell
Restart-Computer -Force
```
