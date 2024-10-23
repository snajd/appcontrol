---
title: Lab 6 - Whitelist Kernel Driver
parent: Module 3
layout: home
nav_order: 6
nav_enabled: false
---

# NOT READY MOVE ON TO THE NEXT


WIRESHARK!

Get-WDACCodeIntegrityEvent -SinceLastPolicyRefresh -Kernel


mkdir c:\wdac\whitelistdrivers
$WDACEvents = Get-WDACCodeIntegrityEvent -SinceLastPolicyRefresh -Kernel
$WDACEvents.FilePath | ls | cp -Destination C:\wdac\whitelistdrivers

# Nu har vi filerna vi vill vitlista i en folder
$SignerInfo = Get-SystemDriver -ScanPath C:\wdac\whitelistdrivers -NoScript -NoShadowCopy
New-CIPolicy -FilePath c:\wdac\whitelist.xml -DriverFiles $SignerInfo -Level WHQLFilePublisher -Fallback FilePublisher
