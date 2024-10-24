---
title: Module 6 - Bonus labs
parent: Labs
layout: home
nav_order: 1
nav_enabled: true
---


# Module 6 - Extra Lab - Do what you want!

Now that you know how to white list things in almost every way possible you can try things out for yourself.


Here are some suggestions on what you now can try out for yourself:

1. Find some applications that you want to package and try to see if you can create your own policies.
2. Download the whole LOLDrivers Git repository and create your own complete block list (Use New-CIPolicy and Hash rules. Remember that these rules needs to be for KMCI)
3. Create an AppId Tagging policy by following the instructions [here](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/appidtagging/design-create-appid-tagging-policies). See if you can tag some of the LOLBINs from Module 2 Lab 4. Then use `Net-NewFirewallRule -PolicyAppId` to create local firewall rules based on the tag you created.
4. If you are really bored: try to sign a App Control policy! Follow Microsofts instructions [here](https://learn.microsoft.com/en-us/windows/security/application-security/application-control/app-control-for-business/deployment/use-signed-policies-to-protect-appcontrol-against-tampering). You should have all the needed tools and certificates ready from the previous labs.
