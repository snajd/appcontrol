---
title: Lab 2 - Exploring the built in driver block policy
parent: Module 2
layout: home
nav_order: 3
nav_enabled: true
---

# Lab 2 - Exploring the built in driver block policy

## Windows Driver Policy

In the previous lab, we deployed the recommended driver block list from Microsoft. But, wasn't that already included in Windows? Especially when running HVCI?

The short answer is Yes!

As you may have noticed, when we are looking at the output from cipolicy.exe -lp and checking the CodeIntegrity logs for 3099 eve ts, this policy will show up:

> Refreshed and activated Code Integrity policy {d2bda982-ccf6-4344-ac5b-0b44427b6816} **Microsoft Windows Driver Policy**. id 10.0.27610.0. Status 0x0

This is the built-in driver block policy that gets deployed with Windows, and is updated when Microsoft feels like it. The obvious question is of course: Is this the same recommended block list as the one Microsoft publishes on their web site? How can we know?


## Converting a binary policy to xml

As we know by now, there is no way to convert a binary policy back to xml, once we have created it with ConvertFrom-CiPolicy (or WDAC Policy Wizard). 
Luckily for us, the WDAC genious [Matt Graeber](https://mattifestation.medium.com/) have devoped just that functionality in his PowerShell Module [WDACTools](https://github.com/mattifestation/WDACTools)

### Downloading and loading WDAC Tools

Go to [GitHub](https://github.com/mattifestation/WDACTools) and download Matt's module
Click on the green "Code" button and chose Download ZIP.
Extract the Downloaded zip to C:\Tools\

Remove the Mark of The Web from the downloaded PowerShell module files by running
```powershell
Get-ChildItem -Recurse C:\tools\WDACTools-master\ | Unblock-File
```
This will bypass the default ExecutionPolicy in PowerShell.
Load the PowerShell module:

```powershell
import-module C:\tools\WDACTools-master\WDACTools.psd1
```

Run `Get-Command -Module WDACTools` to take a look at all the built in functions in the module

```
PS C:\tools> get-command -Module WDACTools

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Function        ConvertTo-WDACCodeIntegrityPolicy                  2.0.1.0    WDACTools
Function        Copy-WDACEventFile                                 2.0.1.0    WDACTools
Function        Disable-WDACDeployedPolicy                         2.0.1.0    WDACTools
Function        Get-WDACApplockerScriptMsiEvent                    2.0.1.0    WDACTools
Function        Get-WDACCodeIntegrityBinaryPolicyCertificate       2.0.1.0    WDACTools
Function        Get-WDACCodeIntegrityEvent                         2.0.1.0    WDACTools
Function        Invoke-WDACCodeIntegrityPolicyBuild                2.0.1.0    WDACTools
Function        New-WDACPolicyConfiguration                        2.0.1.0    WDACTools
Filter          Update-WDACBinaryCodeIntegrityPolicy               2.0.1.0    WDACTools
```

A lot of interesting stuff!

### Finding Windows Driver Policy

The built in Windows Driver Policy is located in C:\Windows\System32\CodeIntegrity and in <EFI System Partition>\Microsoft\Boot\ and is named `driversipolicy.p7b`. 

Copy the driverssipolicy.p7b file to C:\Policies from one of the above locations.

Now we want to extract it and have a look.

```powershell
copy C:\windows\system32\CodeIntegrity\driversipolicy.p7b C:\Policies\
ConvertTo-WDACCodeIntegrityPolicy -BinaryFilePath C:\Policies\driversipolicy.p7b -XmlFilePath C:\Policies\driverssipolicy.xml
```

Now to the interesting part: how does these two files compare?

As we can see, the policy we created ourselves are about twice as big as the one that comes bundled with windows. 

```
PS C:\tools> Get-ChildItem C:\Policies\driverssipolicy.xml,C:\Policies\MSRecommendedDriverBlocklist.xml


    Directory: C:\Policies


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/17/2024  10:17 AM         299102 driverssipolicy.xml
-a----        10/16/2024   4:22 PM         502045 MSRecommendedDriverBlocklist.xml
```

Open the two files and scroll through them. Just a bunch of Hashes that does'nt really tell us that much.

