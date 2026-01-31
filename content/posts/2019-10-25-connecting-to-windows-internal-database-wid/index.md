---
title: "Connecting to Windows Internal Database (WID)"
date: 2019-10-25
slug: "connecting-to-windows-internal-database-wid"
aliases:
  - "/connecting-to-windows-internal-database-wid"
categories:
  - "Windows"
tags:
  - "SQL Server Management Studio"
  - "SSMS"
  - "WID"
  - "Windows"
  - "Windows Internal Database"
  - "Windows Server"
---

Whilst trying to figure out issues with a service which uses Windows Internal Database (WID) I came into the issue of actually connecting and managing it...

Steps to connect to WID:

 	* Download and install Microsoft SQL Server Manager ([found here](https://docs.microsoft.com/en-us/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-ver15)) onto the machine hosting the instance

 	* Run Microsoft SQL Server Manager (SSMS) as **Administrator** (if not run as Administrator a permissions exception will be thrown when connecting)

 	Enter the connection string, this will be different depending on the Windows version:

 	* Windows 2003 - Windows 2008: \\.\pipe\MSSQL$MICROSOFT##SSEE\sql\query

 	* Windows 2012 onward: \\.\pipe\MICROSOFT##WID\tsql\query



 	* Connect using Windows Authentication
