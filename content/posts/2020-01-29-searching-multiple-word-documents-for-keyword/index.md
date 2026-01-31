---
title: "Searching multiple Word documents for keyword"
date: 2020-01-29
slug: "searching-multiple-word-documents-for-keyword"
aliases:
  - "/searching-multiple-word-documents-for-keyword"
categories:
  - "Powershell"
  - "Windows"
tags:
  - "Docx"
  - "Microsoft Word"
  - "PowerShell"
  - "Word"
---

Found myself in a situation I needed to search 20+ documents for a keyword and unfortunately Word cannot search files within a directory for keywords, only the current document.

Thankfully we can automate this with PowerShell.

The script below will:

 	* Accept a directory path and keyword

 	* Open each file found and search for the keyword

 	* If the file contains the keyword it will add to an array of files containing the keyword

 	* Output results


A caveat with this approach is Word needs to be installed on the machine in order to open an instance and then search, but I would imagine if the machine is storing Word documents then it likely has Word installed!

&nbsp;

{{< gist JoshRSec ec54bda71c5254c241b2ad2191dc7362 >}}
