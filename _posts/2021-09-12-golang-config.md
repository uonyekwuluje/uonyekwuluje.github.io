---
layout: post
title:  "Golang Config and Tips"
categories: GoLang 
---

Coding and developing apps in Golang has been very nice and exciting. This is a collection of tips, commands etc. 
A cheetsheet of some sort for useful commands to make your development easy.

#### **Installation**
This is primarily geared at Linux systems. Redhat, Rocky, Debian etc.
```
GOVERSION="1.16.7"
sudo rm -Rf /usr/local/go
sudo wget https://dl.google.com/go/go${GOVERSION}.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go${GOVERSION}.linux-amd64.tar.gz
sudo rm go${GOVERSION}.linux-amd64.tar.gz
/usr/local/go/bin/go version
```

#### **Run & Build**
To run, build compile etc.
```
# Run go code
go run program.go

# Build go code
go build program.go
```
