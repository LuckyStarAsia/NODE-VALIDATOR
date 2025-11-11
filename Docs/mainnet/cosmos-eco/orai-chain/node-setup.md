# Node setup

![image](../../../.gitbook/assets/orai-token.png)

## ORAI NODE SETUP

### Install tools:

```
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### Install GO:

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

### Download & Build binary

#### Clone project repository

```
cd $HOME && \
rm -rf $HOME/wasmd
git clone https://github.com/oraichain/wasmd.git $HOME/wasmd && \
cd $HOME/wasmd && \
git checkout v0.50.12
```

#### Build binaries

<pre><code><strong>make install
</strong></code></pre>

Check version:

```
$HOME/go/bin/oraid version --long | tail
```

result:

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 227473993ac6de3646231c9efb46777ac9d57805
cosmos_sdk_version: v0.50.5-0.20250424103535-b341b75429de
go: go version go1.23.8 linux/amd64
name: orai
server_name: oraid
version: 0.50.12
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
mkdir -p $HOME/.oraid/cosmovisor/upgrades/v0.50.12/bin
cp $HOME/go/bin/oraid $HOME/.oraid/cosmovisor/genesis/bin/
cp $HOME/go/bin/oraid $HOME/.oraid/cosmovisor/upgrades/v0.50.12/bin
```

```
DAEMON_HOME="$HOME/.oraid/" DAEMON_NAME="oraid" cosmovisor init $HOME/go/bin/oraid
```

#### Create service

```
sudo tee /etc/systemd/system/oraid.service > /dev/null << EOF
[Unit]
Description=oraid node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME/.oraid
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.oraid"
Environment="DAEMON_NAME=oraid"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_PREUPGRADE_MAX_RETRIES=0"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.oraid/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable oraid
```

### INITIA NODE:

#### Set var:

with port=`13`xxx

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export ORAI_CHAIN_ID="Oraichain"" >> $HOME/.bash_profile
echo "export ORAI_PORT="13"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
oraid config set client chain-id $ORAI_CHAIN_ID
oraid config set client keyring-backend os
oraid config set client node tcp://localhost:${ORAI_PORT}657
```

#### _Initialize the node_

```
oraid init $MONIKER --chain-id $ORAI_CHAIN_ID --home=$HOME/.oraid
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${ORAI_PORT}317%g;
s%:8080%:${ORAI_PORT}080%g;
s%:9090%:${ORAI_PORT}090%g;
s%:9091%:${ORAI_PORT}091%g;
s%:8545%:${ORAI_PORT}545%g;
s%:8546%:${ORAI_PORT}546%g;
s%:6065%:${ORAI_PORT}065%g" $HOME/.oraid/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${ORAI_PORT}658%g;
s%:26657%:${ORAI_PORT}657%g;
s%:6060%:${ORAI_PORT}060%g;
s%:26656%:${ORAI_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ORAI_PORT}656\"%;
s%:26660%:${ORAI_PORT}660%g" $HOME/.oraid/config/config.toml
```

#### Download genesis and addrbook

```
curl -Ls https://orai-mainnet-services.luckystar.asia/seda/genesis.json > $HOME/.oraid/config/genesis.json
curl -Ls https://orai-mainnet-services.luckystar.asia/seda/addrbook.json > $HOME/.oraid/config/addrbook.json
```

#### Add seeds

```
seeds=""
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" $HOME/.oraid/config/config.toml
```

#### Set minimum gas price

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.0orai"|g' $HOME/.oraid/config/app.toml
```

#### Set pruning

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.oraid/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.oraid/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.oraid/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.oraid/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.oraid/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.oraid/config/config.toml
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
sudo systemctl stop oraid.service
sudo systemctl disable oraid.service
sudo rm -rf /etc/systemd/system/oraid.service
sudo systemctl daemon-reload
sudo rm -f $(which oraid)
sudo rm -rf $HOME/.oraid
sudo rm -rf $HOME/wasmd
```
