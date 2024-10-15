---
title: Lab 4 - Deploying a policy in Audit mode
parent: Module 1
layout: home
nav_order: 4
nav_enabled: true
---

# Lab 4 - Deploying a policy in Audit mode


## Creating and deploying the policy

Let's begin by copying one of the example policies to use as our starting point.

```powershell
Copy-Item C:\Windows\schemas\CodeIntegrity\ExamplePolicies\DefaultWindows_Audit.xml c:\Policies\ -force
```

Rename the policy file to something more manageble:

```powershell
Move-Item c:\policies\DefaultWindows_Audit.xml -destination c:\policies\Win11-Audit.xml -Force
```

Set a name and a version string on the new policy:

```powershell
Set-CIPolicyIdInfo -FilePath c:\policies\Win11-Audit.xml -PolicyName "Corp Windows 11 Audit Policy" -PolicyId "20241010" -ResetPolicyID
```

Open the `C:\Policies\DefaultWindows_Audit.xml` file in a text editor and scoll through it. Look at what RuleOptions are configured and that the name and version we just configured are in the Settings element at the bottom. Close the file.



Get the policy ready for deployment by converting it from XML to binary format:

```powershell
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Win11-Audit.xml -BinaryFilePath C:\Policies\Win11-Audit.bin
```

Because we are using the multiple policy format, we need to name the policy `"<PolicyIDGUID>.clp"`

```powershell
$policyid = ([xml]$id = get-content C:\Policies\Win11-Audit.xml).SiPolicy.PolicyID
Move-Item C:\Policies\Win11-Audit.bin -Destination "C:\Policies\$policyid.clp"
```

Deploy the policy by copying it to the correct location for multiple policy format policies:

**NOTE: This requires elevation**

```powershell
Copy-Item "C:\Policies\$policyid.clp" -Destination C:\windows\system32\CodeIntegrity\CiPolicies\Active
```

Restart the virtual machine to apply the policy.

When the system comes up again, log on and open the Event log. Browse to `Applications and Services logs > Microsoft > Windows > CodeIntegrity` and click on the Operational log. Look for (or filter) for Event ID 3099. This the event that gets registered every time Windows loads a code integrity policy. Verify that you have something like this in your log: 
![EventID 3988](/img/lab4-img1.jpg "EventID 3099")

## Examining the policy

Now that we have an audit policy applied that just allows stuff that comes bundled with Windows, let's try it out.

Download and install [NotePad++](https://notepad-plus-plus.org/downloads/)

Start Notepad++ and then open up the `Microsoft-Windows-CodeIntegrity/Operational` again.

You should be getting a few events with the `Event ID 3076`. 

From the [documentation](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/operations/event-id-explanations)
>This event is the main App Control block event for audit mode policies. It indicates that the file would have been blocked if the policy was enforced.

Doubleclick a 3076 event and click on the Details tab of the event. Scroll down and see that the logged event contains a lot of information about the file, such as file hashes and certificate info (if present). The logged event also contain information abount what policy coused the logged event.

