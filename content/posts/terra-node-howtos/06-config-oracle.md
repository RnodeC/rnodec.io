---
title: "Terraform-Validator06: Oracle Server"
description: "Configure and run the oracle daemons"
date: 2021-04-11
draft: false
tags:
  - terra
---

***SPECIAL NOTICE:  TERRA HAS FALLEN***

## Create oracle wallet
Do this from a local client, not this oracle machine.  You want to protect these keys like you would any other.  
```bash
terracli keys add oracle #save this mnemonic! 
terracli keys show oracle
```

Now over on the machine you are using as your oracle you can start setting things up...  

## Create oracleuser
```bash
sudo useradd oracleuser 
```

## Update OS and add a few packages
```bash
sudo yum update -y
sudo yum install jq git wget gcc-c++ make gcc -y
```

#### ...a node package later on requires a newer version of git, for some reason this is best way to get it
```bash
sudo yum install http://opensource.wandisco.com/rhel/7/git/x86_64/wandisco-git-release-7-2.noarch.rpm -y
sudo rpm --import  http://opensource.wandisco.com/RPM-GPG-KEY-WANdisco
sudo yum upgrade git -y
sudo rm -f /etc/yum.repos.d/wandisco-git.repo #get rid of this insecure repo now that we have what we need from it
```

## Adjust nofiles kernel param
This is probably not necessary, but with only one application here, who cares.
```bash
sudo bash -c "cat > /etc/security/limits.d/terrad.conf << EOF
# /etc/security/limits.conf

*                soft    nofile          65535
*                hard    nofile          65535
EOF"
```


## Install nodejs
```bash
VERSION=v14.16.0
DISTRO=linux-x64
sudo mkdir -p /usr/local/lib/nodejs
wget -O - https://nodejs.org/dist/v14.16.0/node-$VERSION-$DISTRO.tar.xz | sudo tar xJ -C /usr/local/lib/nodejs
export PATH=/usr/local/lib/nodejs/node-$VERSION-$DISTRO/bin:$PATH
echo "export PATH=/usr/local/lib/nodejs/node-$VERSION-$DISTRO/bin:$PATH" >> /home/oracleuser/.bashrc
```



## Clone terra oracle-feeder project
```bash
git clone https://github.com/terra-project/oracle-feeder
```

## Configure and prep feeder
```bash
pushd oracle-feeder/feeder
npm install
npm start update-key
# follow prompts to enter your local encryption key and bip39 mnemonic associated with your oracle wallet
```
```bash
popd
```

## Configure and prep price-server
```bash
pushd oracle-feeder/price-server
gcloud secrets versions access 2 --secret=$SECRET_NAME > config/default.js # this default.js has my api key in it.. if you don't have this, just copy the sample provided, and visit one of the providers mentioned in order to get your own api key
npm install
popd
```
* `SECRET_NAME`: this is the name you gave the secret of the oracle default.js file

## Install price-server systemd unit file
```bash
sudo bash -c "cat > /etc/systemd/system/price-server.service << EOF
[Unit]
Description=Terra Oracle Feeder
After=network.target

[Service]
Type=simple
User=oracleuser
WorkingDirectory=/home/oracleuser/oracle-feeder/price-server
ExecStart=/home/oracleuser/oracle-feeder/price-server/run-price-server.sh
Restart=on-failure
RestartSec=5s
Environment="PATH=/usr/local/lib/nodejs/node-v14.16.0-linux-x64/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin"

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=65535

EOF"
```


## Create price-server run script
```bash
bash -c "cat > /home/oracleuser/oracle-feeder/price-server/run-price-server.sh << EOF
#!/bin/bash

/usr/local/lib/nodejs/node-v14.16.0-linux-x64/bin/npm run start

EOF"
chmod +x /home/oracleuser/oracle-feeder/price-server/run-price-server.sh
```

## Install price-feeder systemd unit file
```bash
sudo bash -c "cat > /etc/systemd/system/price-feeder.service << EOF
[Unit]
Description=Terra Oracle Feeder
After=network.target

[Service]
Type=simple
User=oracleuser
WorkingDirectory=/home/oracleuser/oracle-feeder/feeder
EnvironmentFile=/home/oracleuser/oracle-feeder/feeder/price-feeder.env
ExecStart=/home/oracleuser/oracle-feeder/feeder/run-price-feeder.sh
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target

[Service]
LimitNOFILE=65535

EOF"
```


## Create price-feeder env vars file
```bash
bash -c "cat > /home/oracleuser/oracle-feeder/feeder/price-feeder.env << EOF
#!/bin/bash


VALIDATOR_ADDRESS=$VALIDATOR_ADDRESS  
ORACLE_PASS=$ORACLE_PASS
NETWORK=$NETWORK
PATH=/usr/local/lib/nodejs/node-v14.16.0-linux-x64/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin

EOF"
```
* `VALIDATOR_ADDRESS`: this is the `terravaloperxxx` value associated with the validator wallet you will use.  Should be able to find this on the client with `terracli keys show -a --bech val <validator wallet name>`
* `ORACLE_PASS`: you chose this earlier in your `npm start update-key` command
* `NETWORK`: `tequila-0004`, `columbus-4`, etc

## Create price-feeder run script
```bash
bash -c "cat > /home/oracleuser/oracle-feeder/feeder/run-price-feeder.sh << EOF
#!/bin/bash

npm start vote -- \\
   	--source http://localhost:8532/latest \\
   	--lcd http://validator:1317 \\
   	--chain-id "${NETWORK}" \\
 	--denoms sdr,krw,usd,mnt,eur,cny,jpy,gbp,inr,cad,chf,hkd,aud,sgd \\
	--validator "${VALIDATOR_ADDRESS}" \\
	--password "${ORACLE_PASS}" \\
	--gas-prices 169.77ukrw

EOF"
chmod +x /home/oracleuser/oracle-feeder/feeder/run-price-feeder.sh 
```
* `VALIDATOR_ADDRESS`: this is the `terravaloperxxx` value associated with the validator wallet you will use.  Should be able to find this on the client with `terracli keys show -a --bech val <validator wallet name>`
* `ORACLE_PASS`: you chose this earlier in your `npm start update-key` command
* `NETWORK`: `tequila-0004`, `columbus-4`, etc

## Update price feeder /etc/hosts with validator hostname
```bash
sudo bash -c "cat >> /etc/hosts << EOF
$VALIDATOR_IP validator

EOF"
```
* `VALIDATOR_IP`: ip that this node can reach validator at (should be a private ip)


## Delegate permission
This part cannot be done until after your validator has joined the network (registered as a validator).  Here we are telling the network that we would like (and we authorize) *this* oracle instance to vote on the behalf of our validator.  

This command should be run on your client machine.  

```bash
terracli tx oracle set-feeder $FEEDER_ADDRESS --from=$VALIDATOR --chain-id $NETWORK --fees="30000uluna"
```
* `FEEDER_ADDRESS`: public address associated with the wallet you are using for the feeder.  The same one that you used when you ran 'npm start update-key' earlier.  It asked you for a bip39 mnemonic... 
* `VALIDATOR`: the name of the wallet on your local client that the is being used for your validator
* `NETWORK`: `tequila-0004`, `columbus-4`, etc


## Allocate funds in appropriate demonination, and delegate permission 
This feeder wallet needs Luna to pay the gas for all his transactions.  Lots and lots of transaction (1 tx every 5 blocks, ~30 seconds).  Very miniscule gas fees, but you don't want it to run dry or you will start missing votes!
```bash
terracli tx send $FROM_ADDRESS $TO_ADDRESS $AMOUNT --fees 30000uluna
terracli tx market swap $AMOUNT ukrw --from=$WALLET --fees 30000uluna
```
* `FROM_ADDRESS`: if you are on your client machine where this wallet lives, this can just be the public address of the wallet with the funds
* `TO_ADDRESS`: if you are on your client machine where this wallet lives, this can just be the public address of the wallet for the funds
* `AMOUNT`: `10000000uluna` = 10 Luna.  
* `WALLET`: then name of the wallet on your local client that is being used for your feeder


## Start services

These commands would all be over on the oracle machine


```bash
sudo systemctl daemon-reload
sudo systemctl enable price-server
sudo systemctl enable price-feeder
sudo systemctl start price-server
```
Check on price-server...
```bash
sudo journalctl -u price-server
```
```bash
sudo systemctl start price-feeder
```
And check on feeder...
```bash
sudo journalctl -u price-feeder
```
