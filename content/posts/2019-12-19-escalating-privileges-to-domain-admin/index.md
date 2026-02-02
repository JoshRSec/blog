---
title: "Escalating Privileges to Domain Admin"
date: 2019-12-19
summary: "A quick overview of how local admin rights on a workstation can be misused to escalate privileges when a Domain Admin logs in, along with the reminder that proper least‑privilege policies prevent this risk."
description: "A quick overview of how local admin rights on a workstation can be misused to escalate privileges when a Domain Admin logs in, along with the reminder that proper least‑privilege policies prevent this risk."
slug: "escalating-privileges-to-domain-admin"
aliases:
  - "/escalating-privileges-to-domain-admin"
categories:
  - "Security"
  - "Windows"
tags:
  - "Active Directory"
  - "Security"
  - "Windows"
---

This method may benefit from some social engineering but will require local Administrator on an machine within the network. Social engineering can be used to speedup the process of enticing a Domain Admin (DA) to login to a system and in-return provide us DA rights.

Should you need to obtain local Administrator privileges and you have access to a machine which isn't using Bit-locker, [here is a guide on doing so](https://blog.joshuarobbins.tech/gaining-entry-into-windows-with-administrative-permissions) (It wont take long).

- Create a new scheduled task
- Ensure the "Run with highest privileges" checkbox is checked

![Creating a new Scheduled Task with Run as Highest Privileges ](img/TaskWithRunAsHighPriv.png)
 	Change the user in which the task will run as to that of the target DA account

![Changing the user which the task will be run by to that of a Domain Admin](img/ChangeUSERtoDAAdmin.png)
 	Set the trigger to be when the target DA logs onto the machine

![Creating the trigger for when the Domain Admin logs into the machine](img/CreatingAtLogonTask.png)
Set the action to run 'net.exe' -> add the parameter 'group "Domain Admins" user /add /domain'


![Defining the action for when the Domain Admin logs into the machine](img/CreatingAction.png)

![How the action should look when entered correctly](img/CreatingActionOverview.png)

Now we lay in wait. Perhaps you have Admins which use their DA accounts for remote support, you could always raise a ticket and get them to remote onto the machine. You could also set this trap on a machine which you know the DAs will login to (Provided you have local Admin).

![User mcUser Face's security groups before the DA logged into the machine](img/userGroupMembershipBefore.png)

![User mcUserFace's security groups after the DA logged into the machine](img/userGroupMembershipAfter.png)

This method can be mitigated through blocking Domain Admins from logging into workstations by group policy and following the principle of least privilege.
