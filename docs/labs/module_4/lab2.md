---
title: Lab 2 - Signing a PowerShell Script and Constrained Language Mode
parent: Module 4
layout: home
nav_order: 2
nav_enabled: true
---

# Lab 2 - Signing a PowerShell Script and Constrained Language Mode

## Script Enforcement

Let's start by creating a PowerShell script for us to sign.

Copy the following lines and paste in a text editor and save the file as `C:\Install\script.ps1`

```powershell
# Testscript
Write-Host "Hej från scriptet, vi kör just nu i följande LanguageMode:" $ExecutionContext.SessionState.LanguageMode
```


## Sign a PowerShell script

Make sure that your CodeSigning certificate is installed in the Personal store of the logged on user.
Open a PowerShell prompt
List your installed code signing certificates:

Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert

Hopefully you just got one Code Signing Certificate installed.

If you want to see more information, you can pipe Get-Childitem to Filter-List:

```powershell
PS C:\Policies> Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert | fl


Subject      : CN=CodeSign
Issuer       : CN=appcontrol-ca01-CA, DC=appcontrol, DC=lab
Thumbprint   : 3078086484D9242BD39D8971C419E07C61096BDA
FriendlyName :
NotBefore    : 10/15/2024 11:47:56 AM
NotAfter     : 10/15/2025 11:47:56 AM
Extensions   : {System.Security.Cryptography.Oid, System.Security.Cryptography.Oid, System.Security.Cryptography.Oid,
               System.Security.Cryptography.Oid...}

```

Now, when we have found our certificate we want to assign it to a variable

```powershell
$cert = Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert
Set-AuthenticodeSignature -Certificate $cert -TimestampServer http://timestamp.digicert.com -FilePath C:\install\script.ps1
```

And thats it! Our script is signed and timestamped.

Check the signing of the script:
```powershell
Get-AuthenticodeSignature C:\install\script.ps1 | select *
```

Now open up our trusty WDAC Policy Wizard again. 
Create a new Base policy with the Default Windows Mode template.
Make sure that "Disable Script Enforcement" is not checked and that the policy is not in Audit mode.
In Files Rules, click on +Add Custom Rule
Under Custom Rule Conditions, Uncheck Kernel Mode and set the rule type to Publisher.
Click on Browse next to the "reference file" textbox and browse to the newly signed C:\Install\script.ps1
Click on Create Rule to whitelist everything signed by the CodeSign certificate.
Press Next to build a create the policy.

Deploy the policy and refresh with a tool or reboot the computer to apply the policy.

If everything is working correctly, you should be able to run the signed script and it will "spit out" FullLanguage.
Create a new script that you don't sign and paste the same content as we did at the beginning of this lab. The unsigned script should write "ConstrainedLanguageMode" to the powershell prompt.