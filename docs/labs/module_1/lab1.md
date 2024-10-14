---
title: Lab 1 - Viewing active policies
parent: Module 1
layout: home
nav_order: 1
nav_enabled: true
---

# Lab 1 - Viewing active policies


In this lab we will look at how we can see what policies are applied in Windows. We will also look at where we can find them in the file system.

1.	Start by opening an elevated command prompt. As we are running Windows 11 on our lab machines we have access to citool.exe which is a tool you can use to view and refresh policies.
2.	Run the following command to view what policies are currently applied:
citool –list-policies
Note: if you get an error code, like the one below, you need to reopen the command prompt as an administrator
PS C:\Users\RobinEngström> citool --list-policies
An error occurred: 0x80070005
Press Enter to Continue
If you get weird error codes in numeral format you can use the net helpmsg or certutil command to lookup the error code in human readable format

```
PS C:\Users\RobinEngström> net helpmsg 5

Access is denied.

PS C:\Users\RobinEngström> certutil -error  0x80070005
0x80070005 (WIN32: 5 ERROR_ACCESS_DENIED) -- 2147942405 (-2147024891)
Error message text: Access is denied.
```

When citool runs successfully you will be presented with a list of the currently applied policies on the machine. You will also see the name and version of the policies and if they are included in the Platform or not.

3.	Compare the applied policies with Microsofts list of inbox policies:
Inbox App Control policies | Microsoft Learn
Note that some are Base policies and some are supplemental.
4.	Now open Event Viewer and take a look at the Microsoft-Windows-CodeIntegrity/Operational log (Applications and Services Logs > Microsoft > Windows > CodeIntegrity > Operational).
5.	Look at all events with the 3099 event id and verify that we get a log event each time a policy is applied or refreshed. Also note that the name of the policy is written to the event log.
6.	Run msinfo32.exe and scroll down to the bottom. 
 
This means that we have one or more policies that are configured in Enforce mode, but no policy configured for User Mode Code Inteigry.
