---
title: Lab 2 - Whitelist an application using PackageInspector and New-CIPolicy
parent: Module 3
layout: home
nav_order: 2
nav_enabled: true
---

# Lab 2 - Whitelist an application using PackageInspector and New-CIPolicy


## Using PackageInspector and PowerShell

It's been a long ride but now we are finally going to whitelist a completely normal application.

The principle we are going to have, when creating our policies, is that every application will get it's own supplemental policy. This will be the easiest to manage.


1. Download Firefox Installer.exe from [https://www.mozilla.org/en-US/firefox/windows/](https://www.mozilla.org/en-US/firefox/windows/) to c:\Install\ (create the folder if it does not already exist)
2. Open a elevated Terminal and run: `PackageInspector.exe start C: -path "C:\Install\Firefox Installer.exe"`
3. Start the installer by running `& "C:\Install\Firefox Installer.exe"`
4. Close Firefox when the installation is done.
5. Stop PackageInstaller: `PackageInspector.exe stop C: -out list -listpath C:\install\firefox-files.txt`
6. Open `C:\install\firefox-files.txt` to see what got installed. A few javascript files where placed in our profile directory. These are not critical for whitelistning Firefox so lets remove those lines:

```
C:\Users\RobinEngström\AppData\Roaming\Mozilla\Firefox\Profiles\l0fq7jw3.default-release\prefs.js
C:\Users\RobinEngström\AppData\Roaming\Mozilla\Firefox\Background Tasks Profiles\fv6bdezp.MozillaBackgroundTask-308046B0AF4A39CB-defaultagent\prefs.js
```
Save and close `C:\install\firefox-files.txt`


Now we want to know if all those files are digitally signed
Open powershell again and run:
```powershell
Get-Content C:\Install\firefox-files.txt | Get-AuthentiCodeSignature
```
All files except Uninstall.exe and a js file looks signed. Thats great for us. Firefox also seems to install itself in two folders:

C:\Program Files\Mozilla Firefox\
and
C:\Program Files (x86)\Mozilla Maintenance Service\

Lets get the signing information for all the files

```powershell
$files = Get-SystemDriver -ScanPath "C:\Program Files\Mozilla Firefox" -UserPEs -NoShadowCopy
$files += Get-SystemDriver -ScanPath "C:\Program Files (x86)\Mozilla Maintenance Service\" -UserPEs -NoShadowCopy
```

Now we are going to generate a new App Control policy from the information gathered by Get-SystemDriver. We are going to use the Level FilePublisher som we trust the combination of the signer and the file information for all rules. If we would use Publisher, we would whitelist everything signed by Mozilla. Because we saw files that was unsigned, we are going to specify the fallback option of Hash, to create hash rules for Uninstall.exe.

```powershell
# Lets start by creating a folder structure for our applications
New-Item -type Directory c:\Policies\Apps

New-CIPolicy -DriverFiles $files -FilePath C:\Policies\Apps\Firefox.xml -Level FilePublisher -Fallback Hash
```

So now we have created a new policy file for just Mozilla Firefox.
We now want to use this a *Supplemental policy*, but in order to do that, we need to have a *Base policy*.
 
Let's use one of the Policies we created before as our Base policy. In Module 1, Lab 4 we created `Mod1Lab4-Win11-Audit.xml` that we tested in Enforce mode and we know does the job.

Copy `C:\Policies\Mod1Lab4-Win11-Audit.xml` to `C:\Policies\Mod3Lab2-Win11-Base.xml`, reset the policy id, give it a new name and version:

```powershell
Copy-Item C:\Policies\Mod1Lab4-Win11-Audit.xml C:\Policies\Mod3Lab2-Win11-Base.xml
Set-CIPolicyIdInfo -FilePath C:\Policies\Mod3Lab2-Win11-Base.xml -PolicyName "Win11 Base Policy (UMCI/KMCI)" -PolicyId "20241017" -ResetPolicyID
```

In order for a Base Policy to *allow* the use of supplemental policies, Rule Option 17 needs to be enabled. Open `C:\Policies\Mod3Lab2-Win11-Base.xml`
and make sure that this value exists:

```xml
    <Rule>
      <Option>Enabled:Allow Supplemental Policies</Option>
    </Rule>
```

If not, we would need to run `Set-RuleOption -Option 17 C:\Policies\Mod3Lab2-Win11-Base.xml` to enable supplemental policies.


Then we want to configure the policy for our new whitelisted application, Mozilla Firefox, to be a Supplemental policy to the new Windows 11 Base policy we just created.
We can either copy the guid value from the <BasePolicyID> in the Base policy xml file and use it as the value for the `-SupplementsBasePolicyID` parameter on Set-CIPolicyIDInfo, or you can use -BasePolicyToSupplementPath and point to the file path of the Base policy you want to supplement. The latter is easier, so that is what we are going to use in this lab.

```powershell
Set-CIPolicyIdInfo -FilePath 'C:\Policies\Apps\Firefox.xml' -BasePolicyToSupplementPath C:\Policies\Mod3Lab2-Win11-Base.xml
```

If we open up `C:\Policies\Apps\Firefox.xml`, we can now see that the PolicyType is Supplemental Policy.

```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy" PolicyType="Supplemental Policy">
```

And if we scroll down to the bottom, we can see the BasePolicyID of the Base policy.

```xml
<BasePolicyID>{E12C8B78-5A17-4F5E-A0A2-7F1DC172D073}</BasePolicyID>
```


Thats it! Now we could convert both the base policy and the new supplemental policy for Mozilla Firefox to binary and deploy them.


Let's test the policy by using the excellent WDACConfig module instead of deploying it.

WDACConfig needs to be run in PowerShell 7. Use Winget to install it (if not already installed):

```powershell
winget install pwsh
```

When PowerShell 7 is installed, start it by running pwsh.exe or directly on the Start Menu.

Install the WDAC Config module from PowerShell gallery:

```powershell
Install-Module -Name 'WDACConfig' -Scope 'AllUsers' -Force
```
Then run Invoke-WDACSimulation to test our policy against the Mozilla Firefox directories:

```powershell
Invoke-WDACSimulation -XmlFilePath C:\Policies\Apps\Firefox.xml -Folderpath "C:\Program Files\Mozilla Firefox"
Invoke-WDACSimulation -XmlFilePath C:\Policies\Apps\Firefox.xml -Folderpath "C:\Program Files (x86)\Mozilla Maintenance Service\"
```