![image](https://github.com/user-attachments/assets/b4e78e0d-77eb-46fc-aafe-b9f1c14b14e0)

# SEDA NODE SETUP

## Install tools:
```
sudo apt update && sudo apt upgrade -y
sudo apt install make curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

## Install GO:
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

## Download & Build binary
### Clone project repository
```
cd $HOME
rm -rf seda
git clone https://github.com/sedaprotocol/seda-chain seda
cd seda
git fetch --tags
git checkout v0.1.8-hotfix.3
```

### Build binaries
```
make install
```

Check version:
```
$HOME/go/bin/sedad version --long | tail
```

result:
```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo ledger,
commit: 445d780acf5398b573e37a43223001f8744bb37f
cosmos_sdk_version: v0.50.12
go: go version go1.23.5 linux/amd64
name: seda-chain
server_name: sedad
version: v0.1.8-hotfix.3
```

## Install Cosmovisor and create a service

### Download and install Cosmovisor
```
cd $HOME
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```
Or

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

### Init & prepare for Upgrade Cosmovisor folder
```
mkdir -p $HOME/.sedad/cosmovisor/upgrades/v0.1.8/bin
cp $HOME/go/bin/sedad $HOME/.sedad/cosmovisor/genesis/bin/
cp $HOME/go/bin/sedad $HOME/.sedad/cosmovisor/upgrades/v0.1.8/bin
```

```
DAEMON_HOME="$HOME/.sedad/" DAEMON_NAME="sedad" cosmovisor init $HOME/go/bin/sedad
```

### Create service
```
sudo tee /etc/systemd/system/sedad.service > /dev/null << EOF
[Unit]
Description=osmosis node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
WorkingDirectory=$HOME/.sedad
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.sedad"
Environment="DAEMON_NAME=sedad"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_PREUPGRADE_MAX_RETRIES=0"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.sedad/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```
sudo systemctl daemon-reload
sudo systemctl enable sedad
```


## INITIA NODE:
### Set var: 
with port=`13`xxx
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="LuckyStar"" >> $HOME/.bash_profile
echo "export SEDA_CHAIN_ID="seda-1"" >> $HOME/.bash_profile
echo "export SEDA_PORT="13"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### CONFIG & INIT APP
### _Set node configuration_
```
sedad config set client chain-id $SEDA_CHAIN_ID
sedad config set client keyring-backend os
sedad config set client node tcp://localhost:${SEDA_PORT}657
```

### _Initialize the node_
```
sedad init $MONIKER --chain-id $SEDA_CHAIN_ID --home=$HOME/.sedad
```

## Custom Port:
### set custom ports in `app.toml`
```
sed -i.bak -e "s%:1317%:${SEDA_PORT}317%g;
s%:8080%:${SEDA_PORT}080%g;
s%:9090%:${SEDA_PORT}090%g;
s%:9091%:${SEDA_PORT}091%g;
s%:8545%:${SEDA_PORT}545%g;
s%:8546%:${SEDA_PORT}546%g;
s%:6065%:${SEDA_PORT}065%g" $HOME/.sedad/config/app.toml
```

### set custom ports in `config.toml` file
```
sed -i.bak -e "s%:26658%:${SEDA_PORT}658%g;
s%:26657%:${SEDA_PORT}657%g;
s%:6060%:${SEDA_PORT}060%g;
s%:26656%:${SEDA_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SEDA_PORT}656\"%;
s%:26660%:${SEDA_PORT}660%g" $HOME/.sedad/config/config.toml
```


### Download genesis and addrbook
```
curl -Ls https://raw.githubusercontent.com/sedaprotocol/seda-networks/main/mainnet/genesis.json > $HOME/.sedad/config/genesis.json
curl -Ls https://seda-mainnet-services.luckystar.asia/seda/addrbook.json > $HOME/.sedad/config/addrbook.json
```

### Add seeds
```
seeds="31f54fbcf445a9d9286426be59a17a811dd63f84@18.133.231.208:26656,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@seeds.polkachu.com:25856,cec848e7d4c5a7ae305b27cda133d213435c110f@seed-seda.ibs.team:16679,400f3d9e30b69e78a7fb891f60d76fa3c73f0ecc@seda.rpc.kjnodes.com:17359,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:25856,cec848e7d4c5a7ae305b27cda133d213435c110f@seed-seda.ibs.team:16679,ebc272824924ea1a27ea3183dd0b9ba713494f83@seda-mainnet-seed.autostake.com:26866"
sed -i -e "s|^seeds *=.*|seeds = \"$seeds\"|" $HOME/.sedad/config/config.toml
```

### Set minimum gas price
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"10000000000aseda\"|" $HOME/.sedad/config/app.toml
```

### Set pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.sedad/config/app.toml
```

### Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.sedad/config/config.toml
```

### Enable Prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sedad/config/config.toml
```

## Remove node: _Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your_ `priv_validator_key.json!`
```
cd $HOME
sudo systemctl stop sedad
sudo systemctl disable sedad
sudo rm /etc/systemd/system/sedad.service
sudo systemctl daemon-reload
sudo rm -f $(which sedad)
rm -rf $HOME/.sedad
rm -rf $HOME/seda
```
