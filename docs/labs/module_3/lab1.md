---
title: Lab 1 - Managed Installer
parent: Module 3
layout: home
nav_order: 1
nav_enabled: true
---

# White list stuff automatically by using Managed Installer

To allow the use of an Managed Installer we need to prepare a couple of things. First, we need to add RuleOption `13 Enabled:Managed Installer` to our App Control policy. We also need to configure AppLocker to run in a certain way, because it is the filter driver in AppLocker that tags all files installed by the Managed Installer and automatically white lists them.

Because we are running in an isolated lab environment we don't have access to what you would normally use as an Managed Installer (for example the Configuration Manager agent or the Intune Management Extensions) we need to be creative, and a bit stupid.

 *THIS IS NOT A THING YOU WOULD EVER DO IN PRODUCTION*

 We are going to pretend that `powershell_ise.exe` is our system management agent and use that as our Managed Installer! The result we want is that all installations we run via `powershell_ise.exe` will be automatically whitelisted.

## Prepare the AppLocker policy

1. Start by creating an Applocker policy for our "system management agent", PowerShell ISE:

```powershell
Get-ChildItem C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe | Get-AppLockerFileInformation | New-AppLockerPolicy -RuleType Publisher -User Everyone -Xml > C:\Policies\AppLocker_MI_PS_ISE.xml
```

2. Open C:\Policies\AppLocker_MI_PS_ISE.xml in a text editor and change:

```xml
<RuleCollection Type="Exe" EnforcementMode="NotConfigured">
```

To

```xml
<RuleCollection Type="ManagedInstaller" EnforcementMode="AuditOnly">
```

Copy this example policy and paste in a new empty text editor.

```xml
<AppLockerPolicy Version="1">
  <RuleCollection Type="Dll" EnforcementMode="AuditOnly" >
    <FilePathRule Id="86f235ad-3f7b-4121-bc95-ea8bde3a5db5" Name="Benign DENY Rule" Description="" UserOrGroupSid="S-1-1-0" Action="Deny">
      <Conditions>
        <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.dll" />
      </Conditions>
    </FilePathRule>
    <RuleCollectionExtensions>
      <ThresholdExtensions>
        <Services EnforcementMode="Enabled" />
      </ThresholdExtensions>
      <RedstoneExtensions>
        <SystemApps Allow="Enabled"/>
      </RedstoneExtensions>
    </RuleCollectionExtensions>
  </RuleCollection>
  <RuleCollection Type="Exe" EnforcementMode="AuditOnly">
    <FilePathRule Id="9420c496-046d-45ab-bd0e-455b2649e41e" Name="Benign DENY Rule" Description="" UserOrGroupSid="S-1-1-0" Action="Deny">
      <Conditions>
        <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.exe" />
      </Conditions>
    </FilePathRule>
    <RuleCollectionExtensions>
      <ThresholdExtensions>
        <Services EnforcementMode="Enabled" />
      </ThresholdExtensions>
      <RedstoneExtensions>
        <SystemApps Allow="Enabled"/>
      </RedstoneExtensions>
    </RuleCollectionExtensions>
  </RuleCollection>
  <RuleCollection Type="ManagedInstaller" EnforcementMode="AuditOnly">
    <FilePublisherRule Id="55932f09-04b8-44ec-8e2d-3fc736500c56" Name="MICROSOFT.MANAGEMENT.SERVICES.INTUNEWINDOWSAGENT.EXE version 1.39.200.2 or greater in MICROSOFT&reg; INTUNE&trade; from O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions>
          <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="*" BinaryName="MICROSOFT.MANAGEMENT.SERVICES.INTUNEWINDOWSAGENT.EXE">
            <BinaryVersionRange LowSection="1.39.200.2" HighSection="*" />
          </FilePublisherCondition>
    </Conditions>
    </FilePublisherRule>
    <FilePublisherRule Id="6ead5a35-5bac-4fe4-a0a4-be8885012f87" Name="CMM - CCMEXEC.EXE, 5.0.0.0+, Microsoft signed" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions>
        <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="*" BinaryName="CCMEXEC.EXE">
          <BinaryVersionRange LowSection="5.0.0.0" HighSection="*" />
        </FilePublisherCondition>
      </Conditions>
    </FilePublisherRule>
    <FilePublisherRule Id="8e23170d-e0b7-4711-b6d0-d208c960f30e" Name="CCM - CCMSETUP.EXE, 5.0.0.0+, Microsoft signed" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
      <Conditions>
        <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="*" BinaryName="CCMSETUP.EXE">
          <BinaryVersionRange LowSection="5.0.0.0" HighSection="*" />
          </FilePublisherCondition>
        </Conditions>
      </FilePublisherRule>
    </RuleCollection>
  </AppLockerPolicy>
  ```

  Replace the whole last RuleCollection that contains the rules for Managed Installer with the RuleCollection element in `C:\Policies\AppLocker_MI_PS_ISE.xml`

  It should look like this when you are finished:



```xml
<AppLockerPolicy Version="1">
  <RuleCollection Type="Dll" EnforcementMode="AuditOnly" >
    <FilePathRule Id="86f235ad-3f7b-4121-bc95-ea8bde3a5db5" Name="Benign DENY Rule" Description="" UserOrGroupSid="S-1-1-0" Action="Deny">
      <Conditions>
        <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.dll" />
      </Conditions>
    </FilePathRule>
    <RuleCollectionExtensions>
      <ThresholdExtensions>
        <Services EnforcementMode="Enabled" />
      </ThresholdExtensions>
      <RedstoneExtensions>
        <SystemApps Allow="Enabled"/>
      </RedstoneExtensions>
    </RuleCollectionExtensions>
  </RuleCollection>
  <RuleCollection Type="Exe" EnforcementMode="AuditOnly">
    <FilePathRule Id="9420c496-046d-45ab-bd0e-455b2649e41e" Name="Benign DENY Rule" Description="" UserOrGroupSid="S-1-1-0" Action="Deny">
      <Conditions>
        <FilePathCondition Path="%OSDRIVE%\ThisWillBeBlocked.exe" />
      </Conditions>
    </FilePathRule>
    <RuleCollectionExtensions>
      <ThresholdExtensions>
        <Services EnforcementMode="Enabled" />
      </ThresholdExtensions>
      <RedstoneExtensions>
        <SystemApps Allow="Enabled"/>
      </RedstoneExtensions>
    </RuleCollectionExtensions>
  </RuleCollection>
  <RuleCollection Type="ManagedInstaller" EnforcementMode="AuditOnly">
    <FilePublisherRule Id="76de31fc-f926-4e48-88c3-032b46f06657" Name="POWERSHELL_ISE.EXE version 10.0.22621.1 exactly in MICROSOFT® WINDOWS® OPERATING SYSTEM from O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" Description="" UserOrGroupSid="S-1-1-0" Action="Allow">
        <Conditions>
            <FilePublisherCondition PublisherName="O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US" ProductName="MICROSOFT® WINDOWS® OPERATING SYSTEM" BinaryName="POWERSHELL_ISE.EXE">
                <BinaryVersionRange LowSection="10.0.22621.1" HighSection="10.0.22621.1" />
            </FilePublisherCondition>
        </Conditions>
    </FilePublisherRule>
   </RuleCollection>
  </AppLockerPolicy>
  ```

Save the policy as `C:\Policies\AppLocker_MI-Final.xml`

New we need to deploy this AppLocker policy to make everything PowerShell_ISE.exe installs, flagged as installed by a managed installer.

Open gpedit.msc and browse to: Computer Configuration > Windows Settings > Application Control Policies 
Right click AppLocker and chose Import Policy. Use `C:\Policies\AppLocker_MI-Final.xml` to Import.


![GPEdit.msc](/img/mod3-lab1-img1.jpg "gpedit.msc") 


Now we need to start the Applocker AppId service. Open an elevated Terminal and run:

```powershell
Start-Service appidsvc
```


## Prepare the App Control policy

1. Create a new Base policy in WDAC Policy Wizard from the Default Windows Mode template.
2. Give the policy a descriptive name and save it in C:\Policies
3. Configure the policy to be an Audit policy and *make sure to enable the Managed Installer rule* option.
4. Deploy the policy and use one of the tools used previously to apply the policy or reboot.
5. Make sure that the policy is applied by running `citool.exe -lp`


## Test an installation

Download 7-zip from https://www.7-zip.org/download.html to C:\Install\

Start PowerShell ISE as Administrator and run
& "C:\Users\RobinEngström\Downloads\7z2408-x64.exe"
Click through the installation

## Examine the NTFS Extended attributes

Open a elevated terminal and run fsutil to view the Extended Attributes on the installed files from our Managed Installer:


```
fsutil file queryEA "C:\Program Files\7-Zip\7z.dll"

Extended Attributes (EA) information for file C:\Program Files\7-Zip\7z.dll:

Total Ea Size: 0x273

Ea Buffer Offset: 0
Ea Name: $KERNEL.SMARTLOCKER.ORIGINCLAIM
Ea Value Length: c4
0000:  01 00 00 00 00 00 00 00  00 00 00 00 01 00 00 00  ................
0010:  6e 1b b2 2d 0a 47 21 54  26 5f ce 9e 43 66 68 d4  n..-.G!T&_..Cfh.
0020:  d8 73 a4 ef 14 c1 4c 25  42 ed 03 16 bb 68 66 ed  .s....L%B....hf.
0030:  00 00 00 00 00 00 00 00  82 00 00 00 5c 00 3f 00  ............\.?.
0040:  3f 00 5c 00 43 00 3a 00  5c 00 57 00 69 00 6e 00  ?.\.C.:.\.W.i.n.
0050:  64 00 6f 00 77 00 73 00  5c 00 73 00 79 00 73 00  d.o.w.s.\.s.y.s.
0060:  74 00 65 00 6d 00 33 00  32 00 5c 00 57 00 69 00  t.e.m.3.2.\.W.i.
0070:  6e 00 64 00 6f 00 77 00  73 00 50 00 6f 00 77 00  n.d.o.w.s.P.o.w.
0080:  65 00 72 00 53 00 68 00  65 00 6c 00 6c 00 5c 00  e.r.S.h.e.l.l.\.
0090:  76 00 31 00 2e 00 30 00  5c 00 50 00 6f 00 77 00  v.1...0.\.P.o.w.
00a0:  65 00 72 00 53 00 68 00  65 00 6c 00 6c 00 5f 00  e.r.S.h.e.l.l._.
00b0:  49 00 53 00 45 00 2e 00  65 00 78 00 65 00 00 00  I.S.E...e.x.e...
00c0:  d0 a5 33 3c                                       ..3<

Ea Buffer Offset: ec
Ea Name: $KERNEL.PURGE.APPID.HASHINFO
Ea Value Length: 33
0000:  00 00 00 41 49 44 31 20  00 00 00 00 00 00 00 00  ...AID1 ........
0010:  00 00 00 e7 9d df b6 31  9d bf 9b ac 63 82 03 5d  .......1....c..]
0020:  23 59 7d ad 97 9d b5 e7  1a 60 5d 81 a6 1e e8 17  #Y}......`].....
0030:  c1 e8 12                                          ...

Ea Buffer Offset: 144
Ea Name: $KERNEL.SMARTLOCKER.HASH
Ea Value Length: 2b
0000:  00 00 00 53 4d 48 31 20  00 00 00 e7 9d df b6 31  ...SMH1 .......1
0010:  9d bf 9b ac 63 82 03 5d  23 59 7d ad 97 9d b5 e7  ....c..]#Y}.....
0020:  1a 60 5d 81 a6 1e e8 17  c1 e8 12                 .`]........

Ea Buffer Offset: 190
Ea Name: $KERNEL.PURGE.SMARTLOCKER.VALID
Ea Value Length: 4
0000:  53 4d 56 31                                       SMV1

Ea Buffer Offset: 1bc
Ea Name: $KERNEL.PURGE.SEC.FILEHASH
Ea Value Length: 94
0000:  03 e7 9d df b6 31 9d bf  9b ac 63 82 03 5d 23 59  .....1....c..]#Y
0010:  7d ad 97 9d b5 e7 1a 60  5d 81 a6 1e e8 17 c1 e8  }......`].......
0020:  12 db 38 ac 22 12 75 ac  d0 87 cf 87 eb ad 39 3e  ..8.".u.......9>
0030:  f7 f6 e0 46 56 11 43 c4  90 5b ba 16 d8 cc 02 c6  ...FV.C..[......
0040:  ba 8f 37 f3 65 bf d5 ea  7a 77 ad d9 5b 77 e5 d9  ..7.e...zw..[w..
0050:  9b ef ae e9 65 d9 a7 6b  d9 5e fa 75 ee 7f b7 79  ....e..k.^.u..y
0060:  5d ad 66 56 5e e7 57 ab  f7 f9 7b e6 ae f5 7e 97  ].fV^.W...{...~.
0070:  95 eb 9d 7e fb 6e fb 5f  a5 bd 5e bb 69 ba 57 5a  ...~.n._..^.i.WZ
0080:  dd b9 af aa b6 00 00 00  7f 72 10 ca 5e a8 75 6f  ........r..^.uo
0090:  ee ff e3 38                                       ...8
PS C:\Program Files\7-Zip>

```


"$KERNEL.SMARTLOCKER.ORIGINCLAIM" is the tag placed on the files by AppLocker that will be whitelisted by App Control.


**When completed:**

**Remove the deployed policy from `C:\windows\system32\CodeIntegrity\CiPolicies\Active`.**

**Reboot the virtual machine, log on and use `citool.exe -lp` (or the Event Log) to verify that no custom policies are still applied.**