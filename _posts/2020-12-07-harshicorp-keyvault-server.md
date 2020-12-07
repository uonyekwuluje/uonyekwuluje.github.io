---
layout: post
title:  "HashiCorp Vault Setup"
categories: Security Automation, Secrets
---

Managing secrets, tokens, credentials, sensitive data etc., is one of the hallmarks of IT operations. They are lots of tools out there but we will focus on HashiCorp Vault. [HashiCorp Vault](https://www.vaultproject.io/) is a secrets management tool that helps to provide secure, automated access to sensitive data. In this post, we will install, configure, and access Vault in a DEV envronment to illustrate Vault’s features and API. We will use the latest version of Vault, which is 1.6.0 at the time of this writing.

### **Server Requirements**
We will be installing Vault in a CentOS 7 Server. Ensure you have the required permissions and you also have your firewalls configured accordingly.

### **Update Server**
Update Vault server and install required packages:
```
sudo yum update -y
sudo yum group install -y 'Development Tools'
sudo yum install -y wget
```

### **Prepare Systems Directories**
Prepare directories and folders for our installation:
```
sudo mkdir -p /opt/vault/{logs,bin,data}
sudo mkdir /etc/vault
```

Create Vault user and set folder ownership:
```
sudo useradd -r -d /home/vault -s /bin/nologin vault
sudo chown -R vault:vault /opt/vault
```

### **Install and Configure Vault**
Download and install Vault:
```
VAULT_VERSION=1.6.0
wget https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip
sudo unzip vault_${VAULT_VERSION}_linux_amd64.zip -d /opt/vault/bin
```
Test Vault with this command ```/opt/vault/bin/vault --version```. You should see this
```
Vault v1.6.0 (7ce0bd9691998e0443bc77e98b1e2a4ab1e965d4)
```

Create new file for configuration settings. ```/etc/vault/config.hcl```. 
```
disable_cache = true
disable_mlock = true
ui = true
listener "tcp" {
   address          = "0.0.0.0:8200"
   tls_disable      = 1
}
storage "file" {
   path  = "/opt/vault/data"
 }
api_addr                = "http://0.0.0.0:8200"
max_lease_ttl           = "10h"
default_lease_ttl       = "10h"
cluster_name            = "devvault"
raw_storage_endpoint    = true
disable_sealwrap        = true
disable_printable_check = true
```

Create service config for vault. ```/etc/systemd/system/vault.service```
```
[Unit]
Description="HashiCorp Vault"
Documentation=https://www.vaultproject.io/docs/
Requires=network-online.target
After=network-online.target
ConditionFileNotEmpty=/etc/vault/config.hcl
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
User=vault
Group=vault
ProtectSystem=full
ProtectHome=read-only
PrivateTmp=yes
PrivateDevices=yes
SecureBits=keep-caps
AmbientCapabilities=CAP_IPC_LOCK
NoNewPrivileges=yes
EnvironmentFile=-/etc/sysconfig/vault
Environment=GOMAXPROCS=2
Restart=on-failure
ExecStart=/opt/vault/bin/vault server -config=/etc/vault/config.hcl
ExecReload=/bin/kill --signal HUP
StandardOutput=/opt/vault/logs/output.log
StandardError=/opt/vault/logs/error.log
LimitMEMLOCK=infinity
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
KillSignal=SIGINT
RestartSec=5
TimeoutStopSec=30
StartLimitBurst=3
LimitNOFILE=65536


[Install]
WantedBy=multi-user.target
```

Enable and start service:
```
sudo systemctl enable vault.service 
sudo systemctl start vault.service 
```
Check Status. ```sudo systemctl status vault.service```. You should see this:
```
● vault.service - "HashiCorp Vault"
   Loaded: loaded (/etc/systemd/system/vault.service; enabled; vendor preset: disabled)
   Active: active (running) since Mon 2020-12-07 07:27:31 EST; 37s ago
     Docs: https://www.vaultproject.io/docs/
 Main PID: 966 (vault)
    Tasks: 7 (limit: 17528)
   Memory: 148.3M
   CGroup: /system.slice/vault.service
           └─966 /opt/vault/bin/vault server -config=/etc/vault/config.hcl

Dec 07 07:27:34 vaultserver vault[966]:               Go Version: go1.15.4
Dec 07 07:27:34 vaultserver vault[966]:               Listener 1: tcp (addr: "0.0.0.0:8200", cluster address: "0.0.0.0:8201", max_request_duration: "1m30s">
Dec 07 07:27:34 vaultserver vault[966]:                Log Level: info
Dec 07 07:27:34 vaultserver vault[966]:                    Mlock: supported: true, enabled: false
Dec 07 07:27:34 vaultserver vault[966]:            Recovery Mode: false
Dec 07 07:27:34 vaultserver vault[966]:                  Storage: file
Dec 07 07:27:34 vaultserver vault[966]:                  Version: Vault v1.6.0
Dec 07 07:27:34 vaultserver vault[966]:              Version Sha: 7ce0bd9691998e0443bc77e98b1e2a4ab1e965d4
Dec 07 07:27:34 vaultserver vault[966]: ==> Vault server started! Log data will stream in below:
Dec 07 07:27:34 vaultserver vault[966]: 2020-12-07T07:27:34.117-0500 [INFO]  proxy environment: http_proxy= https_proxy= no_proxy=
```

Configure environment variables:
```
export PATH=$PATH:/opt/vault/bin
echo "export PATH=$PATH:/opt/vault/bin" >> ~/.bashrc
export VAULT_ADDRESS=http://127.0.0.1:8200
echo "export VAULT_ADDR=http://127.0.0.1:8200" >> ~/.bashrc
source .bashrc
```
**NOTE:** *Change IP address. 127.0.0.1 to the IP Address of your server* 

Initialize and unseal your vault:
```
vault operator init
```
You should have this:
```
Unseal Key 1: oFLW7KMk+AgKw6cxPMC2kYhn/SLvjy/ZrkL/XJsBRMO/
Unseal Key 2: KrqJWxHY7GkAOFqNDDKOWnU4d7m2D4IYYovohz69Go/s
Unseal Key 3: 8bJs1u4b1FKLTSzeZ1SiIG+/ks1pbQk78sgT5K9jF6Gv
Unseal Key 4: BiplBCGhkCiVO8QOkA6o++90Y/mtqWidIQ704jpYRX9E
Unseal Key 5: 3L/UTQVLlFtQQaUBVonv+4tNj2z66OcODwW9PHWlPst0

Initial Root Token: s.p5h5p8dliPmq4VwRlBNzR1Y0

Vault initialized with 5 key shares and a key threshold of 3. Please securely
distribute the key shares printed above. When the Vault is re-sealed,
restarted, or stopped, you must supply at least 3 of these keys to unseal it
before it can start servicing requests.

Vault does not store the generated master key. Without at least 3 key to
reconstruct the master key, Vault will remain permanently sealed!

It is possible to generate new unseal keys, provided you have a quorum of
existing unseal keys shares. See "vault operator rekey" for more information.
```

Unseal Vault
```
vault operator unseal oFLW7KMk+AgKw6cxPMC2kYhn/SLvjy/ZrkL/XJsBRMO/
vault operator unseal KrqJWxHY7GkAOFqNDDKOWnU4d7m2D4IYYovohz69Go/s
vault operator unseal 8bJs1u4b1FKLTSzeZ1SiIG+/ks1pbQk78sgT5K9jF6Gv
```
**NOTE:** *Replace keys with first three of your init values*

<br>
Check Status ```vault status```. You should see this
```
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    5
Threshold       3
Version         1.6.0
Storage Type    file
Cluster Name    devvault
Cluster ID      a7004e68-4285-70b1-a6ab-51d6b67ac8b9
HA Enabled      false
```

Enable Secrets.
```
export VAULT_ADDR='http://127.0.0.1:8200'
export VAULT_TOKEN="s.p5h5p8dliPmq4VwRlBNzR1Y0"
vault auth enable approle
vault secrets enable -version=2 -path=secret kv
```

Log into the Web Interface:
```
http://<server ip>:8200/ui/vault/init
password: <token>
```
