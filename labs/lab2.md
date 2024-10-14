---
title: Lab 2: Finding the policies on disk
---

There are a few places a deployed policy can exist in, depending on the type of policy and the configuration:

```
<OS Volume>\Windows\System32\CodeIntegrity\CiPolicies\Active\{PolicyId GUID}.cip
<OS Volume>\Windows\System32\CodeIntegrity\SiPolicy.p7b
<OS Volume>\Windows\System32\CodeIntegrity\driversipolicy.p7b
<EFI System Partition>\Microsoft\Boot\CiPolicies\Active\{PolicyId GUID}.cip

<EFI System Partition>\Microsoft\Boot\SiPolicy.p7b
<EFI System Partition>\Microsoft\Boot\driversipolicy.p7b
```

To view the contents of the EFI-partition you can mount it by running the command 

> mountvol P: /S

1. See if you can find all the corresponding policy files for the policies we listed with citool.exe in the previous exercise