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

<hr>
Command output:<br/>
------------------------
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

<hr>
Sanitization and Administration:<br/>
---------------------------------
```
# List all keys. Accepted Keys, Denied Keys, Unaccepted Keys, Rejected Keys
----------------------------------------------------------------------------
salt-key --list all --out txt      

# delete key
--------------
salt-key -d 'linuxsaltminion0'      

# Delete dead keys for dead minions
------------------------------------
salt --out txt '*' test.ping | grep "Not connected" | cut -d ":" -f 1 | xargs -I dead_minion 
salt-key -y -d dead_minion
```
It does not hurt to automate this via cron
```
# SALT_CRON_IDENTIFIER:Clean old minion keys
*/5 * * * * salt-run manage.down removekeys=True
```



<hr>
SaltStack Grains:<br/>
------------------------------
```
salt '*' grains.ls                 # List grains
salt '*' grains.items              # Get all grains            
salt '*' grains.item os            # Get os information
salt '*' grains.get 'cpu_model'    # Get CPU Model
```

<hr>
Systems Documentation:<br/>
------------------------------
```
salt '*' sys.doc         # output sys.doc (all documentation)
salt '*' sys.doc pkg     # only sys.doc for pkg module
salt '*' sys.doc network # only sys.doc for network module
salt '*' sys.doc system  # only sys.doc for system module
salt '*' sys.doc status  # only sys.doc for status module
```

<hr>
SaltStack Minion Checks:<br/>
--------------------------------
```
salt-run manage.status  # What is the status of all my minions? (both up and down)
salt-run manage.up      # Any minions that are up?
salt-run manage.down    # Any minions that are down?
salt-run manage.alived  # Show all alive minions
salt '*' test.version   # Display salt version
salt '*' test.ping      # Use test module to check if minion is up and responding.
```

<hr>
Saltstack Networking:<br/>
--------------------------------
```
salt 'linuxsaltminion0' network.ip_addrs          # Get IP of your minion
salt 'linuxsaltminion0' network.ping <hostname>   # Ping a host from your minion
salt 'linuxsaltminion0' network.traceroute <hostname>   # Traceroute a host from your minion
salt 'linuxsaltminion0' network.get_hostname      # Get hostname
salt 'linuxsaltminion0' network.mod_hostname      # Modify hostname
```

<hr>
SaltStack Package Management:<br/>
--------------------------------
***note:***  *This primarily affects linux systems*
```
salt '*' pkg.list_upgrades             # get a list of packages that need to be upgrade
salt '*' pkg.upgrade                   # Upgrades all packages via apt-get dist-upgrade (or similar)
salt '*' pkg.version bash              # get current version of the bash package
salt '*' pkg.install bash              # install or upgrade bash package
salt '*' pkg.install bash refresh=True # install or upgrade bash package but
```



***Reference Links***
* <a href="https://github.com/harkx/saltstack-cheatsheet" target="_blank">https://github.com/harkx/saltstack-cheatsheet</a> 
* <a href="https://dev-eole.ac-dijon.fr/doc/cheatsheets/saltstack.html" target="_blank">https://dev-eole.ac-dijon.fr/doc/cheatsheets/saltstack.html</a>
* <a href="https://github.com/saltstack/salt/wiki/Cheat-Sheet" target="_blank">https://github.com/saltstack/salt/wiki/Cheat-Sheet</a>
