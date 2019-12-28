---
layout: post
title:  "SaltStack CheatSheet"
categories: Configuration Management
---

Helpful commands and tips for using saltstack.

Saltmaster Service Status:
```
systemctl status salt-master
```

Saltminion Service Status:
```
systemctl status salt-minion
```

Command output:
<hr>
Saltstack output comes in handy when you need results in various forms for further automation
```
salt '*' grains.item os --out txt
>
linuxsaltminion0: {u'os': u'RedHat'}
winsaltminion0: {u'os': u'Windows'}

salt '*' grains.item os
>
linuxsaltminion0:
    ----------
    os:
        RedHat
winsaltminion0:
    ----------
    os:
        Windows
```


Systems Documentation:
<hr>
```
salt '*' sys.doc         # output sys.doc (all documentation)
salt '*' sys.doc pkg     # only sys.doc for pkg module
salt '*' sys.doc network # only sys.doc for network module
salt '*' sys.doc system  # only sys.doc for system module
salt '*' sys.doc status  # only sys.doc for status module
```

SaltStack Minion Checks:
<hr>
```
salt-run manage.status  # What is the status of all my minions? (both up and down)
salt-run manage.up      # Any minions that are up?
salt-run manage.down    # Any minions that are down?
salt-run manage.alived  # Show all alive minions
salt '*' test.version   # Display salt version
salt '*' test.ping      # Use test module to check if minion is up and responding.
```
