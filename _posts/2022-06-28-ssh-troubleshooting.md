---
layout: post
title:  "SSH Troubleshooting"
categories: Systems Administration
author: Uchechukwu Onyekwuluje
tags: Administration
---
Troubleshooting Tips for SSH Issues

#### **Removing hosts from known_hosts file**
When a server is rebuilt and you try ssh with the same name or IP, you may run into an error like this
```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:a7RlObbYihpR9044e6LcxeIFwUUQnRT95Hho7+8aNxk.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /home/user/.ssh/known_hosts:96
```
Remove the offending key with this command. For example IP `192.168.1.84`
```
ssh-keygen -f "/home/user/.ssh/known_hosts" -R "192.168.1.84"
```
