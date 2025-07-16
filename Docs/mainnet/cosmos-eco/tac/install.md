---
cover: ../../../.gitbook/assets/681860db654f71faabe0ffc4_tac roadmap.png
coverY: 0
---

# Install

## TAC Chain

![TAC chain](../../../.gitbook/assets/tac.jpg)

### TAC Node Installation Guide

Chain ID: `tacchain_239-1` | Current Node Version: `v1.0.1`

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
cd $HOME && \
git clone https://github.com/TacBuild/tacchain $HOME/tacchain && \
cd $HOME/tacchain && \
git checkout v1.0.1 && \
make install
```

### Configure Node

#### Initialize Node

Please replace `YOUR_MONIKER` with `your own moniker`.

```
tacchaind init YOUR_MONIKER --chain-id tacchain_239-1
```

#### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```
wget -O genesis.json https://tac-mainnet-services.luckystar.asia/tac/genesis.json --inet4-only
mv genesis.json $HOME/.tacchaind/config
```

#### Download addrbook

You can use addrbook or persistent\_peers.

```
curl -Ls https://tac-mainnet-services.luckystar.asia/tac/addrbook.json > $HOME/.tacchaind/config/addrbook.json 
```

### Launch Node

Configure Cosmovisor Folder

Create Cosmovisor folders and load the node binary.

#### Create Cosmovisor Folders

```
mkdir -p $HOME/.tacchaind/cosmovisor/genesis/bin
mkdir -p $HOME/.tacchaind/cosmovisor/upgrades
```

#### Load Node Binary into Cosmovisor Folder

```
cp $HOME/go/bin/tacchaind $HOME/.tacchaind/cosmovisor/genesis/bin
```

### Create Service File

Create a `tacchaind.service` file in the `/etc/systemd/system` folder with the following code snippet. Make sure to replace USER with your Linux user name. You need sudo privilege to do this step.

```
sudo nano /etc/systemd/system/tacchaind.service
```

```
[Unit]
Description="tacchain node"
After=network-online.target

[Service]
User=$USER
ExecStart=/home/USER/go/bin/cosmovisor start
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=tacchaind"
Environment="DAEMON_HOME=/home/USER/.tacchaind"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"

[Install]
WantedBy=multi-user.target
```

### Download Snapshot

Please use our popular snapshot download service to download and extract [snapshot](snapshot.md).

### Start Node Service

#### Enable service

```
sudo systemctl enable tacchaind.service
```

#### Start service

```
sudo service tacchaind start
```

#### Check logs

```
sudo journalctl -fu tacchaind
```
