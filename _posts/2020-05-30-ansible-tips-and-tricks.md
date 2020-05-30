---
layout: post
title:  "Ansible Tips & Tricks"
categories: Ansible
---
Configuration management with ansible is nice. Simplifies administrative tasks and ensures you have a system that can be reproduced, 
configured and updated at will. Sometimes you run into use cases that entails going beyond the basics or just employing simple concepts
differently. This post is geared at sharing some of these tips. 

**Appending text blocks to an existing file**<br>
This comes in handy if you want to add more lines to a text file:
```
- name: Update /etc/security/limits.conf 
  lineinfile: 
    dest: "/etc/security/limits.conf"
    line: {{{{ item }}}}
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
     line: name={{{{ ansible_nodename }}}}
  when: test_grep.stdout == "0"
```
