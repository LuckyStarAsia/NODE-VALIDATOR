
# EMPEIRIA INSTALL

## Update & install dependencies:
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq htop tmux chrony liblz4-tool -y
```

## Install Go:
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
go version
```



### Set Vars:
```
MONIKER=LuckyStar
echo "export MONIKER=$MONIKER" >> $HOME/.bash_profile
echo "export EMPE_CHAIN_ID="empe-testnet-2"" >> $HOME/.bash_profile
echo "export EMPE_PORT="15"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```



## 2️⃣ Install node

```
git clone https://github.com/empe-io/empe-chains.git
```


```
cd $HOME
mkdir -p $HOME/.empe-chain/cosmovisor/genesis/bin
mkdir -p $HOME/.empe-chain/cosmovisor/upgrades/v0.2.2/bin
wget https://github.com/empe-io/empe-chain-releases/raw/master/v0.2.2/emped_v0.2.2_linux_amd64.tar.gz
tar -xvf emped_v0.2.2_linux_amd64.tar.gz
rm -rf emped_v0.2.2_linux_amd64.tar.gz
chmod +x emped
```

```
cp $HOME/emped $HOME/.empe-chain/cosmovisor/genesis/bin
mv $HOME/emped $HOME/.empe-chain/cosmovisor/upgrades/v0.2.2/bin
```

```
ln -s $HOME/.empe-chain/cosmovisor/upgrades/v0.2.2 $HOME/.empe-chain/cosmovisor/current -f
ln -s $HOME/.empe-chain/cosmovisor/current/bin/emped $HOME/go/bin/emped -f
```

### Check version:
```
emped version --long | tail
```

result:
```
emperia@Ubuntu-2204-jammy-amd64-base:~$ emped version --long | tail
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 89a4061fa99a171bb2a15a5c23f0c61749511abd
cosmos_sdk_version: v0.47.11
go: go version go1.22.6 linux/amd64
name: empe
server_name: emped
version: 0.2.2
```

### install cosmovisor: (v1.6.0)
```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0
```

**_note: dont use v1.7.0_**

```
DAEMON_HOME="$HOME/.empe-chain/" DAEMON_NAME="emped" cosmovisor init $HOME/go/bin/emped
```

## ➡️ Initialize the node 

### Config and Init App:
```
emped config node tcp://localhost:${EMPE_PORT}657
emped config keyring-backend test
emped config chain-id $EMPE_CHAIN_ID
emped init $MONIKER --chain-id $EMPE_CHAIN_ID
```

### Set Custom Port:

```
sed -i.bak -e "s%:1317%:${EMPE_PORT}317%g;
s%:8080%:${EMPE_PORT}080%g;
s%:9090%:${EMPE_PORT}090%g;
s%:9091%:${EMPE_PORT}091%g;
s%:8545%:${EMPE_PORT}545%g;
s%:8546%:${EMPE_PORT}546%g;
s%:6065%:${EMPE_PORT}065%g" $HOME/.empe-chain/config/app.toml
sed -i.bak -e "s%:26658%:${EMPE_PORT}658%g;
s%:26657%:${EMPE_PORT}657%g;
s%:6060%:${EMPE_PORT}060%g;
s%:26656%:${EMPE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPE_PORT}656\"%;
s%:26660%:${EMPE_PORT}660%g" $HOME/.empe-chain/config/config.toml
```


### Genesis & Addrbook
```
wget -O $HOME/.empe-chain/config/genesis.json "https://raw.githubusercontent.com/empe-io/empe-chains/master/testnet-2/genesis.json"
wget -O $HOME/.empe-chain/config/addrbook.json "https://raw.githubusercontent.com/MictoNode/empe-chain/main/addrbook.json"
```

### Gas Settings
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0001uempe\"/;" ~/.empe-chain/config/app.toml
```


### Pruning
```
sed -i -e "s/^pruning *=.*/pruning = \"custom\"/" $HOME/.empe-chain/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"100\"/" $HOME/.empe-chain/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"10\"/" $HOME/.empe-chain/config/app.toml
sed -i "s/^indexer *=.*/indexer = \"null\"/" $HOME/.empe-chain/config/config.toml
```

### Configure Seeds and Peers:

```
SEEDS=""
PEERS="ce90310b1ff5f8d874b73953723906922c455152@65.109.62.115:20656,edfc10bbf28b5052658b3b8b901d7d0fc25812a0@193.70.45.145:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.empe-chain/config/config.toml
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uempe\"/" $HOME/.empe-chain/config/app.toml
```

### Index Null & prometheus true:

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.empe-chain/config/config.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.empe-chain/config/config.toml
```

```
sudo systemctl daemon-reload
sudo systemctl enable emped
```

## ➡️ Create a service
```
sudo tee /etc/systemd/system/emped.service > /dev/null << EOF
[Unit]
Description=empe-chain node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=${HOME}/.empe-chain"
Environment="DAEMON_NAME=emped"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.emped/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```




## SNAPSHOT
Official:

https://archive-testnet.empe.io

https://archive-testnet.empe.io/empe-chain-1_2024-09-05.tar

Don't forget to look at the latest file name!


## Snapshot every day at 0 UTC

https://empeiria-services.luckystar.asia/empeiria/empe-testnet-2_latest.tar

```
sudo systemctl stop emped
cp $HOME/.empe-chain/data/priv_validator_state.json $HOME/.empe-chain/priv_validator_state.json.backup
```

```
rm -rf $HOME/.empe-chain/data/*
curl https://empeiria-services.luckystar.asia/empeiria/empe-testnet-2_latest.tar | tar -xf - -C $HOME/.empe-chain/data
```

```
mv $HOME/.empe-chain/priv_validator_state.json.backup $HOME/.empe-chain/data/priv_validator_state.json
sudo systemctl start emped
```

```
sudo journalctl -u emped -f
```


