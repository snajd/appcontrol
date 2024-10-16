---
title: Lab 5 - UMCI and Enforce
parent: Module 1
layout: home
nav_order: 5
nav_enabled: true
---

# Lab 5 - Removing UMCI using the WDAC Wizard

So in the previous lab we deployed a policy for both Kernel Mode Integrity (KMCI) and User Mode Code Integrity (UMCI) in Audit mode.

Now we are going to modify our policy using a graphical user interface and make som changes.

## Download WDAC Policy Wizard 

The WDAC Policy Wizard have .NET 8.0 Desktop Runtime as requirement so [download and install](https://download.visualstudio.microsoft.com/download/pr/907765b0-2bf8-494e-93aa-5ef9553c5d68/a9308dc010617e6716c0e6abd53b05ce/windowsdesktop-runtime-8.0.8-win-x64.exe) that first.

Then download the MSIX installer for WDAC Policy Wizard [here](https://webapp-wdac-wizard.azurewebsites.net/)

## Modify an existing policy

Open the WDAC Wizard on the start menu.

Now we want to modify the policy we created in the previous lab, rename it, and remove UMCI.

Click on Policy Editor

Click on Browse

Choose C:\Policies\Mod1Lab4-Win11-Audit.xml

Name the new policy `Module 1 Lab 5 Windows 11 Audit Policy (KMCI)` and put todays date as the Version.

Chose the new file location to `C:\Policies\Mod1Lab5-Win11-Audit-KMCI.xml`

It should look something like this when you're done

![WDAC Wizard](/img/mod1-lab5-img1.jpg)

Click Next

Uncheck User Mode Code Integrity

Press Next two times to save the policy.

As we can see from the output, WDAC Wizard was nice enough to create a policy in binary format for us.

![WDAC Wizard](/img/mod1-lab5-img2.jpg)


## Testing the KMCI policy

OK, so now we have modified our policy to not care about things running in User Mode. Let's try it out.

Start by opening up the Event log and browse to Microsoft-Windows-CodeIntegrity/Operational. 
Click on Clear Log and confirm that we dont want to save the logs.

Deploy the new policy created by WDAC Wizard by copying it to `C:\windows\system32\CodeIntegrity\CiPolicies\Active`

Reboot the machine. Log on. Open the Event log and see if you can find the Event logs that says that the Module 1 Lab 5 Windows 11 Audit Policy (KMCI) was applied.

With Event log still open, start Notepad++ again. Note how nothing gets logged, because our policy is running in Kernel Mode only.
*It is important to know, that App Control only logs when things gets blocked or when things would have gotten blocked if we weren't running in Audit mode.*

## Taking things up a notch: Enforce mode for KMCI

Lets open up our policy in WDAC Wizard again. This time we want to remove Audit mode from the policy.
Edit the previous policy and change the settings to look like this:

![WDAC Wizard](/img/mod1-lab5-img3.jpg)

Press Next
Uncheck "Audit Mode"

Press Next to create a new policy.

Note that a new XML-policy file was written, but becase we are reusing the same policy, with the same PolicyID, the binary policy file is named the same as in the previous lab.

Deploy the new policy created by WDAC Wizard by copying it to `C:\windows\system32\CodeIntegrity\CiPolicies\Active`
If prompted, overwrite the existing file.
Reboot the virtual machine

Make sure that the new policy was applied by checking for Event ID 3099 in the CodeIntegrity Operational log. You should have an entry like this:

> Refreshed and activated Code Integrity policy {c22c1fb1-b088-4000-bac2-10aef4893794} Module 1 Lab 5 Windows 11 Enforce Policy (KMCI). id 20241016. Status 0x0

Start Notepad++ again and verify that we can run it although we are running a App Control Policy in Enforce mode. This is again, because our policy only cares about things running in Kernel Mode.

## Enforce mode for KMCI and UMCI

Open a PowerShell prompt
Create a new policy by copying the previous one:

```powershell
Copy-Item C:\Policies\Mod1Lab5-Win11-Enforce-KMCI.xml C:\Policies\Mod1Lab5-Win11-Enforce.xml
``` 


Activate UMCI in the policy by using Set-RuleOption

*Note, that Set-RuleOptions doesn't support Get-Help, like a normal PowerShell module. Instead they have added their own "-help" parameter.

```powershell
PS C:\Policies> set-ruleoption -help
0 Enabled:UMCI
1 Enabled:Boot Menu Protection
2 Required:WHQL
3 Enabled:Audit Mode
4 Disabled:Flight Signing
5 Enabled:Inherit Default Policy
6 Enabled:Unsigned System Integrity Policy
7 Allowed:Debug Policy Augmented
8 Required:EV Signers
9 Enabled:Advanced Boot Options Menu
10 Enabled:Boot Audit On Failure
11 Disabled:Script Enforcement
12 Required:Enforce Store Applications
13 Enabled:Managed Installer
14 Enabled:Intelligent Security Graph Authorization
15 Enabled:Invalidate EAs on Reboot
16 Enabled:Update Policy No Reboot
17 Enabled:Allow Supplemental Policies
18 Disabled:Runtime FilePath Rule Protection
19 Enabled:Dynamic Code Security
20 Enabled:Revoked Expired As Unsigned
```

```powershell
Set-RulOption -Option 0 -FilePath C:\Policies\Mod1Lab5-Win11-Enforce.xml

# Sätt ett nytt namn och version på policyn
Set-CIPolicyIdInfo -FilePath C:\Policies\Mod1Lab5-Win11-Enforce.xml -PolicyName "Module 1 Lab 5 Windows 11 Enforce Policy (UMCI/KMCI)" -PolicyId "20241016"

# Hämta PolicyID från Policyn
$policyid = ([xml]$id = get-content C:\Policies\Mod1Lab5-Win11-Enforce.xml).SiPolicy.PolicyID

# Deploya policyn:
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Mod1Lab5-Win11-Enforce.xml -BinaryFilePath C:\windows\system32\CodeIntegrity\CiPolicies\Active\$policyid.cip
```

## Updating a policy without rebooting

Instead of rebooting, like we have done a few times before, we know by now that we have the 16 Enabled:Update Policy No Reboot in our policy. This means that we don't really need to reboot.

Instead of rebooting we can refresh the policy in three different ways:

1. We can download a tool called RefreshPolicy from Microsoft
2. We can use citool.exe (if we are running Windows 11 or later)
3. We can trigger a WMI method.

Lets use option 1 for this exercise. Download the Refresh CI Policy tool from Microsoft [here](https://www.microsoft.com/en-us/download/details.aspx?id=102925). As you probably are running a 64-bit Windows Operatingsystem on an Intel compatible processor, you want to choose `RefreshPolicy(AMD64).exe`

Create a new folder called `C:\Tools\` and put `RefreshPolicy(AMD64).exe` there.

Run `RefreshPolicy(AMD64).exe` 

```powershell
PS C:\Policies> & 'C:\tools\RefreshPolicy(AMD64).exe'
Rebootless ConfigCI Policy Refreshing Succeeded!
```

Now when we have a policy in Enforce mode, that only allows software bundled with Windows and is enforcing both User and Kernel Mode we can try and start NotePad++ again:

![Block!](/img/mod1-lab5-img4.jpg)


**When completed. Remove the deployed policy from `C:\windows\system32\CodeIntegrity\CiPolicies\Active`.
Reboot the virtual machine, log on and use `citool.exe -lp` (or the Event Log) to verify that no custom policies are still applied.**
