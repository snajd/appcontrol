---
title: Lab 5 - Whitelist using logged events
parent: Module 3
layout: home
nav_order: 4
nav_enabled: true
---

# Lab 4 - Whitelist using logged events


## Preparing

In this lab we are going to create or policy based on logged events in the Event Log. As we will see we could have used an exported evtx from another machine, or a csv-file from a Defender for Enpoint export instead.

The process will be very simple.
We will start by deploying an App Control policy in audit mode.
We will then install and run our application
After that we will fetch the logs from WDAC Wizard and create a new supplemental policy



Run `citool -lp` to verify that we don't have any of our previous policies deployed at the moment. If you do, delete them and reboot.

Copy C:\Windows\Schemas\ExamplePolicies\DefaultWindows_Audit.xml to C:\Policies\Mod3Lab5-Audit.xml
Set a name and version, reset the PolicyID and deploy the policy:

```powershell
cp C:\windows\schemas\CodeIntegrity\ExamplePolicies\DefaultWindows_Audit.xml C:\Policies\Mod3Lab5-Audit.xml
Set-CIPolicyIdInfo -FilePath C:\Policies\Mod3Lab5-Audit.xml -PolicyName "Module 3 Lab 5 Install App Audit" -PolicyId "20241018" -ResetPolicyID
$policyid = ([xml]$id = get-content C:\Policies\Mod3Lab5-Audit.xml).SiPolicy.PolicyID
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Mod3Lab5-Audit.xml -BinaryFilePath C:\windows\system32\CodeIntegrity\CiPolicies\Active\$policyid.cip
```

Run C:\Tools\RefreshPolicy(AMD64).exe to refresh the policy.

Run citools -lp to verify that the policy was applied

```
Policy:
    Policy ID: 89d4f1ca-2980-42fb-b86b-9e604ccef883
    Base Policy ID: 89d4f1ca-2980-42fb-b86b-9e604ccef883
    Friendly Name: Module 3 Lab 4 Install App Audit
    Version: 2814749767303168
    Platform Policy: false
    Has File on Disk: true
    Is Currently Enforced: true
    Is Authorized: true
    Status: 0

```

Before we start installing applications, it is always good to start with a empty CodeIntegrity event log.
Clean the event log in Event Viewer or by using `wevtutil.exe`:

```powershell
wevtutil.exe cl Microsoft-Windows-CodeIntegrity/Operational
```

With a deployed Audit policy (that will log everything not Windows) and cleared event log we are ready to start.


## Whitelistning WinRAR

Let's use the classic software WinRAR this time. Download it from [here](https://www.win-rar.com/fileadmin/winrar-versions/winrar/winrar-x64-701.exe)
Save the installer to C:\Install
Run and complete the WinRAR installer, accepting all the defaults

Start Winrar one time and close it, to get som log events from when the application starts.

Now we can use the WDACTools policy module to look at what got logged. We can also use the normal Event viewer if we prefer that.

```powershell
import-module C:\tools\WDACTools-master\WDACTools.psd1
Get-WDACCodeIntegrityEvent -SinceLastPolicyRefresh
```
Scroll through the output of the Get-WDACCodeIntegrityEvent and see that we get a lot of information about the different WinRAR files (and problably more things). We could use this to script the creation of a policy file if we want to.

But let's use WDAC Policy Wizard again.

Open WDAC Policy Wizard, but this time, chose the "Policy Editor" middle option.
Check the checkbo next to "Convert Event Log to a WDAC Policy"
Click Parse Event Logs
Confirm and press Next

As we can see we captured a lot of other things besides just Winrar. Press the Product tab to sort by product.

![Events](/img/mod3-lab5-img1.jpg)

Select each of the Winrar files and click on `+ Add Allow`
Press Next when done.

No we don't get any options in WDAC Policy Wizard on what to name the policy or where to save it. It will automatically be saved in the logged on user's Documents folder.
On my computer the saved policy was saved to: `"C:\Users\RobinEngström\Documents\EventLogPolicy_2024-10-18T12-02-54.xml"`

Move and rename the policy from the Documents folder to `C:\Policies\Apps\Winrar.xml`

Open the `C:\Policies\Apps\Winrar.xml` in a text editor. Note that the policy is already a Supplemental policy. Scroll down to the <Settings> element. 
Change the name from "SupplementalTemplate" to "Winrar" and "041922" to "20241018" to look like this:

```xml
  <Settings>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Name">
      <Value>
        <String>Winrar</String>
      </Value>
    </Setting>
    <Setting Provider="PolicyInfo" Key="Information" ValueName="Id">
      <Value>
        <String>20241018</String>
      </Value>
    </Setting>
  </Settings>
```

Save and close the file.


The obvious downside to this approach is that we only will white list the components started by the user during the time our policy was in Audit mode. If it is an complex application there could be components that only get's used once in a while that we didn't catch in our logs.


Now we are done with our policy for Winrar.