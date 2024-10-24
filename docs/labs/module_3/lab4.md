---
title: Lab 4 - When things are hopeless - Whitelist using FilePath-rules
parent: Module 3
layout: home
nav_order: 4
nav_enabled: true
---

# Lab 4 - When things are hopeless - Whitelist using FilePath-rules

In a corporate IT environment thare are somtimes old legacy, or custom software that the business still need, and have used for ever, but that is a hassle for the IT personel. Often these can be tamed to work with App Control by Catalog signing them or by creating Hash rules. But as we know there are software that behave in strange ways, maybe the software creates a lot of temporary executables when the user starts it. Temporary executables that we can't calculate the hash for beforehand or sign in any way.

In scenarios like this we can use our last resort: FilePath rules.

As we talked about earlier there is a golden rule when working with application whitelistning: *No user should be able to create and run executable files from folders that are writable by the user*. Because if they can, all whitelistning work have been a waste of time.

In AppLocker this is really hard, because we have to know all the combination of policies that are hitting our Windows machines and users. We also need to know what NTFS-permissions are being set locally on those computers. If someone for instance, packages and deploys a new software, we have to make sure that they don't mess with the NTFS permissions without also creating apropriate AppLocker policies.

But in App Control we get some more help with this scenario. App Control *will check* if the folder in our FilePath rule is writable for normal users and refuse to allow it if the permissions are wrong. 

It will do this by checking the ACL against a well known list of Security Identifiers:
S-1-3-0; S-1-5-18; S-1-5-19; S-1-5-20; S-1-5-32-544; S-1-5-32-549; S-1-5-32-550; S-1-5-32-551; S-1-5-32-577; S-1-5-32-559; S-1-5-32-568; S-1-15-2-1430448594-2639229838-973813799-439329657-1197984847-4069167804-1277922394; S-1-15-2-95739096-486727260-2033287795-3853587803-1685597119-444378811-2746676523.

These are the only Sids that are allowed to have write or full permissions to the path in the rule

From the [documentation](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/select-types-of-rules-to-create)


## Checking out the SID:s

You can argue that is not very pedagogical of Microsoft to just list some random numbers and just expect us to blindly trust them. Let's see what hides behind the SIDs.

First, install James Farshaws excellent PowerShell Module NTObjectManager:

```powershell
Install-module ntobjectmanager -Scope AllUsers -force
Import-module ntobjectmanager

# Put all the SID in a variable
$sids = "S-1-3-0; S-1-5-18; S-1-5-19; S-1-5-20; S-1-5-32-544; S-1-5-32-549; S-1-5-32-550; S-1-5-32-551; S-1-5-32-577; S-1-5-32-559; S-1-5-32-568; S-1-15-2-1430448594-2639229838-973813799-439329657-1197984847-4069167804-1277922394; S-1-15-2-95739096-486727260-2033287795-3853587803-1685597119-444378811-2746676523"

$sids -split ";" | foreach-Object {Get-NTSid $_.Trim()}
```

The output will list the human readable names hiding behind the well-defined SIDs:

```

Name                                                                              Sid
----                                                                              ---
CREATOR OWNER                                                                     S-1-3-0
NT AUTHORITY\SYSTEM                                                               S-1-5-18
NT AUTHORITY\LOCAL SERVICE                                                        S-1-5-19
NT AUTHORITY\NETWORK SERVICE                                                      S-1-5-20
BUILTIN\Administrators                                                            S-1-5-32-544
BUILTIN\Server Operators                                                          S-1-5-32-549
BUILTIN\Print Operators                                                           S-1-5-32-550
BUILTIN\Backup Operators                                                          S-1-5-32-551
BUILTIN\RDS Management Servers                                                    S-1-5-32-577
BUILTIN\Performance Log Users                                                     S-1-5-32-559
BUILTIN\IIS_IUSRS                                                                 S-1-5-32-568
windows_ie_ac_001                                                                 S-1-15-2-1430448594-2639229838-973813799-439329657-1197984…
S-1-15-2-95739096-486727260-2033287795-3853587803-1685597119-444378811-2746676523 S-1-15-2-95739096-486727260-2033287795-3853587803-16855971…
```

If we don't want App Control to do this ACL checking when applying policies, we can set the RuleOption 18 in our Base policy:

> 18 Disabled:Runtime FilePath Rule Protection

## Creating a FilePath rule

FilePath also rules support wildcards and some environment variables. The normal "*" asterix means what it usually means, but in Windows 11 and newer we can use the "?" wildcard aswell. This is probably familiar for those who are used to working with regular expressions, but the "?" wildcard means that only that one character can be any other character. 

For example C:\Temp\??.exe will mean: All files in C:\Temp that starts with exactly two characters, followed by ".exe"
and: C:\Temp\*.exe means: All files in C:\Temp that ends with ".exe"

You can of also use multiple wildcard in a single rule:
> C:\*\CCMCACHE\*\7z????-x64.exe


Create a folder for our made up legacy application:

> C:\Corporate\Legacy\Apps

Let's open up our trusty WDAC Policy Wizard and create a new supplemental policy for our made up legacy application.

Chose Create new base or supplemental policy

Chose Supplemental Policy. Name the policy "Legacy App", give it todays date as the version and save it in C:\Policies\Apps\LegacyApp.xml
Use `C:\Policies\Mod3Lab2-Win11-Base.xml` as the base policy by browsing to it.

![WDAC](/img/mod3-lab4-img1.jpg)

Press Next two times

Click + Add Custom Rule

Enable User Mode, remove Kernel mode. 
Chose Path in the dropdown.
Change from File to Folder. 
When done it should look like this

![WDAC2](/img/mod1-lab5-img2.jpg)

Press Create Rule and the Next to generate the policies

Now, if you open `C:\Policies\Apps\LegacyApp.xml` you will find our new rule:

```xml
<FileRules>
    <Allow ID="ID_ALLOW_PATH_1" FriendlyName="Allow by path: %OSDRIVE%\Corporate\Legacy\Apps\*" FilePath="%OSDRIVE%\Corporate\Legacy\Apps\*" />
  </FileRules>
```

Thats it. We will move on for now an do all the testing of our policies in a later

