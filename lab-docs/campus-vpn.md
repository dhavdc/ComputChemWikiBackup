---
title: Campus VPN
description: 
published: true
date: 2020-10-08T17:06:29.521Z
tags: 
editor: markdown
dateCreated: 2020-08-25T15:35:56.940Z
---

# Campus VPN
In Mac OS and Windows, you can install Cisco Anyconnect following the instructions on https://www.umaryland.edu/cits/service-catalog/vpn/

In Linux systems, you can install `openconnect` instead. For Debian based distros like Ubuntu, you can install as below
```
sudo apt-get update
sudo apt-get install openconnect
```
And after it finishes succesfully, you can type
```
sudo openconnect vpn.umaryland.edu
```
and then type in 'RX' for the specific group, your UMID for 'username', UMB website password for 'password', and 'PUSH' for 'password' (yes again).  If you have DUO installed in your mobile device, approve it and then you can SSH to lab workstations and use school libraries.