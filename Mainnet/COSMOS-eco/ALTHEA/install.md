---
cover: ../../../Docs/.gitbook/assets/AltheaCover.JPG
coverY: 0
---

# Install

## ALTHEA Network

![ALTHEA Network](https://github.com/user-attachments/assets/1e27bf02-5b08-4ee2-9b66-947bd784c24e)

### Althea Node Installation Guide

Chain ID: `althea_258432-1` | Current Node Version: `v1.4.0`

### Install Go and Cosmovisor

Feel free to skip this step if you already have Go and Cosmovisor.

#### Install Go

We will use Go `v1.23.4` as example here. The code below also cleanly removes any previous Go installation.

```
sudo rm -rvf /usr/local/go/
wget https://golang.org/dl/go1.23.4.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.23.4.linux-amd64.tar.gz
rm go1.23.4.linux-amd64.tar.gz
```

#### Configure Go

Unless you want to configure in a non-standard way, then set these in the `~/.profile` file.

```
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin
```

#### Install Cosmovisor

We will use Cosmovisor `v1.6.0` as example here.

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.6.0
```

### Install Node

#### Install the current version of node binary.

```
rm -rf ${HOME}/althea
git clone https://github.com/althea-net/althea-chain althea
cd ${HOME}/althea
git checkout v1.5.1
make install
```

### Configure Node

#### Initialize Node

Please replace `YOUR_MONIKER` with `your own moniker`.

```
althea init YOUR_MONIKER --chain-id althea_258432-1
```

#### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```
wget -O genesis.json https://snapshots.polkachu.com/genesis/althea/genesis.json --inet4-only
mv genesis.json $HOME/.althea/config
```

#### Configure Seed

Using a seed node to bootstrap is the best practice in our view. Alternatively, you can use addrbook or persistent\_peers.

```
sed -i 's/bootstrap-peers = ""/bootstrap-peers = "ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:12456"/' $HOME/.althea/config/config.toml
```

### Launch Node

Configure Cosmovisor Folder

Create Cosmovisor folders and load the node binary.

#### Create Cosmovisor Folders

```
mkdir -p $HOME/.althea/cosmovisor/genesis/bin
mkdir -p $HOME/.althea/cosmovisor/upgrades
```

#### Load Node Binary into Cosmovisor Folder

```
cp $HOME/go/bin/althea $HOME/.althea/cosmovisor/genesis/bin
```

### Create Service File

Create a `althead.service` file in the `/etc/systemd/system` folder with the following code snippet. Make sure to replace USER with your Linux user name. You need sudo privilege to do this step.

```
sudo nano /etc/systemd/system/althead.service
```

```
[Unit]
Description="althea node"
After=network-online.target

[Service]
User=$USER
ExecStart=/home/USER/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=althea"
Environment="DAEMON_HOME=/home/USER/.althea"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

### Download Snapshot

Please use our popular snapshot download service to download and extract Althea snapshot.

### Start Node Service

#### Enable service

```
sudo systemctl enable althead.service
```

#### Start service

```
sudo service althead start
```

#### Check logs

```
sudo journalctl -fu althead
```
