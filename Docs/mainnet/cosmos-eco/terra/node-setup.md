# Node setup

<img src="../../../.gitbook/assets/luna.png" alt="Luna" width="360">

## TERRA NODE SETUP

### Install tools:

```
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### Install GO:

```
cd $HOME
VER="1.25.8"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
go version
```

### Download & Build binary

#### Clone project repository

```
cd $HOME && \
rm -rf $HOME/core
git clone https://github.com/phoenix-directive/core/ && \
cd $HOME/core && \
git checkout v2.22.0
```

#### Build binaries

```
make install
```

Check version:

```
$HOME/go/bin/terrad version --long | tail
```

result:

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo ledger,
commit: e68dd029990a4cf347157f2426cabba4d9eb5c30
cosmos_sdk_version: v0.50.15-terra.0
go: go version go1.25.6 linux/amd64
name: terra
server_name: terrad
version: v2.22.0
```

### Install Cosmovisor and create a service

#### Download and install Cosmovisor

```
cd $HOME
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

Or

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

#### Init & prepare for Upgrade Cosmovisor folder

```
mkdir -p $HOME/.terra/cosmovisor/upgrades/v2.22/bin
cp $HOME/go/bin/terrad $HOME/.terra/cosmovisor/genesis/bin/
cp $HOME/go/bin/terrad $HOME/.terra/cosmovisor/upgrades/v2.22/bin
```

```
DAEMON_HOME="$HOME/.terra/" DAEMON_NAME="terrad" cosmovisor init $HOME/go/bin/terrad
```

#### Create service

```
sudo tee /etc/systemd/system/terrad.service > /dev/null << EOF
[Unit]
Description=Terra node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME/.terra
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.terra"
Environment="DAEMON_NAME=terrad"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_PREUPGRADE_MAX_RETRIES=0"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.terra/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable sedad
```

### INITIA NODE:

#### Set var:

with port=22xxx

```
echo "export WALLET="Your_wallet_name"" >> $HOME/.bash_profile
echo "export MONIKER="Your-Moniker"" >> $HOME/.bash_profile
echo "export LUNA_CHAIN_ID="phoenix-1"" >> $HOME/.bash_profile
echo "export LUNA_PORT="32"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
terrad config set client chain-id $LUNA_CHAIN_ID && \
terrad config set client keyring-backend os
```

#### _Initialize the node_

```
terrad init "$MONIKER" --chain-id $LUNA_CHAIN_ID --home $HOME/.terra
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${LUNA_PORT}317%g;
s%:8080%:${LUNA_PORT}080%g;
s%:9090%:${LUNA_PORT}090%g;
s%:9091%:${LUNA_PORT}091%g;
s%8545%${LUNA_PORT}545%g;
s%8546%${LUNA_PORT}546%g" $HOME/.terra/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${LUNA_PORT}658%g;
s%:26657%:${LUNA_PORT}657%g;
s%:6060%:${LUNA_PORT}060%g;
s%:26656%:${LUNA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${LUNA_PORT}656\"%;
s%:26660%:${LUNA_PORT}660%g" $HOME/.terra/config/config.toml
```

#### Download genesis and addrbook

```
curl -Ls https://terra-mainnet-services.luckystar.asia/terra/genesis.json > $HOME/.sedad/config/genesis.json
curl -Ls https://terra-mainnet-services.luckystar.asia/terra/addrbook.json > $HOME/.terra/config/addrbook.json
```

#### Add Peers

```
PEERS=44f045e7d4e9204a4e6e524c346f5c28c8f57c81@23.227.220.127:26656,dc865a0d882f30e41e99ef23d9e6164163607523@54.147.79.192:26656,9e2588bb4b8ba2929e33818f6e6c2e2bb03ce08e@65.108.121.190:2020,ac70351693cbd02389c4779d18f8bacc448d6751@37.27.50.161:20056,879ee21d791817f9bdb3c8b48b416971758af8aa@162.55.88.190:26656
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.terra/config/config.toml
```

#### Set minimum gas price

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.015uluna"|g' $HOME/.terra/config/app.toml
```

#### Set pruning

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.terra/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.terra/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.terra/config/app.toml
```

#### Disable indexer

```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.terra/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.terra/config/config.toml
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME && \
sudo systemctl stop terrad && \
sudo systemctl disable terrad && \
sudo rm -rf /etc/systemd/system/terrad.service && \
sudo rm $(which terrad) && \
sudo rm -rf $HOME/.terra && \
sudo rm -rf kiichain
```
