---
title: Lab 3 - Signing an app using catalog signing
parent: Module 4
layout: home
nav_order: 
nav_enabled: true
---

## Download Windows SDK (to get signtool.exe)

https://developer.microsoft.com/sv-se/windows/downloads/windows-sdk/

Click on "Ladda ner installationsprogrammet" :)
Open winsdksetup.exe
Chose Install and press Next, Next, Accept

Uncheck everything except "Windows SDK Signing Tools for Desktop Apps" and click Install
![alt text](/img/mod4-lab3-img1.jpg)

Close the window.

Open a new Terminal and verify that you now have access to signtool.exe.
It should be located in 
> `C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64`


## Install and create a catalog file for an application:

Let's download a real classic applicaton to use for our catalog signing - Total Commander. We are going to pretend this is a totally unsigned legacy application that we need to sign ourselves by creating a catalog file.

Start by downloading Total Commander from here [https://totalcommander.ch/1103/tcmd1103x64.exe](https://totalcommander.ch/1103/tcmd1103x64.exe). Save it to C:\Install

Start packageinspector like we did in Module 3:
`packageinspector.exe start C: -path C:\install\tcmd1103x64.exe`

Start the installer of Total Commander by doubleclicking `C:\install\tcmd1103x64.exe`, confirm all defaults and Press OK to close the installation when the Installation was successful.

Stop PackageInspector and provide a path to a .cat-file and a .cdf-file
`PackageInspector.exe stop c: -out cat -name c:\install\totalcmd.cat -cdfpath c:\install\totalcmd.cdf`

## Inspect and Sign the catalog file

## FELSÖKNING TÅG:


Get the filehashes of all the installed files:
get-childitem -recurse 'C:\Program Files\totalcmd\' | Get-FileHash

Doubleclick c:\install\totalcmd.cat and click the Security Catalog tab.
Can you find some of the hashes? Why or why not?

Download and install CFF Explorer:
https://ntcore.com/files/ExplorerSuite.exe

Open CFF Explorer on the start menu and open one of the files you *did'nt* find a matching hash for.


Sign the cat file:

& 'C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64\signtool.exe' sign /n "CodeSign" /fd sha256 /v C:\install\totalcmd.cat

That should result in this message:
```
The following certificate was selected:
    Issued to: CodeSign
    Issued by: appcontrol-ca01-CA
    Expires:   Wed Oct 15 11:47:56 2025
    SHA1 hash: 3078086484D9242BD39D8971C419E07C61096BDA

Done Adding Additional Store
Successfully signed: C:\install\totalcmd.cat

Number of files successfully Signed: 1
Number of warnings: 0
Number of errors: 0
```

Doubleclick C:\install\totalcmd.cat
Click on View Signature and see that the file is now signed with our CodeSign-certificate and that the signature is valid.

In order for a computer to trust a catalog file, Windows needs to know of it's existance. For Kernel drivers, that cat file is always stored in the same directory as the other driver files. For our own signed catalogs, we need to distribute the cat file to `%windir%\System32\catroot\{F750E6C3-38EE-11D1-85E5-00C04FC295EE}`


`copy-item C:\install\totalcmd.cat "c:\windows\System32\catroot\{F750E6C3-38EE-11D1-85E5-00C04FC295EE}"`

If we run 
`Get-AuthenticodeSignature 'C:\Program Files\totalcmd\TOTALCMD64.EXE' | select *`, we unfortunatly see the binary as AuthentiCode signed. Thats because i couldn't find any good example apps.

## Add our signing certificate to our App Control Policy

```powershell
Add-SignerRule -FilePath 'C:\Policies\Mod3Lab2-Win11-Base.xml' -CertificatePath C:\Policies\codesign-public.cer -User
```

Open 'C:\Policies\Mod3Lab2-Win11-Base.xml' and find our singning certificate under <Signers>
Scroll down to SigningScenario value 12 (this is the User Mode part of the policy) and verify that the SignerId is also stored there.


