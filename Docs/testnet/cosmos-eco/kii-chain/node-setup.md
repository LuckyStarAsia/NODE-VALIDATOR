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
cd $HOME
rm -rf kiichain
git clone https://github.com/KiiChain/kiichain.git
cd kiichain
git checkout v2.0.0
```

#### Build binaries

```
make install
```

Check version:

```
$HOME/go/bin/kiichaind version --long
```

result:

```
name: kiichain
server_name: <appd>
version: v2.0.0
commit: 5a0bc4e5c165d314987b7ac8c87ac06eafc8cdf9
build_tags: netgo ledger,
go: go version go1.22.10 linux/amd64
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
DAEMON_HOME="$HOME/.kiichain3/" DAEMON_NAME="kiichaind" cosmovisor init $HOME/go/bin/kiichaind
```

```
mkdir -p $HOME/.kiichain3/cosmovisor/upgrades
cp $HOME/go/bin/kiichaind $HOME/.sedad/cosmovisor/genesis/bin/
```

```
echo "export DAEMON_NAME="kiichaind"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.kiichain3/"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### INITIA NODE:

#### Set var:

with port=`19`xxx

```
echo "export WALLET="kii01"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export KII_CHAIN_ID="kiichain3"" >> $HOME/.bash_profile
echo "export KII_PORT="19"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
kiichaind config chain-id $KII_CHAIN_ID
kiichaind config keyring-backend test
kiichaind config node tcp://localhost:${KII_PORT}657
kiichaind config broadcast-mode block
```

#### _Initialize the node_

```
kiichaind init "$MONIKER" --chain-id $KII_CHAIN_ID --home=$HOME/.kiichain3
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${KII_PORT}317%g;
s%:8080%:${KII_PORT}080%g;
s%:9090%:${KII_PORT}090%g;
s%:9091%:${KII_PORT}091%g;
s%:8545%:${KII_PORT}545%g;
s%:8546%:${KII_PORT}546%g" $HOME/.kiichain3/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${KII_PORT}658%g;
s%:26657%:${KII_PORT}657%g;
s%:6060%:${KII_PORT}060%g;
s%:26656%:${KII_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${KII_PORT}656\"%;
s%:26660%:${KII_PORT}660%g" $HOME/.kiichain3/config/config.toml
```

#### Download genesis

```
wget -O $HOME/.kiichain3/config/genesis.json https://raw.githubusercontent.com/KiiChain/testnets/refs/heads/main/testnet_oro/genesis.json
```

#### Enable DB

```
sed -i.bak -e "s|^occ-enabled *=.*|occ-enabled = true|" $HOME/.kiichain3/config/app.toml
sed -i.bak -e "s|^sc-enable *=.*|sc-enable = true|" $HOME/.kiichain3/config/app.toml
sed -i.bak -e "s|^ss-enable *=.*|ss-enable = true|" $HOME/.kiichain3/config/app.toml
sed -i.bak -e 's/^# concurrency-workers = 20$/concurrency-workers = 500/' $HOME/.kiichain3/config/app.toml
```

#### Set the node as validator

```
sed -i 's/mode = "full"/mode = "validator"/g' $HOME/.kiichain3/config/config.toml
```

#### Add Peers

```
PERSISTENT_PEERS="5b6aa55124c0fd28e47d7da091a69973964a9fe1@uno.sentry.testnet.v3.kiivalidator.com:26656,5e6b283c8879e8d1b0866bda20949f9886aff967@dos.sentry.testnet.v3.kiivalidator.com:26656,41738a0c881866cd0810dbf5918dfd9f4736d39d@167.235.101.159:26656,837fc0b53b4d045b8aaf9e89566af63f3a46b60d@149.50.96.91:37656,83cf42529a500abe37fc6e6b65573cf038ea287d@172.31.1.237:26656,0fe12600961ab22df47e433418403ea5d492dcd7@172.31.10.28:26656"
sed -i -e "/persistent-peers =/ s^= .*^= \"$PERSISTENT_PEERS\"^" $HOME/.kiichain3/config/config.toml
```

#### Set minimum gas price

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.02ukii"|g' $HOME/.kiichain3/config/app.toml
```

#### Set pruning

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.kiichain3/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.kiichain3/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"30\"/" $HOME/.kiichain3/config/app.toml
```

#### Disable indexer

```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.kiichain3/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kiichain3/config/config.toml
```

#### Create service

```
sudo tee /etc/systemd/system/kiidtest.service > /dev/null << EOF
[Unit]
Description=Cosmovisor and Kiichaind service
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.kiichain3
ExecStart=$(which cosmovisor) run start --x-crisis-skip-assert-invariants --rpc.laddr tcp://127.0.0.1:${KII_PORT}657 --home $HOME/.kiichain3
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.kiichain3"
Environment="DAEMON_NAME=kiichaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
EOF
```

#### Configure state-sync

```
TRUST_HEIGHT_DELTA=500
LATEST_HEIGHT=$(curl -s https://rpc.uno.sentry.testnet.v3.kiivalidator.com/block | jq -r ".block.header.height")
if [[ "$LATEST_HEIGHT" -gt "$TRUST_HEIGHT_DELTA" ]]; then
SYNC_BLOCK_HEIGHT=$(($LATEST_HEIGHT - $TRUST_HEIGHT_DELTA))
else
SYNC_BLOCK_HEIGHT=$LATEST_HEIGHT
fi
```

#### Get the sync block hash

```
SYNC_BLOCK_HASH=$(curl -s "https://rpc.uno.sentry.testnet.v3.kiivalidator.com/block?height=$SYNC_BLOCK_HEIGHT" | jq -r ".block_id.hash")
```

#### Enable State Sync in Configuration

Modify the `config.toml` file to enable state sync and set the required parameters.

```
sed -i.bak -e "s|^enable *=.*|enable = true|" $HOME/.kiichain3/config/config.toml
sed -i.bak -e "s|^rpc-servers *=.*|rpc-servers = \"https://rpc.uno.sentry.testnet.v3.kiivalidator.com,https://rpc.dos.sentry.testnet.v3.kiivalidator.com\"|" $HOME/.kiichain3/config/config.toml
sed -i.bak -e "s|^db-sync-enable *=.*|db-sync-enable = false|" $HOME/.kiichain3/config/config.toml
sed -i.bak -e "s|^trust-height *=.*|trust-height = $SYNC_BLOCK_HEIGHT|" $HOME/.kiichain3/config/config.toml
sed -i.bak -e "s|^trust-hash *=.*|trust-hash = \"$SYNC_BLOCK_HASH\"|" $HOME/.kiichain3/config/config.toml
```

#### Enable and start service

```
sudo systemctl daemon-reload
sudo systemctl enable kiidtest
sudo systemctl restart kiidtest && sudo journalctl -u kiidtest -fo cat
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME
sudo systemctl stop kiidtest
sudo systemctl disable kiidtest
sudo rm /etc/systemd/system/kiidtest.service
sudo systemctl daemon-reload
sudo rm -f $(which kiichaind)
rm -rf $HOME/.kiichain3
rm -rf $HOME/kiichain
```
