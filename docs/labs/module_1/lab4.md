---
title: Lab 4 - Deploying a policy in Audit mode
parent: Module 1
layout: home
nav_order: 4
nav_enabled: true
---

# Lab 4 - Deploying a policy in Audit mode


test powershell

{% highlight powershell %}

# Kopiera en Audit policy som tillåter allt som kommer med Windows
copy-item C:\Windows\schemas\CodeIntegrity\ExamplePolicies\DefaultWindows_Audit.xml c:\Policies\Webinar\

# Döp om filen till något mer hanterbart

move-item c:\policies\Webinar\DefaultWindows_Audit.xml -destination c:\policies\webinar\WebinarAudit.xml

# Sätt ett trevligt namn och version och nollställ PolicyID
Set-CIPolicyIdInfo -FilePath c:\policies\webinar\WebinarAudit.xml -PolicyName "Webinar Audit Policy" -PolicyId "20241010" -ResetPolicyID

# Konvertera XML till binärt format
ConvertFrom-CIPolicy -XmlFilePath C:\Policies\Webinar\WebinarAudit.xml -BinaryFilePath C:\Policies\webinar\WebinarAudit.bin

# En policy måste heta {PolicyID}.clp
$policyid = ([xml]$id = get-content C:\Policies\webinar\WebinarAudit.xml).SiPolicy.PolicyID
Move-Item C:\Policies\Webinar\WebinarAudit.bin -Destination "C:\Policies\Webinar\$policyid.clp"

# Kopiera policy till rätt sökväg (kräver elevering)
Copy-Item "C:\Policies\Webinar\$policyid.clp" -Destination C:\windows\system32\CodeIntegrity\CiPolicies\Active


{% endhighlight %}
