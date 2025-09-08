# Node setup

![image](<../../../.gitbook/assets/Bitway -blackbackground@2x-1.png>)

## BITWAY NODE SETUP

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
rm -rf $HOME/bitway
git clone https://github.com/bitwaylabs/bitway.git
cd bitway
git checkout v2.0.0
```

#### Build binaries

```
make install
```

Check version:

```
$HOME/go/bin/bitwayd version --long | tail
```

result:

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo ledger,
commit: ebf5e00c2fcf6f8d2fc1f831979f10cda2113a7a
cosmos_sdk_version: v0.50.14
go: go version go1.23.3 linux/amd64
name: bitway
server_name: bitwayd
version: 2.0.0
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
mkdir -p $HOME/.bitway/cosmovisor/upgrades
cp $HOME/go/bin/bitwayd $HOME/.bitway/cosmovisor/genesis/bin/
```

```
DAEMON_HOME="$HOME/.bitway/" DAEMON_NAME="bitwayd" cosmovisor init $(which bitwayd)
```

#### Create service

```
sudo tee /etc/systemd/system/bitwayd.service > /dev/null << EOF
[Unit]
Description=Cosmovisor daemon
After=network-online.target

[Service]
Environment="DAEMON_NAME=bitwayd"
Environment="DAEMON_HOME=$HOME/.bitway"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_POLL_INTERVAL=300ms"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_PREUPGRADE_MAX_RETRIES=0"
User=$USER
ExecStart=$HOME/go/bin/cosmovisor run start
Restart=on-failure
RestartSec=10
LimitNOFILE=4096

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable bitwayd
```

### INITIA NODE:

#### Set var:

with port=`15`xxx

```
echo "export WALLET="BitwayM"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export BTW_CHAIN_ID="bitway-1"" >> $HOME/.bash_profile
echo "export BTW_PORT="15"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
bitwayd config set client chain-id $BTW_CHAIN_ID && \
bitwayd config set client keyring-backend os && \
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${BTW_PORT}657\"|" $HOME/.bitway/config/client.toml
```

#### _Initialize the node_

```
bitwayd init $MONIKER --chain-id $BTW_CHAIN_ID --home=$HOME/.bitway
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${BTW_PORT}317%g;
s%:8080%:${BTW_PORT}080%g;
s%:9090%:${BTW_PORT}090%g;
s%:9091%:${BTW_PORT}091%g;
s%:8545%:${BTW_PORT}545%g;
s%:8546%:${BTW_PORT}546%g;
s%:6065%:${BTW_PORT}065%g" $HOME/.bitway/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${BTW_PORT}658%g;
s%:26657%:${BTW_PORT}657%g;
s%:6060%:${BTW_PORT}060%g;
s%:26656%:${BTW_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${BTW_PORT}656\"%;
s%:26660%:${BTW_PORT}660%g" $HOME/.bitway/config/config.toml
```

#### Download genesis and addrbook

```
curl -Ls https://bitway-mainnet-services.luckystar.asia/bitway/genesis.json > $HOME/.bitway/config/genesis.json
curl -Ls https://bitway-mainnet-services.luckystar.asia/bitway/addrbook.json > $HOME/.bitway/config/addrbook.json
```

#### Add peers

```
peers="ab68cefc837ed11989dea304f2364e0f84d5124c@209.250.232.135:26656,ac5e9176ea17ec6a0e5e60a91913c5638b1851e2@202.182.119.24:26656"
sed -i -e "s|^persistent_peers *=.*|persistent_peers = \"$peers\"|" $HOME/.bitway/config/config.toml
```

#### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0006ubtw,0.000001sat\"|" $HOME/.bitway/config/app.toml
```

#### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.bitway/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.bitway/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.bitway/config/config.toml
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME
sudo systemctl stop bitwayd
sudo systemctl disable bitwayd
sudo rm /etc/systemd/system/bitwayd.service
sudo systemctl daemon-reload
sudo rm -f $(which bitwayd)
rm -rf $HOME/.bitway
rm -rf $HOME/bitway
```
