
# XRPL install

![image](https://github.com/user-attachments/assets/8d45382a-6c96-4e7d-9407-5427e99b007e)

## Manual Installation

Official Documentation: 

**Explorer:** https://explorer.luckystar.asia/xrpl-Testnet/staking

Recommended Hardware: `4 Cores`, `32GB RAM`, `500GB` of storage (NVME)

### install dependencies, if needed
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

### install go, if needed
```
cd $HOME
VER="1.21.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### set vars
```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="Your-moniker"" >> $HOME/.bash_profile
echo "export XRPL_CHAIN_ID="exrp_1440002-1"" >> $HOME/.bash_profile
echo "export XRPL_PORT="19"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### download binary
```
cd $HOME
rm -rf xrp
git clone https://github.com/xrplevm/node xrp
cd xrp
git checkout v6.0.0
make install
```

Check version
```
exrpd version --long | tail
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

### config and init app
```
exrpd init $MONIKER --chain-id $XRPL_CHAIN_ID && \
exrpd config set client chain-id $XRPL_CHAIN_ID && \
exrpd config set client keyring-backend test && \
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${XRPL_PORT}657\"|" $HOME/.exrpd/config/client.toml
```

### Install Cosmovisor (`@v1.6.0` or `@latest`)
```
cd $HOME
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

### Prepare cosmovisor directories
```
mkdir -p $HOME/.exrpd/cosmovisor/genesis/bin
ln -s $HOME/.exrpd/cosmovisor/genesis $HOME/.exrpd/cosmovisor/current -f
```

### Copy binary to cosmovisor directory
```
cp $(which exrpd) $HOME/.exrpd/cosmovisor/genesis/bin
```
### init
```
DAEMON_HOME="$HOME/.exrpd/" DAEMON_NAME="exrpd" cosmovisor init $HOME/.exrpd/cosmovisor/genesis/bin/exrpd
```

Check version again:
```
$HOME/.exrpd/cosmovisor/genesis/bin/exrpd version --long | tail
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

### download genesis and addrbook
```
wget -O $HOME/.exrpd/config/genesis.json https://xrpl-testnet-services.luckystar.asia/xrpl/genesis.json
wget -O $HOME/.exrpd/config/addrbook.json  https://xrpl-testnet-services.luckystar.asia/xrpl/addrbook.json
```

### set seeds and peers
```
SEEDS=""
PEERS="fb0ca9d970416bc640e927d43ff5509dba6e5ea6@65.109.113.219:30056,ef2d620a5f14a4616e46ab22f581eb633e71cefa@135.181.178.120:19656,166f7e48e1c756cc663fee5be96ab2bbdb4db970@65.108.193.254:13656,3cd0dcc0ae9bf7fdb1dc1ed69ce869aac9fcbf64@135.181.132.239:26656,a9887791768c3acf96ea93c94f0488ab48f18424@57.129.39.134:26656,47b21299a2c5dac07bb9dd79bcd4d56f0e009493@94.130.164.82:37956,8de8cce1529a1d7bff78c0d6b75f8e5870ef0ec8@128.140.53.164:26656,2b2d3c0147abb30cbbe0def377f552d39ec29eef@87.246.108.90:32130"
sed -i -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*seeds *=.*/seeds = \"$SEEDS\"/}" \
       -e "/^\[p2p\]/,/^\[/{s/^[[:space:]]*persistent_peers *=.*/persistent_peers = \"$PEERS\"/}" $HOME/.exrpd/config/config.toml
```

### set custom ports in `app.toml`
```
sed -i.bak -e "s%:1317%:${XRPL_PORT}317%g;
s%:8080%:${XRPL_PORT}080%g;
s%:9090%:${XRPL_PORT}090%g;
s%:9091%:${XRPL_PORT}091%g;
s%:8545%:${XRPL_PORT}545%g;
s%:8546%:${XRPL_PORT}546%g;
s%:6065%:${XRPL_PORT}065%g" $HOME/.exrpd/config/app.toml
```

### set custom ports in config.toml file
```
sed -i.bak -e "s%:26658%:${XRPL_PORT}658%g;
s%:26657%:${XRPL_PORT}657%g;
s%:6060%:${XRPL_PORT}060%g;
s%:26656%:${XRPL_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${XRPL_PORT}656\"%;
s%:26660%:${XRPL_PORT}660%g" $HOME/.exrpd/config/config.toml
```

### config pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.exrpd/config/app.toml 
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.exrpd/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"19\"/" $HOME/.exrpd/config/app.toml
```

### set minimum gas price, enable prometheus and disable indexing
```
sed -i 's|minimum-gas-prices =.*|minimum-gas-prices = "0axrp"|g' $HOME/.exrpd/config/app.toml
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.exrpd/config/config.toml
sed -i -e "s/^indexer *=.*/indexer = \"null\"/" $HOME/.exrpd/config/config.toml
```

### create service file
Cosmovisor:
```
sudo tee /etc/systemd/system/exrpd.service > /dev/null << EOF
[Unit]
Description=XRPL node
After=network-online.target

[Service]
User=$USER
WorkingDirectory=$HOME/.exrpd
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.exrpd"
Environment="DAEMON_NAME=exrpd"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"

[Install]
WantedBy=multi-user.target
EOF
```

Or:
```
sudo tee /etc/systemd/system/exrpd.service > /dev/null <<EOF
[Unit]
Description=XRPL node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.exrpd
ExecStart=$(which exrpd) start --home $HOME/.exrpd
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```

### enable and start service
```
sudo systemctl daemon-reload
sudo systemctl enable exrpd
sudo systemctl restart exrpd && sudo journalctl -u exrpd -fo cat
```

## Create wallet
### to create a new wallet, use the following command. don’t forget to save the mnemonic
```
exrpd keys add $WALLET
```

### to restore exexuting wallet, use the following command
```
exrpd keys add $WALLET --recover
```

### save wallet and validator address
```
WALLET_ADDRESS=$(exrpd keys show $WALLET -a)
VALOPER_ADDRESS=$(exrpd keys show $WALLET --bech val -a)
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### check sync status, once your node is fully synced, the output from above will print "false"
```
exrpd status 2>&1 | jq .sync_info
```

### before creating a validator, you need to fund your wallet and check balance
```
exrpd query bank balances $WALLET_ADDRESS 
```


## Create validator
### Create `validator.json` file
```
cd $HOME
rm $HOME/.exrpd/config/validator.json
nano $HOME/.exrpd/config/validator.json
```
```
{
  "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"Q/IYvSqZQzoXLZQciYIKH1oBAbruMJ8rYjwuSrRn4mg="},
  "amount": "10000000000000000000axrp",
  "moniker": "LuckyStar",
  "identity": "F2BA212A924EA1F3",
  "details": "https://github.com/LuckyStarAsia",
  "website": "https://www.luckystar.asia",
  "security": "aluckystarasia@gmail.com",
  "commission-rate": "0.1",
  "commission-max-rate": "0.20",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

### Create a validator using the JSON configuration
```
exrpd tx staking create-validator $HOME/.exrpd/config/validator.json \
    --from $WALLET \
    --chain-id $XRPL_CHAIN_ID \
	--gas auto --gas-adjustment 1.5 \
  -y
```

## Delete node
```
sudo systemctl stop exrpd
sudo systemctl disable exrpd
sudo rm -rf /etc/systemd/system/exrpd.service
sudo rm $(which exrpd)
sudo rm -rf $HOME/.exrpd
sed -i "/XRPL_/d" $HOME/.bash_profile
```

# END

