# install

## AXONE <img src="../../../.gitbook/assets/axone_400x400.jpg" alt="" data-size="line">

### INSTALL

```
sudo apt update && apt upgrade -y
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

#### Install GO

Go Installation (for even user)

```
cd $HOME
VER="1.21.13"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f $HOME/.bash_profile ] && touch $HOME/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile
echo "export GOROOT=$(go1.21.13 env GOROOT) PATH=$GOROOT/bin:$PATH" >> $HOME/.bash_profile
source $HOME/.bash_profile
[ ! -d $HOME/go/bin ] && mkdir -p $HOME/go/bin
```

```
go version --long | tail
```

#### Install Binary

```
cd $HOME
rm -rf axoned
git clone https://github.com/axone-protocol/axoned.git
cd axoned
git checkout v12.0.0
make install
```

```
GOROOT=$(go1.21.13 env GOROOT) PATH=$GOROOT/bin:$PATH make install
```

```
axoned version --long | tail
```

Result

```
- pgregory.net/rapid@v1.2.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 02cf86f0599d6c9b861a74ae14802ec121174088
cosmos_sdk_version: v0.50.13
go: go version go1.24.1 linux/amd64
name: axoned
server_name: axoned
version: 12.0.0
```

#### Set Vars:

```
MONIKER=Your-Moniker
echo "export MONIKER=$MONIKER" >> $HOME/.bash_profile
echo "export AXONE_CHAIN_ID="axone-1"" >> $HOME/.bash_profile
echo "export AXONE_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Config and Init App:

```
axoned config node tcp://localhost:${AXONE_PORT}657
axoned config keyring-backend os
axoned config chain-id $AXONE_CHAIN_ID
axoned init $MONIKER --chain-id $AXONE_CHAIN_ID
```

#### Configure Seeds and Peers:

```
curl -sS https://axone-mainnet-rpc.luckystar.asia/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Result

```
46f9edbbce02f0e6cf8aac82f65fe7aedecf3abd@51.159.96.236:36656
88a89303f7efed5310d2333fc40940aaacac7d3d@217.160.102.31:26656
b356ae3dbfc97a21a89db0d58fdf00c7158d4d85@142.132.131.184:26656
ea1d3b5b70ac85d553a645561fbfd95577afee4c@148.113.153.62:26656
b5b93e898b3df6ac6ee658cf6bc8cb7cac52b3d4@104.234.124.70:26656
```

```
SEEDS=""
PEERS="46f9edbbce02f0e6cf8aac82f65fe7aedecf3abd@51.159.96.236:36656,88a89303f7efed5310d2333fc40940aaacac7d3d@217.160.102.31:26656,b356ae3dbfc97a21a89db0d58fdf00c7158d4d85@142.132.131.184:26656,ea1d3b5b70ac85d553a645561fbfd95577afee4c@148.113.153.62:26656,b5b93e898b3df6ac6ee658cf6bc8cb7cac52b3d4@104.234.124.70:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.axoned/config/config.toml
```

#### Config Pruning:

```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.axoned/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.axoned/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.axoned/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.axoned/config/app.toml
```

#### Set Custom Port:

```
sed -i.bak -e "s%:1317%:${AXONE_PORT}317%g;
s%:8080%:${AXONE_PORT}080%g;
s%:9090%:${AXONE_PORT}090%g;
s%:9091%:${AXONE_PORT}091%g;
s%:8545%:${AXONE_PORT}545%g;
s%:8546%:${AXONE_PORT}546%g;
s%:6065%:${AXONE_PORT}065%g" $HOME/.axoned/config/app.toml
sed -i.bak -e "s%:26658%:${AXONE_PORT}658%g;
s%:26657%:${AXONE_PORT}657%g;
s%:6060%:${AXONE_PORT}060%g;
s%:26656%:${AXONE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${AXONE_PORT}656\"%;
s%:26660%:${AXONE_PORT}660%g" $HOME/.axoned/config/config.toml
```

#### Set Service File:

```
sudo tee /etc/systemd/system/axoned.service > /dev/null <<EOF
[Unit]
Description=atomone-mainnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which atomoned) start --home $HOME/.atomone
Restart=on-failure
RestartSec=10
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```

## Enable and Start Service:

```
sudo systemctl daemon-reload
sudo systemctl enable axoned
sudo systemctl start axoned && sudo journalctl -fu axoned -o cat
```

```
sudo systemctl stop axoned
sudo systemctl disable axoned
```

#### Check Sync:

```
axoned status 2>&1 | jq .SyncInfo
```

#### Check validator info

```
axoned status 2>&1 | jq .ValidatorInfo
```

#### Create Validator

```
axoned tx staking create-validator --amount 1000000uatone --pubkey $(axoned comet show-validator) --moniker "Your-moniker" --identity "your-keybase-identity" --details "your-detail" --website "your-website" --chain-id atomone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from $WALLET --gas-adjustment 1.4 --gas auto --gas-prices 0.025uaxone -y
```

### Key management

#### Add new key

```
axoned keys add $WALLET
```

#### Recover existing key

```
axoned keys add $WALLET --recover
```

#### List all keys

```
axoned keys list
```

#### Wallet balance

```
axoned q bank balances $(axoned keys show $WALLET -a)
```

### VALIDATOR:

#### Show pubkey:

```
axoned comet show-validator
```

#### Create Validator

```
axoned tx staking create-validator --amount 1000000uaxone --pubkey $(axoned comet show-validator) --moniker "Your-moniker" --identity "your-keybase-identity" --details "your-detail" --website "your-website" --chain-id axone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from $WALLET --gas-adjustment 1.4 --gas auto --gas-prices 0.025uaxone -y
```

### Remove node

_**- Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv\_validator\_key.json!**_

```
cd $HOME
sudo systemctl stop axoned
sudo systemctl disable axoned
sudo rm /etc/systemd/system/axoned.service
sudo systemctl daemon-reload
rm -f $(which axoned)
rm -rf $HOME/.axoned
rm -rf $HOME/axone
```

## END
