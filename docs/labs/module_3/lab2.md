---
title: Lab 2 - Whitelist an application using PackageInspector and New-CIPolicy
parent: Module 3
layout: home
nav_order: 1
nav_enabled: true
---

# Lab 2 - Whitelist an application using PackageInspector and New-CIPolicy


## Using PackageInspector and PowerShell

It's been a long ride but now we are finally going to whitelist a completely normal application.

The principle we are going to have, when creating our policies, is that every application will get it's own supplemental policy. This will be the easiest to manage.


Download Firefox Installer.exe from [https://www.mozilla.org/en-US/firefox/windows/](https://www.mozilla.org/en-US/firefox/windows/) to c:\Install\ (create the folder if it does not already exist)
Open a elevated Terminal and run: `PackageInspector.exe start C: -path "C:\Install\Firefox Installer.exe"`
Start the installer by running `& "C:\Install\Firefox Installer.exe"`
Close Firefox when the installation is done.
Stop PackageInstaller: `PackageInspector.exe stop C: -out list -listpath C:\install\firefox-files.txt`
Open `C:\install\firefox-files.txt` to see what got installed. A few javascript files where placed in our profile directory. These are not critical for whitelistning Firefox so lets remove those lines:

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

Lets get the signatures for all the files

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

TODO: ÖPPNA OCH GÅ IGENOM FILEN OCH VAD SAKER BETYDER


So now we have created a new policy file for just Mozilla Firefox.
We now want to use this a *Supplemental policy*, but in order to do that, we need to have a *Base policy*.
 
Let's use one of the Policies we created before as our Base policy. In Module 1, Lab 4 we created `Mod1Lab4-Win11-Audit.xml` that we tested in Enforce mode and we know does the job.

Copy C:\Policies\Mod1Lab4-Win11-Audit.xml to C:\Policies\Mod3Lab2-Win11-Base.xml, reset the policy id, give it a new name and version:

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




ÄNDRA ALLT TILL WDACCONFIG!!!!

winget install pwsh

Install-Module -Name 'WDACConfig' -Scope 'AllUsers' -Force

Starta om terminal för att få Powershell som ett nytt val.







1. Han gör en dedikerad policy för VARJE installerad mjukvara. Gör att det blir enklare att uppdatera en specifik app. 
2. Kollar om installern är signed. (i exemplet är den inte det). Jag kör med notepadplusplus som är signerad.
3. Använder packageinspector.exe. Gör en file trace under en installation.
4. PackageInspector.exe start C: -path "C:\Users\re\Desktop\npp.7.8.6.Installer.exe"
5. starta installern. gör gärna en "full install" så man inte missar några exekverbara filer.
6. går att få den att spotta ut en .cat (eller cdf, som är lite oklar)
7. PackageInspector.exe stop C: -out list -listpath C:\users\re\desktop\npp-files.txt
8. är alla filer signerade?
9. cat .\npp-files.txt | Get-AuthenticodeSignature
10. allt verkar signerat. så kul har inte Matt. Han ska skapa hashar. Numera GÅR det att vitlista path.
11. $files = Get-SystemDriver -ScanPath "C:\Program Files (x86)\Notepad++\" -UserPEs -NoShadowCopy
12. varje objekt har UserMode, och ibland blir det fel så en usermode räknas som en driver.
13. $usermodefiles = $files | Where-Object {$_.usermode}
14. New-CIPolicy -DriverFiles $usermodefiles -FilePath notepadplusplus.xml -Level FilePublisher
15. C:\Users\re\Desktop> New-CIPolicy -DriverFiles $usermodefiles -FilePath notepadplusplus.xml -Level FilePublisher -Fallback Hash
16. återvänder efter en paus. Men han har startat om i Audit mode med UMCI:Enabled.
17. $Events = Get-WDACCodeIntegrityEvent -User -SinceLastPolicyRefresh
18. Ett vanligt event som skräpar ner är ngen.exe = performance enhancment för .NET. ngen konverterar .NET-kod till assembly. Dessa slutar på .ni.dll eller .ni.exe.
19. $Events.ResolvedFilePath | sort -unique
20. Startar vim och kör samma kommando igen
21. Develepment tools / IDE:s är så jobbigt att vitlista så rekommenderar en dedikerad VM istället.
22. Merge-CIPolicy -OutputFilepath merged.xml -policypaths .\policy2,policy2
23. Deploya