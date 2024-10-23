---
title: Lab 1 - Prepare for the code signing labs
parent: Module 4
layout: home
nav_order: 1
nav_enabled: true
---


# Lab 1 - Prepare for the code signing labs

The certificates you are about to download is created by simply following this instruction: [https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/deployment/create-code-signing-cert-for-appcontrol](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/deployment/create-code-signing-cert-for-appcontrol)

[CA Root Certificate](https://appcontrollabs.snajd.net/downloads/ca01-LAB.crt)
[Code Signing Certificate](https://appcontrollabs.snajd.net/downloads/codesign.pfx)

Download the certificate.

## Importing the Code Singning certificate

Doubleclick codesign.pfx, chose Current User as the store location and press next two times.
Enter "Password01" when prompted for a password. Accept the other options and press next.
Chose to automatically select the store and press next.
In the warning that is displayed, chose Yes. This will install the Root CA certificate in the users Trusted Root store.

Open certmgr.msc. Expand Personal > Certificates. Verify that you now have a certificate in your personal store called CodeSign, issues by appcontrol-ca01-CA.

Right click the CodeSign certificate and chose All Tasks > Export. Press Next three times and export the public part of the certificate to C:\Policies\codesign-public.cer. Press Next and then Finish.

Now browse to Trusted Root Certificate Authorities and find appcontrol-ca01-CA. Export that certificate like you did in the previous step, but name it c:\policies\ca01.cer.

Now we want to import these two certificates in the Computer certificate store, instead of just for our logged on user.
Open `certlm.msc`

Expand Trusted Root Certificate Authorities and right click on Certificates. Chose All Tasks and then import. Continue the wizard and chose to install C:\Policies\ca01.cer when prompted for a file.

Now do the same thing but import the C:\Policies\codesign-public.cer in Trusted Publishers.


What we now have is:
1. One complete certificate, including the private key that can be used for Codesigning in our personal store
2. The certificate for the CA that have issued the Code Signing certificate is trusted by our computer
3. Things signed with our codesigning certificate will be trusted because the public part of that certificate is placed in Trusted Publishers for the computer.

Now let's move on to the next lab and sign some stuff!






länka till hur jag gjorde certen

Lägg till certen till policy

undersök en fil och se vem som signerat den

