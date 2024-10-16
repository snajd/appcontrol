---
title: Lab 4 - LOLDrivers
parent: Module 2
layout: home
nav_order: 4
nav_enabled: true
---


If you have the [Attack Surface Reduction rule](https://learn.microsoft.com/en-us/defender-endpoint/attack-surface-reduction-rules-reference) "Block abuse of exploited vulnerable signed drivers" configured, and Windows Defender Antivirus real-time protection configured, certain vulnerable drivers will be blocked from being written to the file system. That ASR-rule doesn't protect from drivers that already exists in the file system.

> If you are curious on how ASR-rules works at a technical level you can dissect them by following the steps [here](https://adamsvoboda.net/extracting-asr-rules/)