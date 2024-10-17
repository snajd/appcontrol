---
title: Lab 3 - LOLDrivers
parent: Module 2
layout: home
nav_order: 4
nav_enabled: true
---


If you have the [Attack Surface Reduction rule](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference) "Block abuse of exploited vulnerable signed drivers" configured, and Windows Defender Antivirus real-time protection configured, certain vulnerable drivers will be blocked from being written to the file system. That ASR-rule doesn't protect from drivers that already exists in the file system.

> If you are curious on how ASR-rules works at a technical level you can dissect them by following the steps [here](https://adamsvoboda.net/extracting-asr-rules/)



Loldrivers.io is a great site for finding vulnerable drivers! They list the hashes and provides rules for YARA, sysmon and even WDAC (App Control). They also provide copies of the actual driver files.


## Find a blocked driver

If we take look in the `MSRecommendedBlocklist.xml` we deployed in Lab 2, we can test if our policy actually is working.

Lets take a random hash and search for it on loldrivers.io:

![Hash](/img/mod2-lab3-img1.jpg)

Bingo! 

If we click on AsrDrv we get more information. And if we scroll down we can see file hashes, certificate signer info and other things:
![AsrDrv](/img/mod2-lab2-img2.jpg)

Let's download the vulnerable driver and see what happens when we load it.

Press Download and save the file to C:\Drivers



PS C:\drivers> mv .\ab859723016484790c87b2218931d55f.bin asrock.sys
PS C:\drivers> sc.exe create asrock.sys binpath=c:\drivers\asrock.sys type=kernel
[SC] CreateService SUCCESS
PS C:\drivers> sc start asrock.sys
PS C:\drivers> sc query asrock.sys
PS C:\drivers>
PS C:\drivers> get-service asrock.sys

Status   Name               DisplayName
------   ----               -----------
Stopped  asrock.sys         asrock.sys

PS C:\drivers> start-service asrock.sys
start-service : Service 'asrock.sys (asrock.sys)' cannot be started due to the following error: Cannot start service
asrock.sys on computer '.'.
At line:1 char:1
+ start-service asrock.sys
+ ~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OpenError: (System.ServiceProcess.ServiceController:ServiceController) [Start-Service],
   ServiceCommandException
    + FullyQualifiedErrorId : CouldNotStartService,Microsoft.PowerShell.Commands.StartServiceCommand



A lot of errors.

Lets see if we have something in our CodeIntegrity Logs!

It's starting to become a bit tiresome to constantly open the Event log and browse to the CodeIntegrity\Operational log. Lets use another great function i WDACTools: `Get-WDACCodeIntegrityEvent`

```
PS C:\drivers> Get-WDACCodeIntegrityEvent -Kernel -SinceLastPolicyRefresh


TimeCreated            : 10/17/2024 12:18:28 PM
ProcessID              : 4
User                   : DESKTOP-MR6QBF1\SYSTEM
EventType              : Enforce
SigningScenario        : Driver
UnresolvedFilePath     : \Device\HarddiskVolume3\drivers\asrock.sys
FilePath               : C:\drivers\asrock.sys
SHA1FileHash           : F42453C6A062BDC95D84E3C7CF1521A94AE615E5
SHA1AuthenticodeHash   : 6C1BB3A72EBFB5359B9E22CA44D0A1FF825A68F2
SHA256FileHash         : 4BF974F5D3489638A48EE508B4A8CFA0F0262909778CCDD2E871172B71654D89
SHA256AuthenticodeHash : 904D8D0DB7B3ED747ECFBB04386DFBE23B71FFD054F32AB17F65BC17D500F730
UnresolvedProcessName  : System
ProcessName            :
RequestedSigningLevel  : Authenticode-Signed
ValidatedSigningLevel  : Unsigned
PolicyName             : Microsoft Windows Driver Policy
PolicyID               : 10.0.27610.0
PolicyGUID             : D2BDA982-CCF6-4344-AC5B-0B44427B6816
PolicyHash             : 8EEA11A84577969779CF3EFBF89756D8F9A0715533489EF3A140C9634CADF7D5
OriginalFileName       : AsrDrv.sys
InternalName           : AsrDrv.sys
FileDescription        : ASRock IO Driver
ProductName            : ASRock IO Driver
FileVersion            : 1.0.0.0
PackageFamilyName      :
UserWriteable          : False
FailedWHQL             :
SignerInfo             :

TimeCreated            : 10/17/2024 12:18:28 PM
ProcessID              : 4
User                   : DESKTOP-MR6QBF1\SYSTEM
EventType              : Audit
SigningScenario        : Driver
UnresolvedFilePath     : \Device\HarddiskVolume3\drivers\asrock.sys
FilePath               : C:\drivers\asrock.sys
SHA1FileHash           : F42453C6A062BDC95D84E3C7CF1521A94AE615E5
SHA1AuthenticodeHash   : 6C1BB3A72EBFB5359B9E22CA44D0A1FF825A68F2
SHA256FileHash         : 4BF974F5D3489638A48EE508B4A8CFA0F0262909778CCDD2E871172B71654D89
SHA256AuthenticodeHash : 904D8D0DB7B3ED747ECFBB04386DFBE23B71FFD054F32AB17F65BC17D500F730
UnresolvedProcessName  : System
ProcessName            :
RequestedSigningLevel  : Authenticode-Signed
ValidatedSigningLevel  : Unsigned
PolicyName             : Corp: Microsoft Driver Block List
PolicyID               : 20241016
PolicyGUID             : 97ADA625-70F7-49C9-A9C1-14F021E53DE0
PolicyHash             : 24F2BD10295741D64F06BEFBECE958A295E2519A66DA2DBE85F04E6A892B5AB7
OriginalFileName       : AsrDrv.sys
InternalName           : AsrDrv.sys
FileDescription        : ASRock IO Driver
ProductName            : ASRock IO Driver
FileVersion            : 1.0.0.0
PackageFamilyName      :
UserWriteable          : False
FailedWHQL             :
SignerInfo             :

```

As we can see, we got two events. One Audit event from our `Corp: Microsoft Driver Block List` policy, and one Enforce event from `Microsoft Windows Driver Policy`. The error message we got when trying to load the driver was really App Control blocking the kernel binary from loading!


**When completed. Remove the deployed policy from `C:\windows\system32\CodeIntegrity\CiPolicies\Active`.
Reboot the virtual machine, log on and use `citool.exe -lp` (or the Event Log) to verify that no custom policies are still applied.**
