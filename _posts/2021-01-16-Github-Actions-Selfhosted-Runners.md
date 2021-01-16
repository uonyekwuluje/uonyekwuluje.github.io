---
layout: post
title:  "GitHub Actions Self Hosted Runner Setup"
categories: Automation CICD
---

GitHub Actions is an API for cause and effect on GitHub. Orchestrate and automate any workflow, based on any event. 
With GitHub Actions, workflows and steps are just code in a repository, so you can create, share, reuse your software 
development practices.
<br>
A core component of this process is the use of runners. Runners/Agents etc. are essentially the work horses of most
CICD systems and Github Actions is no exception. in this post, we will be focusing on installing and configuring 
self hosted runners.  

### **Hardware Requirements**
For the purpose of this post, setup or provision a VM with the following specs:
* Ubuntu 18.04/20.04 LTS.
* Admin Privilleges on the server
* Network and firewall configuration as needed. 
* GitHub Personal Access Token
* GitHub Organization Name

### **Package Requirements**
Update the server and install base packages
```
sudo apt upgrade
sudo apt install -y libncursesw5 libncursesw5-dev libncurses5 gcc-7 open-vm-tools open-vm-tools-dev \
gcc dkms libaio1 libcanberra-gtk-module build-essential linux-headers-$(uname -r) libssl-dev openssl \
zip zlib1g-dev cmake zlib1g-dev libxi-dev libxtst-dev curl gnupg2 libgtk2.0-dev jq 
```

**Docker Installation**<br>
This is optional but gives you an added dimension for container builds
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose
sudo usermod -aG docker $USER
```

**Docker Machine Installation**<br>
This is optional but gives you an added dimension for extending functionality in other environments like building agents
in AWS, Azure, Goofle, VMware, Virtualbox etc
```
curl -L https://github.com/docker/machine/releases/download/v0.16.2/docker-machine-`uname -s`-`uname -m` > /tmp/docker-machine
chmod +x /tmp/docker-machine
sudo mv /tmp/docker-machine /bin/docker-machine
```

### **Action Runner Setup**
Prepare and install the runners
```
sudo mkdir /opt/actions-runner
sudo chown -R ubuntu:ubuntu /opt/actions-runner
cd /opt/actions-runner

RUNNER_VERSION="2.271.5"
curl -O -sL https://github.com/actions/runner/releases/download/v${RUNNER_VERSION}/actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz 
tar xzf actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz
rm actions-runner-linux-x64-${RUNNER_VERSION}.tar.gz
chown -R ubuntu:ubuntu /opt/actions-runner 
sudo /opt/actions-runner/bin/installdependencies.sh
```

### **Action Runner Registration for Organization**
This section highlights how to register action runners against a given organization
```
export ORGANIZATION=<Organization Name>
export ACCESS_TOKEN=<PAT Personal Access Token>
export BUILD_LABELS=test

REGISTRATION_TOKEN=$(curl -sX POST -H "Authorization: token ${ACCESS_TOKEN}" https://api.github.com/orgs/${ORGANIZATION}/actions/runners/registration-token | jq .token --raw-output)
./config.sh --url https://github.com/${ORGANIZATION} --token ${REGISTRATION_TOKEN} --labels ${BUILD_LABELS} --name "runner" --work "_work"
```

### **Action Runner Registration for Repositories**
This section highlights how to register action runners against a repository
```
export OWNER="<account name>"
export REPOSITORY="<repository>"
export ACCESS_TOKEN=<PAT Personal Access Token>
export BUILD_LABELS=test

REGISTRATION_TOKEN=$(curl -sX POST -H "Authorization: token ${ACCESS_TOKEN}" https://api.github.com/repos/${OWNER}/${REPOSITORY}/actions/runners/registration-token | jq .token --raw-output)

./config.sh --url https://github.com/${OWNER}/${REPOSITORY} --token ${REGISTRATION_TOKEN} --labels ${BUILD_LABELS} --name "runner" --work "_work"
```

### **Registration Output**
if the registration process completes successfully, you should see this
```
--------------------------------------------------------------------------------
|        ____ _ _   _   _       _          _        _   _                      |
|       / ___(_) |_| | | |_   _| |__      / \   ___| |_(_) ___  _ __  ___      |
|      | |  _| | __| |_| | | | | '_ \    / _ \ / __| __| |/ _ \| '_ \/ __|     |
|      | |_| | | |_|  _  | |_| | |_) |  / ___ \ (__| |_| | (_) | | | \__ \     |
|       \____|_|\__|_| |_|\__,_|_.__/  /_/   \_\___|\__|_|\___/|_| |_|___/     |
|                                                                              |
|                       Self-hosted runner registration                        |
|                                                                              |
--------------------------------------------------------------------------------

# Authentication

√ Connected to GitHub
# Runner Registration

√ Runner successfully added
√ Runner connection is good

# Runner settings

√ Settings Saved.
```

### **Action Sunner Service**
If registration is successful, you can configure a service for your action runners. To do so, change directory to your installation path
and run the following:
```
# Install service
sudo ./svc.sh install

# Start Service
sudo ./svc.sh start

# Check Status
sudo ./svc.sh status

# Stop Service
sudo ./svc.sh stop
```

### **Remove Runner**
To remove a given agent, loginto Github and check the commands from the console. It should be of the form:
```
./config.sh remove --token <runner token>
```
