---
layout: post
title:  "Shell Scripting Tips & Tricks"
categories: Scripting
---

Collection of shell and bash snippets.  

### **Error Handling**
```
# Disable exit on non 0. Continue if script fails
set +e

#Enable exit on non 0. Stop if script fails
set -e
```

### **Trace in Scripts**
```
# Helps debug code as it executes
set -xv
```

### **Packaging & Un-Packaging**
```
# open a .bz2 file
bzip2 -dc my_file.tar.bz2 | tar xvf -
```

### **Untar contents into a folder**
```
mkdir helm
tar -zxvf helm-v3.8.1-linux-amd64.tar.gz -C helm --strip-components=1
```

### **Password Management**
Set initial password
```
echo -e "linuxpassword\nlinuxpassword" | passwd linuxuser
```
Set Password for remote servers
```
for ((i=1;i<=100;i++)); do \
ssh 192.168.1.10$i 'echo -e "linuxpassword\nlinuxpassword" | passwd linuxuser'; \
done;
```
Create user and set password
```
useradd newuser; echo -e "passwdofuser\npasswdofuser" | passwd newuser
```
