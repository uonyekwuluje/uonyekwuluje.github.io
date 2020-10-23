---
layout: post
title:  "Ansible Tips & Tricks"
categories: Ansible
---
Configuration management with ansible is nice. Simplifies administrative tasks and ensures you have a system that can be reproduced, 
configured and updated at will. Sometimes you run into use cases that entails going beyond the basics or just employing simple concepts
from different angles. This post is geared at sharing some of these tips. 


**Gathering Ansible Facts**<br>
This comes in handy when you need host facts for automation. Things like hostname, distribution, network etc
```
ansible <hostname> -m setup
```
You can run filters based on the outputs for example, finding target host distribution
```
ansible -m setup testcluster --limit pubserver
ansible -m setup testcluster -a "filter=ansible_distribution*" --limit pubserver
ansible -m setup testcluster -a "filter=ansible_distribution_major_version" --limit pubserver
```


**Conditional Logic based on system facts**<br>
This comes in handy if you want to make decisions based on system facts. For example distribution of linue, package management
```
---
- name: install Apache webserver
  hosts: all
  tasks:
    - name: Install Apache on Ubuntu Server
      apt: name=apache2 state=present
      become: yes
      when: ansible_os_family == "Debian" and ansible_distribution_version == "18.04"

- name: Check disk space usage
  hosts: all
  tasks:
    
     - name: Check disk space usage on servers
       shell: df -Th
       register: result
      - debug:
          var: result.stdout_lines
        when: ansible_os_family == "Debian" or ansible_os_family == "RedHat"


- name: install Apache Web-Server
  hosts: all
  tasks:
    - name: Install Apache on CentOS  Server
      yum: name=httpd  state=present
      become: yes
      when: ansible_os_family == "RedHat"

    - name: Install Apache on Ubuntu Server
      apt:name=apache2 state=present
      become: yes
      when: ansible_os_family == "Debian"

```


**Appending text blocks to an existing file**<br>
This comes in handy if you want to add more lines to a text file:
```
- name: Update /etc/security/limits.conf 
  lineinfile: 
    dest: "/etc/security/limits.conf"
    line: {{"{{ item "}}}}
  with_items:
    - 'elasticsearch soft nofile 65536'
    - 'elasticsearch hard nofile 65536'
    - 'elasticsearch soft memlock unlimited' 
    - 'elasticsearch hard memlock unlimited'
```

**Updating existing file(s)**<br>
This comes in handy if you want to add a line if it does not exist after a given line
```
- name: Check for occurence of name in /opt/config/user.properties
  shell: grep -c "^name=" /opt/config/user.properties || true
  register: test_grep

- name: Update /opt/config/user.properties configuration file
  lineinfile:
     dest: "/opt/config/user.properties"
     insertafter: "access_id=xxxxxxxxxxxxxxxxxxxxx"
     line: name={{"{{ ansible_nodename" }}}}
  when: test_grep.stdout == "0"
```



**Unarchive into folder**<br>
This comes in handy if you want to unzip, untar a file into a directory
```
- name: Extract archive
  unarchive:
    src: file.tar.gz
    dest: /foo/bar
    extra_opts: [--strip-components=1]
```
