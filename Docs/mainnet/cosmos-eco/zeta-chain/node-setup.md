# Node setup

<figure><img src="../../../.gitbook/assets/image (2).png" alt="" width="100"><figcaption><p>ZETA CHAIN</p></figcaption></figure>

## ZETA NODE SETUP

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
wget -O $HOME/zetacored https://github.com/zeta-chain/node/releases/download/v28.0.0/zetacored-linux-amd64
chmod +x $HOME/zetacored 
mv $HOME/zetacored $HOME/go/bin
```

Check version:

```
$HOME/go/bin/zetacored version --long | tail
```

result:

```
- rsc.io/tmplfunc@v0.0.3
- sigs.k8s.io/yaml@v1.4.0
build_tags: ""
commit: 02f7a33ada5c6917d086447bc36851ffef45a80c
cosmos_sdk_version: v0.50.12
go: go version go1.22.11 linux/amd64
name: zetacore
server_name: <appd>
version: 28.0.0
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
mkdir -p $HOME/.zetacored/cosmovisor/upgrades/v28/bin
cp $HOME/go/bin/sedad $HOME/.sedad/cosmovisor/genesis/bin/
cp $HOME/go/bin/sedad $HOME/.sedad/cosmovisor/upgrades/v28/bin
```

```
DAEMON_HOME="$HOME/.zetacored/" DAEMON_NAME="zetacored" cosmovisor init $HOME/go/bin/zetacored
```

#### Create service

```
sudo tee /etc/systemd/system/zetacored.service > /dev/null << EOF
[Unit]
Description=ZETA node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.zetacored
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.zetacored"
Environment="DAEMON_NAME=zetacored"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable zetacored
```

### INITIA NODE:

#### Set var:

with port=`13`xxx

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export SEDA_CHAIN_ID="zetachain_7000-1"" >> $HOME/.bash_profile
echo "export SEDA_PORT="13"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### CONFIG & INIT APP

#### _Set node configuration_

```
zetacored config set client chain-id ZETA_CHAIN_ID
zetacored config set client keyring-backend os
zetacored config set client node tcp://localhost:${ZETA_PORT}657
```

#### _Initialize the node_

```
zetacored init $MONIKER --chain-id $ZETA_CHAIN_ID --home=$HOME/.zetacored
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${ZETA_PORT}317%g;
s%:8080%:${ZETA_PORT}080%g;
s%:9090%:${ZETA_PORT}090%g;
s%:9091%:${ZETA_PORT}091%g;
s%:8545%:${ZETA_PORT}545%g;
s%:8546%:${ZETA_PORT}546%g;
s%:6065%:${ZETA_PORT}065%g" $HOME/.zetacored/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${ZETA_PORT}658%g;
s%:26657%:${ZETA_PORT}657%g;
s%:6060%:${ZETA_PORT}060%g;
s%:26656%:${ZETA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ZETA_PORT}656\"%;
s%:26660%:${ZETA_PORT}660%g" $HOME/.zetacored/config/config.toml
```

#### Download genesis and addrbook

```
curl -Ls https://zeta-mainnet-services.luckystar.asia/zeta/genesis.json > $HOME/.zeta/config/genesis.json
curl -Ls https://zeta-mainnet-services.luckystar.asia/zeta/addrbook.json > $HOME/.zeta/config/addrbook.json
```

#### Add seeds

```
seeds="4e668be2d80d3475d2350e313bc75b8f0646884f@zetachain-mainnet-seed.itrocket.net:39656"
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" $HOME/.zetacored/config/config.toml
```

#### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0azeta\"|" $HOME/.zetacored/config/app.toml
```

#### Set pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.zetacored/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.zetacored/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zetacored/config/config.toml
```

### Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`

```
cd $HOME
sudo systemctl stop zetacored
sudo systemctl disable zetacored
sudo rm /etc/systemd/system/zetacored.service
sudo systemctl daemon-reload
sudo rm -f $(which zetacored)
rm -rf $HOME/.zetacored
rm -rf $HOME/zetacored
```
