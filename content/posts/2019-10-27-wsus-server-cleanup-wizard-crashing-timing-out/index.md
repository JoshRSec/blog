---
title: "WSUS Server Cleanup Wizard Crashing/Timing out"
date: 2019-10-27
summary: "A quick note on resolving WSUS cleanup failures caused by long‑running SQL queries timing out; the fix involves connecting to the Windows Internal Database and increasing the query timeout value in the SQL instance properties so WSUS can complete its cleanup operations without crashing."
description: "A quick note on resolving WSUS cleanup failures caused by long‑running SQL queries timing out; the fix involves connecting to the Windows Internal Database and increasing the query timeout value in the SQL instance properties so WSUS can complete its cleanup operations without crashing."
slug: "wsus-server-cleanup-wizard-crashing-timing-out"
aliases:
  - "/wsus-server-cleanup-wizard-crashing-timing-out"
categories:
  - "Windows"
tags:
  - "Error"
  - "WID"
  - "WSUS"
  - "WSUS Maintainance"
  - "WSUS Server Cleanup"
---

Whilst trying to cleanup and remove older Windows updates to free up space on my WSUS box I ran into the issue of WSUS crashing with the following exception:

![WSUS WID Timeout message](img/SQLServer-error.png)

The WSUS administration console was unable to connect to the WSUS Server Database.
Verify that SQL server is running on the WSUS Server. If the problem persists, try restarting SQL.



System.Data.SqlClient.SqlException -- Execution Timeout Expired. The timeout period elapsed prior to completion of the operation or the server is not responding.



Source
.Net SqlClient Data Provider




Stack Trace:
at System.Windows.Forms.Control.MarshaledInvoke(Control caller, Delegate method, Object[] args, Boolean synchronous)
at System.Windows.Forms.Control.Invoke(Delegate method, Object[] args)
at Microsoft.UpdateServices.UI.SnapIn.Wizards.ServerCleanup.ServerCleanupWizard.OnCleanupComplete(Object sender, PerformCleanupCompletedEventArgs e)

Which from my understanding the query is taking too long to complete and the SQL server is simply timing out... To correct this we need to connect to the WID and extend the timeout time.

Steps to increase the query timeout time:

 	* Connect to the WID ([Steps here](https://blog.joshuarobbins.tech/?p=219))

 	Right-Click the SQL instance and select properties

![Selecting the properties of the SQL instance](img/WID-SSMS-1.png)
 	Go to connections, then modify the timeout time to 1200 or greater (0 can prevent timeouts but is not recommended

![Changing the query timeout](img/WID-SSMS-2.png)
