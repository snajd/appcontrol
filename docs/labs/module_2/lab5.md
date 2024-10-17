---
title: Lab 5 - LOLBINS/LOLBAS
parent: Module 2
layout: home
nav_order: 5
nav_enabled: true
---


# Lab 5 - Living of the Land Binaries

One cannot talk about a list of known bypasses without giving some more info. 
In the security community, we often talk about "living of the land", which simply means that instead of an threat actor (or red teamer) downloads their own tools to a compromised system, they instead use the built in tools and capabilities in the operating system. This is much harder to detect and doesn't trigger antiviruses. Nowadays with the prevalance of EDR:s, they can still trigger some alarms if they for instance use certutil.exe to download a file from the internet.

A list of known LOLBINS and example on how to use them, for Windows can be found [here](https://lolbas-project.github.io/)

Lolbins are a real hassle for us defenders, because if block them with App Control the operating system would get unusable.
We can use AppLocker and block most of them for Normal user but still allow them for Administrators or other groups of users.

Another smart way is to block lolbins in the local firewall from talking to the internet. And we can actually use App Control for just that. More on that in a later lab.


There is also another related project by the security researcher @bohops: [The Ultimate WDAC Bypass List](https://github.com/bohops/UltimateWDACBypassList)

Bohops have tried to find information on all the stuff Microsoft adds to the Recommended Block list and how you can exploit them.

GÖR EN LABB MED EN BYPASS. Kanske räcker med bash.exe? Svårt när det kräver virtualisering.

