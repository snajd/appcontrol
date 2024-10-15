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
Get-ChildItem C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe | Get-AppLockerFileInformation | New-AppLockerPolicy -RuleType Publisher -User Everyone -Xml > C:\Policies\AppLocker_MI_PS_ISE.xml

Open C:\Policies\AppLocker_MI_PS_ISE.xml in a text editor and change:

```xml
<RuleCollection Type="Exe" EnforcementMode="NotConfigured">
```

To

```xml
<RuleCollection Type="ManagedInstaller" EnforcementMode="AuditOnly">
```

Copy this example policy and paste in a text editor.

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

import gpedit

start-service appidsvc



## Prepare the App Control policy

Set-RuleOption -Option 13 .\Win11-Audit.xml

## Test an installation

## Examine the NTFS Extended attributes