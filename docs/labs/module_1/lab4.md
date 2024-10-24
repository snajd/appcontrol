---
title: Lab 4 - Deploying a policy in Audit mode
parent: Module 1
layout: home
nav_order: 4
nav_enabled: true
---

# Lab 4 - Deploying a policy in Audit mode

In this lab we will see how you can deploy a policy in audit mode, how you give it a name and version and what it looks like in the Event Log when the policy is applied.



## Creating and deploying the policy

Let's begin by copying one of the example policies to use as our starting point. We are also going to create a folder to store our policies in: `C:\Policies`

```powershell
New-Item -Path c:\Policies -Type Directory -force
Copy-Item C:\Windows\schemas\CodeIntegrity\ExamplePolicies\DefaultWindows_Audit.xml c:\Policies\Mod1Lab4-Win11-Audit.xml -force
```

Set a name and a version string on the new policy:

```powershell
Set-CIPolicyIdInfo -FilePath c:\Policies\Mod1Lab4-Win11-Audit.xml -PolicyName "Module 1 Lab 4 Windows 11 Audit Policy" -PolicyId "20241010" -ResetPolicyID
```

Open the `C:\Policies\Mod1Lab4-Win11-Audit.xml` file in a text editor and scoll through it. Look at what RuleOptions are configured and that the name and version we just configured are in the Settings element at the bottom. Close the file.



Get the policy ready for deployment by converting it from XML to binary format:

```powershell
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Mod1Lab4-Win11-Audit.xml -BinaryFilePath C:\Policies\Mod1Lab4-Win11-Audit.bin
```

Because we are using the multiple policy format, we need to name the policy `"<PolicyIDGUID>.cip"`. You can get the PolicyID from the XML by using the command below:


```powershell
$policyid = ([xml](get-content C:\Policies\Mod1Lab4-Win11-Audit.xml)).SiPolicy.PolicyID
 
```
Check that $policyid now contains a guid. If the variable is empty - run the above command again.


Rename the policy to the correct naming convention:

```powershell
Move-Item C:\Policies\Mod1Lab4-Win11-Audit.bin -Destination "C:\Policies\$policyid.cip"
```


Deploy the policy by copying it to the correct location for multiple policy format policies:

**NOTE: This requires elevation**

```powershell
Copy-Item "C:\Policies\$policyid.cip" -Destination C:\windows\system32\CodeIntegrity\CiPolicies\Active
```

Restart the virtual machine to apply the policy.

When the system comes up again, log on and open the Event log. Browse to `Applications and Services logs > Microsoft > Windows > CodeIntegrity` and click on the Operational log. Look for (or filter) for Event ID 3099. This the event that gets registered every time Windows loads a code integrity policy. Verify that you have something like this in your log, but named "Module 1 Lab 4 Windows 11 Audit Policy": 
![EventID 3988](/img/lab4-img1.jpg "EventID 3099")

## Examining the policy

Now that we have an audit policy applied that just allows stuff that comes bundled with Windows, let's try it out.

Download and install [NotePad++](https://notepad-plus-plus.org/downloads/)

Start Notepad++ and then open up the `Microsoft-Windows-CodeIntegrity/Operational` again. Click Refresh in the Actions pane to the right to reload new events.

You should be getting a few events with the `Event ID 3076`. 

From the [documentation](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/operations/event-id-explanations)
>This event is the main App Control block event for audit mode policies. It indicates that the file would have been blocked if the policy was enforced.

Doubleclick a 3076 event and click on the Details tab of the event. Scroll down and see that the logged event contains a lot of information about the file, such as file hashes and certificate info (if present). The logged event also contain information abount what policy coused the logged event.

**When completed:**

**Remove the deployed policy from `C:\windows\system32\CodeIntegrity\CiPolicies\Active`.**

**Reboot the virtual machine, log on and use `citool.exe -lp` (or the Event Log) to verify that no custom policies are still applied.**