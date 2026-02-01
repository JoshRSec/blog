---
title: "Setting up OpenVPN on Kali"
date: 2019-11-28
summary: "A straightforward walkthrough for configuring OpenVPN on Kali or Debian by placing the provider’s certificates and config files in /etc/openvpn, setting up credential handling, enabling DNS updates, and configuring the service to auto‑start so the VPN connects reliably at boot."
description: "A straightforward walkthrough for configuring OpenVPN on Kali or Debian by placing the provider’s certificates and config files in /etc/openvpn, setting up credential handling, enabling DNS updates, and configuring the service to auto‑start so the VPN connects reliably at boot."
aliases:
  - "/setting-up-openvpn-on-kali"
categories:
  - "Security"
tags:
  - "Kali"
  - "OpenVPN"
  - "Set-up"
  - "VPN"
---

Here is how to set-up OpenVPN on Kali. The process is (*unsurprisingly* the same for Debian), steps below:

 	Obtain OpenVPN certificates, key and openvpn.ovpn files from the provider

 	**ca.crt:** This is the certificate of the certification authority

 	* **client.crt:** This is the user certification file 

 	* **client.key:** This is your private key file

 	* **openvpn.ovpn:** This is your **OpenVPN** configuration file



 	* Rename the **openvpn.ovpn **config file to something identifiable for the server being used (Server.conf in this example)

 	Copy the downloaded files to the **/etc/openvpn/** directory

 	* sudo cp Server.conf /etc/openvpn/

 	* sudo cp ca.cert /etc/openvpn/

 	* sudo cp client.crt /etc/openvpn/

 	* sudo cp client.key /etc/openvpn/



 	Update the  packages on Kali

 	* sudo apt-get update



 	Ensure OpenVPN is installed with the required packages

 	* sudo apt-get install openvpn openssl openresolv



 	Change directory to **/etc/openvpn/**

 	* cd /etc/openvpn



 	Create a new file for the credentials to access the VPN

 	* sudo nano user.txt

 	Enter the credentials in the following format:

 	* Username

 	* Password





 	Open the Server.conf file and make the following changes:

 	* Change auth-user-pass -> auth-user-pass /etc/openvpn/user.txt

 	Under **comp-lzo **add the lines

 	* up /etc/openvpn/update-resolv-conf

 	* down /etc/openvpn/update-resolv-conf





 	Now we need to point OpenVPN to this config file

 	* sudo nano /etc/default/openvpn

 	* Add the line AUTOSTART="Server" (without the .conf)



 	Ensure the DNS resolver is being pointed to your gateway

 	* Run cat /etc/resolv.conf

 	* If needed modify the file



 	* Run the command: sudo update-rc.d openvpn enable

 	* Now we can start the service with:  sudo service openvpn start

 	Check the service is running by running: systemctl status openvpn@server

 	* "Server" being the name of the .conf file being used

 	If successful the output should look similar to:

 	openvpn@server.service - OpenVPN connection to server
Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset: enabled)
Active: active (running) since Wed 2019-11-27 00:00:00 GMT; 1h 00min ago
Docs: man:openvpn(8)




 	* Check for DNS leaks by visiting: https://dnsleaktest.com/
