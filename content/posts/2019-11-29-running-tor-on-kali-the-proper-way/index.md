---
title: "Running TOR on Kali | The proper way"
date: 2019-11-29
summary: "A short guide on running the Tor Browser safely by creating a non‑root user and launching Tor under that account, ensuring it never runs with elevated privileges."
description: "A short guide on running the Tor Browser safely by creating a non‑root user and launching Tor under that account, ensuring it never runs with elevated privileges."
slug: "running-tor-on-kali-the-proper-way"
aliases:
  - "/running-tor-on-kali-the-proper-way"
categories:
  - "Security"
tags:
  - "How-To"
  - "Install"
  - "Kali"
  - "Non-Root"
  - "TOR"
---

TOR shouldn't be run as root. Many guides gloss over this by removing the root check in the **start-tor-browser.desktop** file, here is the better way:

 	* Create a new user: adduser --home-dir /home/kali kali

 	* Download and extract TOR from [https://www.torproject.org/projects/torbrowser.html.en](https://www.torproject.org/projects/torbrowser.html.en)

 	* Add the newly created user to the **xhost** file: xhost si:localuser:kali

 	* Copy the extracted TOR files to the home directory for the new user

 	* Change the permissions for the TOR files sudo chown -R /home/kali

 	* Run TOR with the command: sudo -u kali -H /home/kali/start-tor-browser.desktop

 	* Verify TOR is being run with the new user account by running: ps aux | grep /tor


Now TOR can be run by an account without root permissions
