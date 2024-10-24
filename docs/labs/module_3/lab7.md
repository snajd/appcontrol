---
title: Lab 7 - Testing policies
parent: Module 3
layout: home
nav_order: 7
nav_enabled: true
---


# Lab 7 - Testing policies

In this lab we are going to combine all the policies we created in the previous lab. We are first going to deploy them in audit mode, to verifiy that everything work, and the we are going to switch to enforce mode.




## Deploying policies in audit mode

Let's keep clean by just copying our base policy and our four App-policies to a new folder called c:\Policies\Module3\.


```
    Directory: C:\Policies\Module3


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/17/2024   8:47 PM          13756 Firefox.xml
-a----        10/18/2024  12:07 PM           1928 LegacyApp.xml
-a----        10/21/2024  10:42 AM          11731 Mod3Lab2-Win11-Base.xml
-a----        10/17/2024  10:32 PM           1640 putty.xml
-a----        10/18/2024   2:02 PM           3418 WinRAR.xml

```

Open the base policy and take a note of what the BasePolicyID is. Because we have used -ResetPolicyID a few times, this will be different for all students.

In my Mod3Lab2-Win11-Base.xml the BasePolicyID is "{E12C8B78-5A17-4F5E-A0A2-7F1DC172D073}". Also make sure that the PolicyType is "Base Policy", that the base policy is configured for Audit mode `<Option>Enabled:Audit Mode</Option>`
and that `<Option>Enabled:Allow Supplemental Policies</Option>` is present in the policy file.

For our Supplemental Policies to work, they need to be linked to this BasePolicy. Open the supplemental policies, one by one and find the BasePolicyID and compare it to the value you just noted. Also make sure that the PolicyType is "Supplemental Policy".

Also make sure that all Supplemental policies have proper names

If some of the policies (like LegacyApp.xml) doesn't match we need to link them to our base policy.

Do the following for all the policies that doesn't match the BasepolicyID:

```powershell
Set-CIPolicyIdInfo -FilePath c:\policies\module3\LegacyApp.xml -BasePolicyToSupplementPath C:\policies\module3\Mod3Lab2-Win11-Base.xml
```

Now we only have to convert all the policies to binary format, rename them and copy them to `C:\Windows\System32\CodeIntegrity\CiPolicies\Active`


```powersbell
Get-ChildItem *.xml | ForEach-Object {convertfrom-cipolicy -XmlFilePath $_ -BinaryFilePath ((([xml]$policyid = get-content $_).SiPolicy.PolicyID) + ".cip")}
copy *.cip C:\Windows\system32\CodeIntegrity\CiPolicies\Active\
& 'C:\tools\RefreshPolicy(AMD64).exe'
```

Clear the `Microsoft-Windows-CodeIntegrity/Operational` event log




## Switch to Enforce mode
Set-RuleOption -Option 3 -Delete -FilePath .\Mod3Lab2-Win11-Base.xml
restart-computer

Verify that you can start the whitelisted programs but not NotePad++ for example.

Copy the whole "C:\Program Files\Notepad++" folder to C:\Corporate\Legacy\Apps

Try and open "C:\Corporate\Legacy\Apps\Notepad++\notepad++.exe"

Still doesnt work

Check the ACL on c:\Corporate\Legacy\Apps folder. Authenticated Users have Modify permissions. Disable inheritance on the Apps folder and remove Authenticated users from the DACL.

Restart the computer again to force a reapply of the policies.

# FELSÖK



## Testing policies with WDACConfig