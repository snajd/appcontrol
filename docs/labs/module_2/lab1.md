---
title: Lab 1 - Adding Recommended Driver Blocklist
parent: Module 2
layout: home
nav_order: 1
nav_enabled: true
---

# Lab 1 - Adding Recommended Driver Blocklist manually


## Downloading the policy

Microsoft [hosts a list of vulnerable drivers](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules). This block list also gets deployed via Windows Update, but not at the frequency you might want:

>The blocklist is updated with each new major release of Windows, typically 1-2 times per year, including most recently with the Windows 11 2022 update >released in September 2022. The most current blocklist is now also available for Windows 10 20H2 and Windows 11 21H2 users as an optional update from >Windows Update. Microsoft will occasionally publish future updates through regular Windows servicing.

Visit [https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/design/microsoft-recommended-driver-block-rules) and scroll down to where the Vulnerable driver blocklist XML starts. Click the Copy button on the upper right side of the code window that displays the policy xml.
 
## Examining the policy

Open a new Text editor window and paste the policy xml. Save this to `C:\Policys\Mod2Lab1-MSRecommendedDriverBlockList.xml`

OK, now we have a policy downloaded from the internet, now what?

Let's start by looking at the policy format. Is the policy in Single Policy Format or Multiple Policy Format? If we would like to use this on Windows Server 2016 or Windows 10 1903 or older, we would have to merge it with our existing company AppControl policy, because those older operating systems could only use the old format. The same thing if we would want to deploy it via Group Policy.

By looking at the first few lines, we can compare it to one of our own Multple Policy format policys that we created in Module 1


**C:\Policys\Mod2Lab1-MSRecommendedDriverBlockList.xml**:
```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy">
  <VersionEx>10.0.27685.0</VersionEx>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
```

**C:\Policies\Mod1Lab5-Win11-Audit.xml**:
```xml
<?xml version="1.0" encoding="utf-8"?>
<SiPolicy xmlns="urn:schemas-microsoft-com:sipolicy" PolicyType="Base Policy">
  <VersionEx>10.0.3.0</VersionEx>
  <PolicyID>{C22C1FB1-B088-4000-BAC2-10AEF4893794}</PolicyID>
  <BasePolicyID>{C22C1FB1-B088-4000-BAC2-10AEF4893794}</BasePolicyID>
  <PlatformID>{2E07F7E4-194C-4D20-B7C9-6F44A6C5A234}</PlatformID>
```

The top snippet above that we copied from Microsoft is clearly in Single Policy Format. We will need to convert it to Multiple Policy Format before we can use it.

If we scroll down a bit further in `C:\Policys\Mod2Lab1-MSRecommendedDriverBlockList.xml` we come to another intresting part:

```xml
 <Allow ID="ID_ALLOW_ALL_2" FriendlyName="" FileName="*" />
 <Allow ID="ID_ALLOW_ALL_1" FriendlyName="" FileName="*" />
```

After the RuleOption part (where we can see that this is a policy configured for Audit mode) The policy begins with two allow rules, one for KMCI and one for UMCI. This is because every other rule in this policy file is a Deny rule. But you can't have a policy with only deny rules becase then nothing would be allowed. This policy is configured to allow everything - except the vulnerable drivers. Makes sense.

The two allow rules also allows us to do something else: we can use multiple Base polices! One just for the Block list and another one for our own environment. As you may remember from yesterday: if you have multiple Base policies, everything that runs needs to be allowed in _all_ base policies.


## Modifying and deploying the policy

Lets start by converting the policy to Multiple Policy format and give it a name and a version with the date we downloaded the policy on.

```powershell
Set-CIPolicyIdInfo -FilePath C:\Policies\Mod2Lab1-MSRecommendedDriverBlockList.xml -PolicyName "Custom: Microsoft Driver Block List" -PolicyId "20241016" -ResetPolicyID
```
The ResetPolicyID parameter converts a policy to Multiple Policy format for us, and also generates a new guid for PolicyID

Deploy the new policy by running the command below.
```powershell
# Get the policyid from the policy file:
$policyid = ([xml](get-content C:\Policies\Mod2Lab1-MSRecommendedDriverBlockList.xml)).SiPolicy.PolicyID


# Convert the policy to binary format and make it an active policy by putting it in the right folder
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Mod2Lab1-MSRecommendedDriverBlockList.xml -BinaryFilePath C:\Windows\System32\CodeIntegrity\CiPolicies\Active\$policyid.cip

# Refresh the policy
& 'C:\tools\RefreshPolicy(AMD64).exe'

```

Run citool to verify that the policy was applied:
```
PS C:\Policies> citool -lp
Policy:
    Policy ID: 97ada625-70f7-49c9-a9c1-14f021e53de0
    Base Policy ID: 97ada625-70f7-49c9-a9c1-14f021e53de0
    Friendly Name: Custom: Microsoft Driver Block List
    Version: 2814751581470720
    Platform Policy: false
    Has File on Disk: true
    Is Currently Enforced: true
    Is Authorized: true
    Status: 0
```


Congratulations, we are now auditing all recommended vulnerable drivers, according to Microsoft.

Not very exiting, I know. But in a future lab, we will put this policy to the test!