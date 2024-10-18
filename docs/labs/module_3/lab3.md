---
title: Lab 3 - Whitelist an app using WDAC Policy Wizard
parent: Module 3
layout: home
nav_order: 3
nav_enabled: true
---

# Lab 3 - Whitelist an app using WDAC Policy Wizard

Let's whitelist another app, this time using WDAC Policy Wizard

## Whitelst an application using WDAC Policy Wizard

This way is not as controlled, but will work as long as we have an application that we know what folder it installs in.

Download putty msi installer from [https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.81-installer.msi](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.81-installer.msi).

Run the msi and take a mental note of the install directory `C:\Program Files\PuTTY\`

Open WDAC Policy Wizard on the start menu and chose to Create a new or supplemental policy.
Chose Supplemental policy

Name the policy "Putty" and set the location to `C:\Policies\Apps\putty.xml`
Browse to the Base Policy path and chose `C:\Policies\Mod3Lab2-Win11-Base.xml`

![WDACWizard](/img/mod3-lab2-img1.jpg)

Notice that we get a green checkmark that verifies that we have RuleOption 17 already set in the Base policy and that it allows supplemental policies.

Press Next.

Notice how many of the options are not set. **This is because almost all RuleOptions will be configured at the Base policy level**. We can hover our mouse over the different options so see which once are possible to set in a supplemental policy. For instance, a supplemental policy can't itself be configured in audit mode, it will inherit the Audit/Enforce mode of the Base policy.

Press Next.

Click on +Add Custom Rule

Set the rule scope to include Usermode rules. Click Yes if prompted that the policy doesnt include UMCI rules.
In the Rule Type dropdown, chose "Publisher"

On the Reference File, Click Browse, chose `C:\Program Files\PuTTY\putty.exe`

In this example we are going to trust the Publisher that has signed the application, but just for Putty, not everything they sign. Instead of creating a rule for each file, uncheck File name and check Product Name instead. Keep the Version checked.


![WDACWizard](/img/mod3-lab2-img3.jpg)



Click Create Rule and then Next to create the policy.

The resulting rule will be like this:

All files with the embedded Product name of "PuTTY suite" with the version "0.81.0.0" or above will be tagged with ID ID_FILEATTRIB_A_0_0_1
```xml
<FileAttrib ID="ID_FILEATTRIB_A_0_0_1" FriendlyName="Allow files based on file attributes: 0.81.0.0 and PuTTY suite" FileName="*" ProductName="PuTTY suite" MinimumFileVersion="0.81.0.0" />
```

The ID of the File Attributes will be linked to the specified signer:

```xml
<Signers>
    <Signer Name="Sectigo Public Code Signing Root R46" ID="ID_SIGNER_S_1_0_1">
      <CertRoot Type="TBS" Value="A229D2722BC6091D73B1D979B81088C977CB028A6F7CBF264BB81D5CC8F099F87D7C296E48BF09D7EBE275F5498661A4" />
      <CertPublisher Value="Simon Tatham" />
      <FileAttribRef RuleID="ID_FILEATTRIB_A_0_0_1" />
    </Signer>
```

That signer is listed as an allowed Signer in our policy:

```xml
  <CiSigners>
    <CiSigner SignerId="ID_SIGNER_S_1_0_1" />
  </CiSigners>
```

and finally, because we only checked "User Mode" and not "Kernel Mode" for this rule, the signed is added to the ID_SIGNINGSCENARIO_WINDOWS (that has the value 12 for some reason). If we would have left Kernel Mode checked, we would gotten an AllowedSigned in the ID_SIGNINGSCENARIO_DRIVER_1 element as well.

```xml
<SigningScenarios>
    <SigningScenario ID="ID_SIGNINGSCENARIO_DRIVERS_1" FriendlyName="Auto generated policy on 10-18-2024" Value="131">
      <ProductSigners />
    </SigningScenario>
    <SigningScenario ID="ID_SIGNINGSCENARIO_WINDOWS" FriendlyName="Auto generated policy on 10-18-2024" Value="12">
      <ProductSigners>
        <AllowedSigners>
          <AllowedSigner SignerId="ID_SIGNER_S_1_0_1" />
        </AllowedSigners>
      </ProductSigners>
    </SigningScenario>
  </SigningScenarios>
```


That it. We now have two supplemental policys created, one for FireFox and one for Putty. Let's move on and try a few other ways of creating policies for applications.













