
# SIDE ![image](https://github.com/user-attachments/assets/9f5f12f8-5958-4e5e-9a72-764cdfe1c8ac)



## Update & upgrade tools:
```
sudo apt update && sudo apt upgrade -y
sudo apt install make net-tools curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

## Install GO:
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


## Cloning side repository and installing it locally:

### v1.0.0-rc.1 (Mainnet)

```
git clone https://github.com/sideprotocol/side.git
git checkout v1.0.0-rc.1
cd side
make install
```

Check Side version
```
$HOME/go/bin/sided version --long | tail
```

result
```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo ledger,
commit: c013008ff674dcfc82264875b7e36a08db329803
cosmos_sdk_version: v0.50.9-btc1
go: go version go1.22.8 linux/amd64
name: sidechain
server_name: sided
version: 1.0.0-rc.1
```


### Configure Cosmovisor:
```
cd $HOME
mkdir -p $HOME/.side/cosmovisor/genesis/bin
mkdir -p $HOME/.side/cosmovisor/upgrades
cp $HOME/go/bin/sided $HOME/.side/cosmovisor/genesis/bin/
```


### Download and install Cosmovisor
```
cd $HOME
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

## INITIA NODE:
### Set var: 
with port=`12`xxx

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="your-moniker"" >> $HOME/.bash_profile
echo "export SIDE_CHAIN_ID="luwak-1"" >> $HOME/.bash_profile
echo "export SIDE_PORT="12"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
DAEMON_HOME="$HOME/.side/" DAEMON_NAME="sided" cosmovisor init $HOME/go/bin/sided
```

### config and init app
```
sided init $MONIKER && \
sided config set client chain-id $SIDE_CHAIN_ID && \
sided config set client keyring-backend os && \
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${SIDE_PORT}657\"|" $HOME/.side/config/client.toml
```

## Custom Port:
### set custom ports in `app.toml`
```
sed -i.bak -e "s%:1317%:${SIDE_PORT}317%g;
s%:8080%:${SIDE_PORT}080%g;
s%:9090%:${SIDE_PORT}090%g;
s%:9091%:${SIDE_PORT}091%g;
s%:8545%:${SIDE_PORT}545%g;
s%:8546%:${SIDE_PORT}546%g;
s%:6065%:${SIDE_PORT}065%g" $HOME/.side/config/app.toml
```

### set custom ports in `config.toml` file
```
sed -i.bak -e "s%:26658%:${SIDE_PORT}658%g;
s%:26657%:${SIDE_PORT}657%g;
s%:6060%:${SIDE_PORT}060%g;
s%:26656%:${SIDE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SIDE_PORT}656\"%;
s%:26660%:${SIDE_PORT}660%g" $HOME/.side/config/config.toml
```


## Downloading genesis and adjust config files...:

```
IP=$(curl -s -4 icanhazip.com)
sed -i -e "s/external_address = \"\"/external_address = \"${IP}:26656\"/g" ~/.side/config/config.toml
sed -i '/^seeds/ c\seeds = "85919e3dcc7eec3b64bfdd87657c4fac307c9d23@65.109.34.145:26656"' ~/.side/config/config.toml
sed -i -e 's/timeout_propose = "3s"/timeout_propose = "1s"/g' ~/.side/config/config.toml
sed -i -e 's/timeout_commit = "5s"/timeout_commit = "1s"/g' ~/.side/config/config.toml
sed -i -e 's/minimum-gas-prices = ""/minimum-gas-prices = "0uside"/g' ~/.side/config/app.toml
```


### Setting pruning
```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.side/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.side/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.side/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.side/config/app.toml
```

### Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.side/config/config.toml
```

### Enable Prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.side/config/config.toml
```

### Downloading chain data:
```
cd $HOME/.side/config
rm genesis.json 
wget https://github.com/sideprotocol/networks/raw/main/mainnet/sidechain-1/pre-genesis.json -O genesis.json
```


## Create service:
```
sudo tee /etc/systemd/system/sided.service > /dev/null << EOF
[Unit]
Description=Cosmovisor daemon
After=network-online.target

[Service]
Environment="DAEMON_NAME=sided"
Environment="DAEMON_HOME=$HOME/.side"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_POLL_INTERVAL=300ms"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.side"
Environment="UNSAFE_SKIP_BACKUP=false"
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



### Service:
```
sudo systemctl daemon-reload
sudo systemctl enable sided.service
sudo systemctl start sided
```

Stop service:
```
sudo systemctl stop sided
sudo systemctl disable sided
```


### CHECK:
Check status:
```
sudo systemctl status sided
```

Check log:
```
sudo journalctl -u sided -f --no-hostname -o cat
```

Check Sync:
```
sided status 2>&1 | jq .sync_info
```

Check validator info:
```
sided status 2>&1 | jq .validator_info
```


## Delete node:
**_Make sure you have backup your validator before do this action:_**

```
sudo systemctl stop sided
sudo systemctl disable sided
sudo rm -rf /etc/systemd/system/sided.service
sudo systemctl daemon-reload
sudo rm -f $(which sided)
sudo rm -rf $HOME/.side
sudo rm -rf $HOME/side
```

# END
