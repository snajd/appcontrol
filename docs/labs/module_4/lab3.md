---
title: Lab 3 - Signing an app using catalog signing
parent: Module 4
layout: home
nav_order: 
nav_enabled: true
---

distribuera cat-filen

ladda hem SDK och kopia ut signtool.exe så folk kan tanka hem dem.

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