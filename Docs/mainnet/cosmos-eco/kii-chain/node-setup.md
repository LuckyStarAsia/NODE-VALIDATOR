# Node setup

<figure><img src="../../../.gitbook/assets/image (9).png" alt="" width="200"><figcaption><p>Kiichain</p></figcaption></figure>

## Kiichain - Node setup

### Install tools:

```
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### Install GO:

```
cd $HOME
VER="1.23.8"
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
cd $HOME
rm -rf kiichain
git clone https://github.com/KiiChain/kiichain.git
cd kiichain
git checkout v6.1.0
```

#### Build binaries

```
make install
```

Check version:

```
$HOME/go/bin/kiichaind version --long | tail
```

result:

```
- rsc.io/qr@v0.2.0
- sigs.k8s.io/yaml@v1.6.0
build_tags: netgo,ledger
commit: 79bdc967f558c71f20f44c3c40d6ef942a95867e
cosmos_sdk_version: v0.53.4
go: go version go1.24.5 linux/amd64
name: kiichain
server_name: kiichaind
version: v6.1.0
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
DAEMON_HOME="$HOME/.kiichain/" DAEMON_NAME="kiichaind" cosmovisor init $HOME/go/bin/kiichaind
```

```
mkdir -p $HOME/.kiichain/cosmovisor/upgrades
cp $HOME/go/bin/kiichaind $HOME/.kiichain/cosmovisor/genesis/bin/
```

```
echo "export DAEMON_NAME="kiichaind"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.kiichain/"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### INITIA NODE:

#### Set var:

with port=`19`xxx

```
echo "export WALLET="kii01"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export KII_CHAIN_ID="kiichain_1783-1"" >> $HOME/.bash_profile
echo "export KII_PORT="19"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
kiichaind config set client chain-id $KII_CHAIN_ID && \
kiichaind config set client keyring-backend os && \
kiichaind config set client node tcp://localhost:${KII_PORT}657

```

#### _Initialize the node_

```
kiichaind init "$MONIKER" --chain-id $KII_CHAIN_ID --home $HOME/.kiichain
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${KII_PORT}317%g;
s%:8080%:${KII_PORT}080%g;
s%:9090%:${KII_PORT}090%g;
s%:9091%:${KII_PORT}091%g;
s%:8545%:${KII_PORT}545%g;
s%:8546%:${KII_PORT}546%g" $HOME/.kiichain/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${KII_PORT}658%g;
s%:26657%:${KII_PORT}657%g;
s%:6060%:${KII_PORT}060%g;
s%:26656%:${KII_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${KII_PORT}656\"%;
s%:26660%:${KII_PORT}660%g" $HOME/.kiichain/config/config.toml
```

#### Download genesis & addrbook

```
wget -O $HOME/.kiichain/config/genesis.json https://kiichain-mainnet-services.luckystar.asia/kii/genesis.json
wget -O $HOME/.kiichain/config/addrbook.json https://kiichain-mainnet-services.luckystar.asia/kii/addrbook.json
```

#### Add Peers

```
PERSISTENT_PEERS="" &&\
sed -i -e "/^persistent_peers =/ s^= .*^= \"$PERSISTENT_PEERS\"^" $HOME/.kiichain/config/config.toml
```

#### Set minimum gas price

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "1000000000akii"|g' $HOME/.kiichain/config/app.toml
```

#### Set pruning

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.kiichain/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.kiichain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"30\"/" $HOME/.kiichain/config/app.toml
```

#### Disable indexer

```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.kiichain/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kiichain/config/config.toml
```

#### Create service

```
sudo tee /etc/systemd/system/kiichaind.service > /dev/null << EOF
[Unit]
Description=Cosmovisor and Kiichaind service
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.kiichain
ExecStart=$(which cosmovisor) run start --rpc.laddr tcp://127.0.0.1:${KII_PORT}657 --home $HOME/.kiichain
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.kiichain"
Environment="DAEMON_NAME=kiichaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
EOF
```

#### Configure state-sync

```
TRUST_HEIGHT_DELTA=500
LATEST_HEIGHT=$(curl -s "$PRIMARY_ENDPOINT"/block | jq -r ".result.block.header.height")
if [[ "$LATEST_HEIGHT" -gt "$TRUST_HEIGHT_DELTA" ]]; then
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - $TRUST_HEIGHT_DELTA))
else
SYNC_BLOCK_HEIGHT=$LATEST_HEIGHT
fi
```

#### Get the sync block hash

```
SYNC_BLOCK_HASH=$(curl -s "$PRIMARY_ENDPOINT/block?height=$SYNC_BLOCK_HEIGHT" | jq -r ".result.block_id.hash")
```

#### Enable State Sync in Configuration

Modify the `config.toml` file to enable state sync and set the required parameters.

```
sed -i.bak -e "s|^enable *=.*|enable = true|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^rpc_servers *=.*|rpc_servers = \"$PRIMARY_ENDPOINT,$SECONDARY_ENDPOINT\"|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^trust_height *=.*|trust_height = $SYNC_BLOCK_HEIGHT|" $NODE_HOME/config/config.toml
sed -i.bak -e "s|^trust_hash *=.*|trust_hash = \"$SYNC_BLOCK_HASH\"|" $NODE_HOME/config/config.toml
```

#### Enable and start service

```
sudo systemctl daemon-reload
sudo systemctl enable kiichaind
sudo systemctl restart kiichaind && sudo journalctl -u kiichaind -fo cat
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME
sudo systemctl stop kiichaind
sudo systemctl disable kiichaind
sudo rm /etc/systemd/system/kiichaind.service
sudo systemctl daemon-reload
sudo rm -f $(which kiichaind)
rm -rf $HOME/.kiichain
rm -rf $HOME/kiichain
```
