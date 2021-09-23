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

