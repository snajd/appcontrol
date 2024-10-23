---
title: Lab 4 - Adding Recommended block list
parent: Module 2
layout: home
nav_order: 4
nav_enabled: true
---


# Lab 4 - Adding Recommended block list

Microsoft also publishes [another recommended block list](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/applications-that-can-bypass-appcontrol), this time for binaries running in User Mode.

This is tools and applications that are built into Windows or common Windows tools that can be used to bypass App Control policies and execute code by running them via another, whitelisted, binary. One example for instance is the popular BGInfo tool, that have been shipped by Sysinternals since the NT4 days. A security researcher noticed that if you whitelist BGInfo, you could trigger vbscripts or executables inside the application and those was allowed.
This is fixed in later versions of BGInfo, but it is an interesting example. More info [here](https://oddvar.moe/2017/05/18/bypassing-application-whitelisting-with-bginfo/). 

So long story short: If we want full control of what can run on our systems, we need to block known bypasses to our defenses.

## Adding Recommended Blocklist with WDAC Policy Wizard

Instead of doing things the hard way, with PowerShell at the command line, we now are going to create our policy using the WDAC Policy Wizard. You could of course do this exercise in the exact same way as in the previous lab, if you rather use PowerShell than a graphical application.


1. Start by opening WDAC Policy Wizard
2. Chose Create a new base or supplemental policy
3. Press Next to confirm that we want to create a Multiple Policy format Base policy
4. Check "Default Windows Mode". This is the same as the DefaultWindows_Audit.xml from the ExamplePolicies.
5. Name the policy: "Module 2 Lab 4 - Recommended Block list (UMCI/KMCI)" and save it to C:\Policies\Mod2Lab4-Recommended-UMCI-KMCI.xml


![WDACWizard](/img/mod2-lab4-img1.jpg)

Press Next

Leave the defaults, including User Mode Code Integrity enabled and Audit mode and press Next

Check the two checkboxes at the bottom for Merge with Recommended User Mode Block Rules and Merge with Recommended Kernel Block Rules and press next.

And thats it. 

Open up C:\Policies\Mod2Lab4-Recommended-UMC-KMCI.xml in a text editor and search for example bginfo. Note that all deny rules are for the FriendlyName, FileName and Version metadata. Not file hashes.

If you scroll down a bit further you will see hundreds of entries like this:

```xml
  <Deny ID="ID_DENY_D_1_1" FriendlyName="Powershell 1" Hash="02BE82F63EE962BCD4B8303E60F806F6613759C6" />
    <Deny ID="ID_DENY_D_2_1" FriendlyName="Powershell 2" Hash="13765D9A16CC46B2113766822627F026A68431DF" />
    <Deny ID="ID_DENY_D_3_1" FriendlyName="Powershell 3" Hash="148972F670E18790D62D753E01ED8D22B351A57E45544D88ACE380FEDAF24A40" />
    <Deny ID="ID_DENY_D_4_1" FriendlyName="Powershell 4" Hash="29DF1D593D0D7AB365F02645E7EF4BCCA060763A" />
    <Deny ID="ID_DENY_D_5_1" FriendlyName="Powershell 5" Hash="2E3C47BBE1BA99842EE187F756CA616EFED61B94" />
    <Deny ID="ID_DENY_D_6_1" FriendlyName="Powershell 6" Hash="38DC1956313B160696A172074C6F5DA9852BF508F55AFB7FA079B98F2849AFB5" />
    <Deny ID="ID_DENY_D_7_1" FriendlyName="Powershell 7" Hash="513B625EA507ED9CE83E2FB2ED4F3D586C2AA379" />
    <Deny ID="ID_DENY_D_8_1" FriendlyName="Powershell 8" Hash="71FC552E66327EDAA72D72C362846BD80CB65EECFAE95C4D790C9A2330D95EE6" />
    <Deny ID="ID_DENY_D_9_1" FriendlyName="Powershell 9" Hash="72E4EC687CFE357F3E681A7500B6FF009717A2E9538956908D3B52B9C865C189" />
    <Deny ID="ID_DENY_D_10_1" FriendlyName="Powershell 10" Hash="74E207F539C4EAC648A5507EB158AEE9F6EA401E51808E83E73709CFA0820FDD" />
    <Deny ID="ID_DENY_D_11_1" FriendlyName="Powershell 11" Hash="75288A0CF0806A68D8DA721538E64038D755BBE74B52F4B63FEE5049AE868AC0" />
    <Deny ID="ID_DENY_D_12_1" FriendlyName="Powershell 12" Hash="7DB3AD53985C455990DD9847DE15BDB271E0C8D1" />
    <Deny ID="ID_DENY_D_13_1" FriendlyName="Powershell 13" Hash="84BB081141DA50B3839CD275FF34854F53AECB96CA9AEB8BCD24355C33C1E73E" />
    <Deny ID="ID_DENY_D_14_1" FriendlyName="Powershell 14" Hash="86DADE56A1DBAB6DDC2769839F89244693D319C6" />
    <Deny ID="ID_DENY_D_15_1" FriendlyName="Powershell 15" Hash="BD3139CE7553AC7003C96304F08EAEC2CDB2CC6A869D36D6F1E478DA02D3AA16" />
    <Deny ID="ID_DENY_D_16_1" FriendlyName="Powershell 16" Hash="BE3FFE10CDE8B62C3E8FD4D8198F272B6BD15364A33362BB07A0AFF6731DABA1" />
    <Deny ID="ID_DENY_D_17_1" FriendlyName="Powershell 17" Hash="C1196433541B87D22CE2DD19AAAF133C9C13037A" />
    <Deny ID="ID_DENY_D_18_1" FriendlyName="Powershell 18" Hash="C6C073A80A8E76DC13E724B5E66FE4035A19CCA0C1AF3FABBC18E5185D1B66CB" />
    <Deny ID="ID_DENY_D_19_1" FriendlyName="Powershell 19" Hash="CE5EA2D29F9DD3F15CF3682564B0E765ED3A8FE1" />
    <Deny ID="ID_DENY_D_20_1" FriendlyName="Powershell 20" Hash="D027E09D9D9828A87701288EFC91D240C0DEC2C3" />
    <Deny ID="ID_DENY_D_21_1" FriendlyName="Powershell 21" Hash="D2CFC8F6729E510AE5BA9BECCF37E0B49DDF5E31" />
    <Deny ID="ID_DENY_D_22_1" FriendlyName="Powershell 22" Hash="DED853481A176999723413685A79B36DD0F120F9" />
```

They are referring to all versions (and patch levels) of PowerShell v2, which itself is a known application control bypass because PowerShell v2 doesn't support Constrained Langague Mode!



Note that `bash.exe` is included in the recommended block list. Bash.exe is actually a component in Windows Subsystem for Linux, that by design is a big applicaton whitelist bypass, because it is a whole Linux distro running on Windows :).