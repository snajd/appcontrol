---
title: Lab 2 - Signing a PowerShell Script and Constrained Language Mode
parent: Module 4
layout: home
nav_order: 2
nav_enabled: true
---

# Lab 2 - Signing a PowerShell Script and Constrained Language Mode

## Script Enforcement

Skapa en base policy i WDAC Policy Wizard som inte har Script Enforcement:Disabled

## Skapa ett powershell script som bara spottar ur sig

$Executioncontext.session.lanaguagemode

## Skriv lite om CLM

PowerShell använder inte regelverket i policyn utan vad _datorn litar på_


## Sign a PowerShell script

Make sure that your CodeSigning certificate is installed in the Personal store of the logged on user.
Open a PowerShell prompt
List your installed code signing certificates:

> Get-ChildItem Cert:\CurrentUser\My -CodeSigningCert

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
```
Set-AuthenticodeSignature -Certificate $cert -TimestampServer http://timestamp.digicert.com -FilePath C:\Policies\Webinar\webinar.ps1


Get-AuthenticodeSignature C:\Policies\Webinar\webinar.ps1 | select *




SKAPA ETT SCRIPT SOM SKRIVER UT VILKEN EXECUTIONCONTEXT VI KÖR I!!!