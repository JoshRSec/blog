---
title: "Automating Log File Archiving - PowerShell"
date: 2019-10-10
slug: "automating-log-file-archiving-powershell"
aliases:
  - "/automating-log-file-archiving-powershell"
categories:
  - "Powershell"
tags:
  - "Archiving Logs"
  - "File Management"
  - "PowerShell"
  - "Windows"
---

The purpose of this PowerShell script was to automate the cleaning of logs files by zipping them and shipping them to another folder or drive, but to also give the option to delete the source files after being processed.

The script has the option to group the files by the day they were written or to archive them by individual file, which can be useful if the source of the logs is particularly spammy.

{{< gist JoshRSec 5948ef3a0f57d629229d56ff538f7a99 >}}
## Example

I noticed my webserver had been filling up over the last few months, so for the purpose of an example of this scripts usefulness I put this script to use (despite IIS having the functionality to compress folders, but I needed an example)
### Before

![Log files before compression](img/LogFilesBeforeZIP-1.png)
### After

![Log files after compression](img/LogFilesAfterZIP-1.png)
### Command used

{{< gist JoshRSec d0424b60c8605dbd929622d3ab9f1a8d >}}
