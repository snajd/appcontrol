---
title: Lab 3 - Signing an app using catalog signing
parent: Module 4
layout: home
nav_order: 3
nav_enabled: true
---

# Lab 3 - Signing an app using catalog signing

## Download Windows SDK (to get signtool.exe)

[Download the Windows 11 SDK here](https://developer.microsoft.com/sv-se/windows/downloads/windows-sdk/)

1. Click on "Ladda ner installationsprogrammet" :)
2. When downloaded, open `winsdksetup.exe`
3. Chose Install and press Next, Next, Accept

4. Uncheck everything except "Windows SDK Signing Tools for Desktop Apps" and click Install
![alt text](/img/mod4-lab3-img1.jpg)

5. Close the window.

Open a new Terminal and verify that you now have access to signtool.exe.
It should be located in 
> `C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64`


## Install and create a catalog file for an application:

It turs out, modern application developers are pretty good at signing their binaries, but I managed to find a software that is unsigned and that we also can use in this lab - CFF Explorer. 

Start by downloading CFF Explorer from here [https://ntcore.com/files/ExplorerSuite.exe](https://ntcore.com/files/ExplorerSuite.exe). Save it to C:\Install

Start packageinspector like we did in Module 3:
`packageinspector.exe start C: -path C:\install\ExplorerSuite.exe`

Start the installer of CFF Explorer by doubleclicking `C:\install\ExplorerSuite.exe`, confirm all defaults and Press Finish to close the installation when the Installation was successful.

Stop PackageInspector and provide a path to a .cat-file and a .cdf-file
`PackageInspector.exe stop c: -out cat -name c:\install\cffexplorer.cat -cdfpath c:\install\cffexplorer.cdf`

## Inspect and Sign the catalog file

Get the filehashes of all the installed files:

```powershell
get-childitem -recurse Get-ChildItem -Recurse "C:\Program Files\NTCore\Explorer Suite" | Get-FileHash
```

Doubleclick c:\install\cffexplorer.cat and click the Security Catalog tab.
Can you find some of the hashes? Why or why not?

Open CFF Explorer on the start menu and open one of the files you *did'nt* find a matching hash for.
CFF Explorer can display the SHA-1 or MD5 embedded hash in the files.

## Sign the cat file:

& 'C:\Program Files (x86)\Windows Kits\10\bin\10.0.26100.0\x64\signtool.exe' sign /n "CodeSign" /fd sha256 /v C:\install\cffexplorer.cat

That should result in this message:
```
The following certificate was selected:
    Issued to: CodeSign
    Issued by: appcontrol-ca01-CA
    Expires:   Wed Oct 15 11:47:56 2025
    SHA1 hash: 3078086484D9242BD39D8971C419E07C61096BDA

Done Adding Additional Store
Successfully signed: C:\install\cffexplorer.cat

Number of files successfully Signed: 1
Number of warnings: 0
Number of errors: 0
```


Doubleclick C:\install\cffexplorer.cat
Click on View Signature and see that the file is now signed with our CodeSign-certificate and that the signature is valid.

In order for a computer to trust a catalog file, Windows needs to know of it's existance. For Kernel drivers, that cat file is always stored in the same directory as the other driver files. For our own signed catalogs, we need to distribute the cat file to `%windir%\System32\catroot\{F750E6C3-38EE-11D1-85E5-00C04FC295EE}`


If we run 
```powershell
Get-AuthenticodeSignature "C:\Program Files\NTCore\Explorer Suite\Task Explorer.exe"


    Directory: C:\Program Files\NTCore\Explorer Suite


SignerCertificate                         Status                                               Path
-----------------                         ------                                               ----
                                          NotSigned                                            Task Explorer.exe
```


Copy the signed cat file to catroot:
Microsofts have a few suggestions on how this can be done with Group Policy or ConfigMgr [here](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/deployment/deploy-catalog-files-to-support-appcontrol)

`copy-item C:\install\cffexplorer.cat "c:\windows\System32\catroot\{F750E6C3-38EE-11D1-85E5-00C04FC295EE}"`


Now, let's use New-CIPolicy to scan the directory and create a App Control Policy for us:

```powershell
New-CIPolicy -FilePath c:\policies\apps\explorer.xml -ScanPath "C:\Program Files\NTCore\Explorer Suite" -level Publisher -NoShadowCopy -fallback Hash
```

When the command completes. Open up ´c:\policies\apps\explorer.xml` and see that we only got one single rule created:
```xml
 <Signer ID="ID_SIGNER_S_3" Name="CodeSign">
      <CertRoot Type="TBS" Value="D7AE1CE5B50C772F7CF195CCE6CE1784FD9015235CE07EF5B1464D534C09C24C" />
      <CertPublisher Value="CodeSign" />
    </Signer>
```

## Examine the effect of the cat file

How would the App Control policy look if the app we just signed wasn't signed?
We can test that pretty easily by just removing the cat file and run the New-CIPolicy command again.

1. Start by deleting the cat
```powershell
del "c:\windows\System32\catroot\{F750E6C3-38EE-11D1-85E5-00C04FC295EE}\cffexplorer.cat"
```
2. Run the same command as above to generate a App Control policy
```powershell
New-CIPolicy -FilePath c:\policies\apps\explorer-nocat.xml -ScanPath "C:\Program Files\NTCore\Explorer Suite" -level Publisher -NoShadowCopy -fallback Hash
```




Optionally:
Deploy and test the policy