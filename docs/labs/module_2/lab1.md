---
title: Lab 1 - Adding Recommended Driver Blocklist manually
parent: Module 2
layout: home
nav_order: 1
nav_enabled: true
---

# Lab 1 - Adding Recommended Driver Blocklist manually


## Downloading the policy

Microsoft [hosts a list of vulnerable drivers](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules). This block list also gets deployed via Windows Update, but not at the frequency you might want:

>The blocklist is updated with each new major release of Windows, typically 1-2 times per year, including most recently with the Windows 11 2022 update >released in September 2022. The most current blocklist is now also available for Windows 10 20H2 and Windows 11 21H2 users as an optional update from >Windows Update. Microsoft will occasionally publish future updates through regular Windows servicing.

Visit https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules and scroll down to where the policy start. Click the Copy button on the upper right side of the code window that displays the policy xml.
 
## Examining the policy

Open a new Text editor window and paste the policy xml. Save this to `C:\Policys\MSRecommendedDriverBlockList.xml`

OK, now we have a policy downloaded from the internet, now what?

Let's start by looking at the policy format. Is the policy in Single Policy Format or Multiple Policy Format? If we would like to use this on Windows Server 2016 or Windows 10 1903 or older, we would have to merge it with our existing company AppControl policy, because those older operating systems could only use the old format. The same thing if we would want to deploy it via Group Policy.

By looking at the first few lines, we can compare it to one of our own Multple Policy format policys that we created in Module 1


```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy">
  <VersionEx>10.0.27685.0</VersionEx>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy" PolicyType="Base Policy">
  <VersionEx>10.0.3.0</VersionEx>
  <PolicyID>{C22C1FB1-B088-4000-BAC2-10AEF4893794}</PolicyID>
  <BasePolicyID>{C22C1FB1-B088-4000-BAC2-10AEF4893794}</BasePolicyID>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
```

The above snippet that we copied from Microsoft is clearly in Single Policy Format. We will need to convert it before we use it.

If we scroll down a bit further in `C:\Policys\MSRecommendedDriverBlockList.xml` we come to another intresting part:

```xml
 <Allow ID="ID_ALLOW_ALL_2" FriendlyName="" FileName="*" />
 <Allow ID="ID_ALLOW_ALL_1" FriendlyName="" FileName="*" />
```

The policy begins with two allow rules, one for KMCI and one for UMCI. This is because every other rule in this policy file is a Deny rule. But you can't have a policy with only deny rules becase then nothing would be allowed. This policy is configured to allow everything - except the vulnerable drivers. Makes sense.

The two allow rules also allows us to do something else: we can use multiple Base polices! One just for the Block list and another one for our own environment. As you may remember from yesterday: if you have multiple Base policies, everything that runs needs to be allowed in _all_ base policies.

## Modifying and deploying the policy

Lets start by converting the policy to Multiple Policy format and give it a name and a version with the date we downloaded the policy on.

```powershell
$policyid = Set-CIPolicyIdInfo -FilePath C:\Policies\MSRecommendedDriverBlocklist.xml -PolicyName "Custom: Microsoft Driver Block List" -PolicyId "20241016" -ResetPolicyID
```
The ResetPolicyID parameter converts a policy to Multiple Policy format for us, and as we know from previous labs, the result that is written to stdout from Set-CIPolicyIdInfo, is the new PolicyID, that we need when we rename the policy, we can store that in a variable.

## Deploy the Base policy

```
ConvertFrom-CIPol

ConvertFrom-CIPolicy -XmlFilePath C:\Policies\MSRecommendedDriverBlocklist.xml -BinaryFilePath C:\Windows\System32\CodeIntegrity\CiPolicies\Active\$policyid.cip <-- funkar skitdåligt. Skriver ju ut mer än bara policyid>