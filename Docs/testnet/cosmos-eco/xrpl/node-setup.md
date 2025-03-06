# Node setup

## ![](../../../.gitbook/assets/image.png)

## XRPLEVM NODE SETUP

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
rm -rf xrp
git clone https://github.com/xrplevm/node xrp
cd xrp
git checkout v6.0.0
```

#### Build binaries

```
make install
```

Check version:

```
$HOME/go/bin/exrpd version --long | tail
```

result:

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo ledger,
commit: 6688ca628b4787b41c9f8cfe431dd718753f542b
cosmos_sdk_version: v0.50.11-evmos
go: go version go1.22.9 linux/amd64
name: exrp
server_name: exrpd
version: v6.0.0
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
mkdir -p $HOME/.exrpd/cosmovisor/upgrades/
cp $HOME/go/bin/exrpd $HOME/.sedad/cosmovisor/genesis/bin/
```

```
DAEMON_HOME="$HOME/.exrpd/" DAEMON_NAME="exrpd" cosmovisor init $HOME/go/bin/exrpd
```

```
echo "export DAEMON_NAME="exrpd"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.exrpd/"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Create service

```
sudo tee /etc/systemd/system/exrpdtest.service > /dev/null << EOF
[Unit]
Description=XRPL node
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=.exrpd
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=.exrpd"
Environment="DAEMON_NAME=exrpd"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:.exrpd/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable exrpdtest
```

### INITIA NODE:

#### Set var:

with port=`19`xxx

```
echo "export WALLET="xrpl01"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export XRPL_CHAIN_ID="xrplevm_1449000-1"" >> $HOME/.bash_profile
echo "export XRPL_PORT="19"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
exrpd config set client chain-id $XRPL_CHAIN_ID
exrpd config set client keyring-backend test
exrpd config set client node tcp://localhost:${XRPL_PORT}657
```

#### _Initialize the node_

```
exrpd init "$MONIKER" --chain-id $XRPL_CHAIN_ID --home=$HOME/.exrpd
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${XRPL_PORT}317%g;
s%:8080%:${XRPL_PORT}080%g;
s%:9090%:${XRPL_PORT}090%g;
s%:9091%:${XRPL_PORT}091%g;
s%:8545%:${XRPL_PORT}545%g;
s%:8546%:${XRPL_PORT}546%g;
s%:6065%:${XRPL_PORT}065%g" $HOME/.exrpd/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${XRPL_PORT}658%g;
s%:26657%:${XRPL_PORT}657%g;
s%:6060%:${XRPL_PORT}060%g;
s%:26656%:${XRPL_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${XRPL_PORT}656\"%;
s%:26660%:${XRPL_PORT}660%g" $HOME/.exrpd/config/config.toml
```

#### Download genesis

```
wget -O ~/.exrpd/config/genesis.json https://raw.githubusercontent.com/xrplevm/networks/refs/heads/main/testnet/genesis.json
```

#### Add seeds

```
PEERS=`curl -sL https://raw.githubusercontent.com/xrplevm/networks/main/testnet/peers.txt | sort -R | head -n 10 | awk '{print $1}' | paste -s -d, -`
sed -i.bak -e "s/^seeds *=.*/seeds = \"$PEERS\"/" ~/.exrpd/config/config.toml
cat ~/.exrpd/config/config.toml | grep seeds
```

#### Set minimum gas price

```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0axrp"|g' $HOME/.exrpd/config/app.toml
```

#### Set pruning

```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.exrpd/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.exrpd/config/app.toml
```

#### Disable indexer

```
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.exrpd/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.exrpd/config/config.toml
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME
sudo systemctl stop exrpdtest
sudo systemctl disable exrpdtest
sudo rm /etc/systemd/system/exrpdtest.service
sudo systemctl daemon-reload
sudo rm -f $(which exrpd)
rm -rf $HOME/.exrpd
rm -rf $HOME/xrp
```
