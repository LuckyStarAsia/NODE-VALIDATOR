# Cosmosvisor

## Cosmos setup

### Update & upgrade tools:

```
sudo apt update && sudo apt upgrade -y
sudo apt install make net-tools curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### Install GO:

```
cd $HOME
VER="1.23.2"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

or just this to use go on other user:

```
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

### Cloning Cosmos repository and installing it locally:

#### v23.3.0 (Mainnet)

```
cd $HOME && \
rm -rf $HOME/gaia
git clone https://github.com/cosmos/gaia && \
cd $HOME/gaia && \
git checkout v23.3.0 && \
make install
```

Check Kopi version

```
$HOME/go/bin/gaiad version --long | tail
```

result

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 875b68be0df1e9e8940c568e93e7478edc55a27f
cosmos_sdk_version: v0.50.11-lsm
go: go version go1.23.6 linux/amd64
name: gaia
server_name: gaiad
version: v23.3.0
```

#### Configure Cosmovisor:

```
cd $HOME && \
mkdir -p $HOME/.gaia/cosmovisor/upgrades/v23.3.0/bin && \
cp $HOME/go/bin/gaiad $HOME/.gaia/cosmovisor/upgrades/v23.3.0/bin
```

#### Download and install Cosmovisor

```
cd $HOME && \
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

### INITIA NODE:

#### Set var:

with port=`19`xxx

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="your-moniker"" >> $HOME/.bash_profile
echo "export KOPI_CHAIN_ID="cosmoshub-4"" >> $HOME/.bash_profile
echo "export KOPI_PORT="19"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
DAEMON_HOME="$HOME/.gaia/" DAEMON_NAME="gaiad" cosmovisor init $(which gaiad)
```

```
echo "export DAEMON_NAME="gaiad"" >> $HOME/.bash_profile
echo "export DAEMON_HOME="$HOME/.gaia/"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Config and init app

```
gaiad config set client chain-id $GAIA_CHAIN_ID && \
gaiad config set client keyring-backend os && \
gaiad config set client node tcp://localhost:${GAIA_PORT}657 && \
gaiad init "$MONIKER" --chain-id $GAIA_CHAIN_ID
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${GAIA_PORT}317%g;
s%:8080%:${GAIA_PORT}080%g;
s%:9090%:${GAIA_PORT}090%g;
s%:9091%:${GAIA_PORT}091%g;
s%:8545%:${GAIA_PORT}545%g;
s%:8546%:${GAIA_PORT}546%g;
s%:6065%:${GAIA_PORT}065%g" $HOME/.gaia/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${GAIA_PORT}658%g;
s%:26657%:${GAIA_PORT}657%g;
s%:6060%:${GAIA_PORT}060%g;
s%:26656%:${GAIA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${GAIA_PORT}656\"%;
s%:26660%:${GAIA_PORT}660%g" $HOME/.gaia/config/config.toml
```

### Downloading genesis and adjust config files...:

<pre><code><strong>SEEDS=""
</strong>PEERS=""
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.gaia/config/config.toml
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0.0025uatom"|g' $HOME/.gaia/config/app.toml
</code></pre>

#### Setting pruning

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.gaia/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.gaia/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gaia/config/config.toml
```

#### Downloading chain data:

```
wget -O $HOME/.gaia/config/genesis.json https://server-5.itrocket.net/mainnet/cosmoshub/genesis.json
wget -O $HOME/.gaia/config/addrbook.json  https://server-5.itrocket.net/mainnet/cosmoshub/addrbook.json
```

### Create service:

```
sudo tee /etc/systemd/system/gaiad.service > /dev/null << EOF
[Unit]
Description=COSMOS node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.gaia
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.gaia"
Environment="DAEMON_NAME=gaiad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"

[Install]
WantedBy=multi-user.target
EOF
```

#### Service:

```
sudo systemctl daemon-reload
sudo systemctl enable gaiad.service
sudo systemctl start gaiad
```

Stop service:

```
sudo systemctl stop gaiad
sudo systemctl disable gaiad
```

#### CHECK:

Check status:

```
sudo systemctl status gaiad
```

Check log:

```
sudo journalctl -u gaiad -f --no-hostname -o cat
```

Check Sync:

```
gaiad status 2>&1 | jq .sync_info
```

Check validator info:

```
gaiad status 2>&1 | jq .validator_info
```

### Delete node:

_**Make sure you have backup your validator before do this action:**_

```
sudo systemctl stop gaiad
sudo systemctl disable gaiad
sudo rm -rf /etc/systemd/system/gaiad.service
sudo systemctl daemon-reload
sudo rm -f $(which gaiad)
sudo rm -rf $HOME/.gaia
sudo rm -rf $HOME/gaia
```

## END
