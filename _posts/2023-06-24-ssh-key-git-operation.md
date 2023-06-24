---
layout: post
title:  "Git ops with specific key"
categories: systems Administration
author: Uchechukwu Onyekwuluje
---

Working with multiple projects and repositories is common for most DevOps/SysOps/SysEng/Development tasks. One problem I ran into
was working with non-default SSH keys. This threw me for a loop because the operating systems I used in the past took care of this.
In the event your operating system does not or if you need a different key for other reasons, the command below should fix it

### List Regions
```
GIT_SSH_COMMAND='ssh -i /home/$USER/.ssh/id_rsa_PRIVATE_KEY' git {pull,clone,push}
```
