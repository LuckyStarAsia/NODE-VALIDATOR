---
cover: ../../../.gitbook/assets/arkeo-top-bg.jpeg
coverY: 0
---

# Install

## ARKEO Network

![ARKEO Network](../../../.gitbook/assets/ARKEO.jpg)

### Arkeo Node Installation Guide

Chain ID: arkeo-main-v1 | Current Node Version: `v1.0.9`

### Install Go and Cosmovisor

Feel free to skip this step if you already have Go and Cosmovisor.

#### Install Go

We will use Go `v1.22.8` as example here. The code below also cleanly removes any previous Go installation.

```
cd $HOME
VER="1.22.8"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

#### Set vars

Use custom port: 17

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="Your_moniker"" >> $HOME/.bash_profile
echo "export ARKEO_CHAIN_ID="arkeo-main-v1"" >> $HOME/.bash_profile
echo "export ARKEO_PORT="17"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Install Cosmovisor

We will use Cosmovisor `latest` as example here.

```
go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@latest
```

### Install Node

#### Install the current version of node binary.

```
cd $HOME
rm -rf $HOME/arkeo
git clone https://github.com/arkeonetwork/arkeo
cd arkeo
git checkout v1.0.9
make install
```

### Initialize Node

Cosmovisor:

```
DAEMON_HOME="$HOME/.arkeo/" DAEMON_NAME="arkeod" cosmovisor init $HOME/go/bin/arkeod
```

```
mkdir -p $HOME/.arkeo/cosmovisor/upgrades
```

```
echo "export DAEMON_NAME="arkeod"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.arkeo/"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

Node:

```
arkeod config set client chain-id $ARKEO_CHAIN_ID
arkeod config set client keyring-backend os
arkeod config set client node tcp://localhost:${ARKEO_PORT}657
arkeod init "$MONIKER" --chain-id $ARKEO_CHAIN_ID --home $HOME/.arkeo
```

### Configure Node

#### Download Genesis

The genesis file link below is Polkachu's mirror download. The best practice is to find the official genesis download link.

```
wget https://github.com/arkeonetwork/arkeo/raw/refs/heads/master/networks/mainnet/arkeo-main-v1/genesis.mainnet.json.gz
gzip -d genesis.mainnet.json.gz
mv genesis.mainnet.json  $HOME/.arkeo/config/genesis.json
```

Check

```
sha256sum $HOME/.arkeo/config/genesis.json
```

result

`2746207cb2e0e7006ec7ac03f41418d81a3b79108f89578bd550ad1ab3d176ab /home/arkeo/.arkeo/config/genesis.json`

#### Addrbook

```
wget -O $HOME/.arkeo/config/addrbook.json https://arkeo-mainnet-services.luckystar.asia/arkeo/addrbook.json
```

#### Configure Seed, peers

Using a seed node to bootstrap is the best practice in our view. Alternatively, you can use addrbook or persistent\_peers.

```
SEEDS=""
PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.arkeo/config/config.toml
```

#### Custom Port:

Set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${ARKEO_PORT}317%g;
s%:8080%:${ARKEO_PORT}080%g;
s%:9090%:${ARKEO_PORT}090%g;
s%:9091%:${ARKEO_PORT}091%g;
s%:8545%:${ARKEO_PORT}545%g;
s%:8546%:${ARKEO_PORT}546%g;
s%:6065%:${ARKEO_PORT}065%g" $HOME/.arkeo/config/app.toml
```

Set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${ARKEO_PORT}658%g;
s%:26657%:${ARKEO_PORT}657%g;
s%:6060%:${ARKEO_PORT}060%g;
s%:26656%:${ARKEO_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ARKEO_PORT}656\"%;
s%:26660%:${ARKEO_PORT}660%g" $HOME/.arkeo/config/config.toml
```

#### Setting minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.001uarkeo\"|" $HOME/.arkeo/config/app.toml
```

#### Setting pruning

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.arkeo/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.arkeo/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.arkeo/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.arkeo/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.arkeo/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.arkeo/config/config.toml
```

### Launch Node

### Create Service File

Create a `arkeod.service` file in the `/etc/systemd/system` folder with the following code snippet. Make sure to replace USER with your Linux user name. You need sudo privilege to do this step.

```
sudo nano /etc/systemd/system/arkeod.service
```

```
sudo tee /etc/systemd/system/arkeod.service > /dev/null << EOF
[Unit]
Description=Arkeo Node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_NAME=arkeod"
Environment="DAEMON_HOME=$HOME/.arkeo"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

[Install]
WantedBy=multi-user.target
EOF
```

### Download Snapshot

Please use our popular [SNAPSHOT ](snapshot.md)download service to download and extract Arkeo snapshot.

### Start Node Service

#### Enable service

```
sudo systemctl enable arkeod.service
```

#### Start service

```
sudo service arkeod start
```

#### Check logs

```
sudo journalctl -fu arkeod
```
