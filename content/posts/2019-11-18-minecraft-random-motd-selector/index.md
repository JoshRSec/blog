---
title: "Minecraft Random MOTD Selector"
date: 2019-11-18
slug: "minecraft-random-motd-selector"
aliases:
  - "/minecraft-random-motd-selector"
categories:
  - "Powershell"
tags:
  - "Minecraft"
  - "PowerShell"
---

A fun micro-project, this Powershell script will select a random message of the day (MOTD) and write it to disk. Being an addition to the MC process monitoring script the scipt runs everytime the server restarts (Every 12 hours), also is engaging for the community to submit phrases and see their submissions displayed.

Below is the script, personally I placed it within a function and called it when required. Its pretty self-explanatory, I would like to mention more time was spent diagnosing why MC would have a fit and corrupt the file. Then I noticed the server.properties file was not being encoded in UTF-8, once amended it worked flawlessly.

{{< gist JoshRSec 05649ad69d5e2d7d1fc2917abba47240 >}}

{{< gist JoshRSec 1cb64dc13a9a267156bed0026005a91a >}}

{{< gist JoshRSec ec2afa2adf6fcfef25f0c772d57ceaf0 >}}

The script also allows specific date overrides, which could be used for holidays and server related events.

MOTD.txt is used to store the MOTD's, which are separated by a new line.

ColourChart.txt contains the MOTD colours which are randomly selected and applied to the text.
