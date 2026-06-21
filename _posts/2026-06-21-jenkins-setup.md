---
layout: post
title:  "Jenkins Setup"
categories: Build Automation
author: Uchechukwu Onyekwuluje
---

Jenkins is one of the most popular CI/CD tool out there. It is used to automate a myriad of tasks ranging from software development tasks, testing code changes, packaging, deploying, infrastructure builds etc. The list goes on.

**Key Features:** Pipeline Automation: Define CI/CD processes as code. Extensibility: Supports over 1,400 plugins for various integrations (e.g., Git, Docker, Kubernetes). Distributed Builds: Run jobs across multiple machines for scalability. Easy Integration: Works with numerous tools and platforms (e.g., Maven, Gradle, GitHub). User-Friendly Interface: Simple web UI for job management and monitoring. Strong Community Support: Regular updates and extensive documentation.

### Ubuntu Setup
The steps below show how to install jenkins in Ubuntu.
```
sudo apt update
sudo apt install -y openjdk-21-jdk openjdk-21-jre wget curl

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update && sudo apt upgrade -y && sudo apt install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
```


### Rocky Linux / Almalinux Setup
The steps below show how to install jenkins in Rocky Linux / AlmaLinux.
```
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo rpm --import https://pkg.jenkins.io/rpm-stable/jenkins.io-2026.key
cd /etc/yum.repos.d/ && sudo curl -O https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo dnf makecache
sudo dnf repolist
sudo dnf install -y jenkins

sudo systemctl start jenkins
sudo systemctl enable jenkins
```

To test jenkins, type `http://<JENKINS-IP>:8080`
