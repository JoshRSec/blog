---
title: "Finding users in AD and adding to distribution group"
date: 2019-10-30
summary: "A simple PowerShell script that pulls users from Active Directory, checks whether they’re already members of an Exchange Online distribution group, and adds any missing users—making it easy to keep cloud‑hosted groups aligned with on‑prem AD."
description: "A simple PowerShell script that pulls users from Active Directory, checks whether they’re already members of an Exchange Online distribution group, and adds any missing users—making it easy to keep cloud‑hosted groups aligned with on‑prem AD."
slug: "finding-users-in-ad-and-adding-to-distribution-group"
aliases:
  - "/finding-users-in-ad-and-adding-to-distribution-group"
categories:
  - "Powershell"
  - "Windows"
---

I had a scenario in which a group of users needed to be added to a distribution group hosted on Exchange online.

So I thought Id made a little script which pulls users from the local Active Directory; checks them against the distribution group and if they are not present, adds them.

You may want to customize the "-SearchBase" parameter depending on your requirements. You could also pipe the ADUsers with "where" to get users with a specific property such as location.

{{< gist JoshRSec d5736368a179772029815dcc497e962f >}}
