---
cover: ../../../.gitbook/assets/1500x500.jfif
coverY: 0
---

# Setup

## SUNRISE <img src="../../../.gitbook/assets/Sunrise_400x400.jpg" alt="" data-size="line">

### INSTALL

```
sudo apt update && apt upgrade -y
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

#### Install GO

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
rm -rf $HOME/sunrise
git clone https://github.com/sunriselayer/sunrise $HOME/sunrise
cd $HOME/sunrise
git checkout v1.0.0
make install
```

```
GOROOT=$(go1.21.13 env GOROOT) PATH=$GOROOT/bin:$PATH make install
```

```
sunrised version --long | tail
```

Result

```
- rsc.io/tmplfunc@v0.0.3
- sigs.k8s.io/yaml@v1.4.0
build_tags: ""
commit: e835a25e8ea572682207ea543ee71b9b31093a39
cosmos_sdk_version: v0.53.2
go: go version go1.24.2 linux/amd64
name: sunrise
server_name: sunrised
version: HEAD-e835a25e8ea572682207ea543ee71b9b31093a39
```

#### Set Vars:

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="Your-moniker"" >> $HOME/.bash_profile
echo "export SUN_CHAIN_ID="sunrise-1"" >> $HOME/.bash_profile
echo "export SUN_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

#### Config and Init App:

```
sunrised config set client chain-id $SUN_CHAIN_ID && \
sunrised config set client keyring-backend os && \
sunrised config set client node tcp://localhost:${SUN_PORT}657 && \
sunrised init "$MONIKER" --chain-id $SUN_CHAIN_ID --home $HOME/.sunrise
```

#### Download genesis and addrbook

```
curl -Ls https://sunrise-mainnet-services.luckystar.asia/sunrise/genesis.json > $HOME/.sunrise/config/genesis.json
curl -Ls https://sunrise-mainnet-services.luckystar.asia/sunrise/addrbook.json > $HOME/.sunrise/config/addrbook.json
```

#### Configure Seeds and Peers:

```
curl -sS https://sunrise-mainnet-rpc.luckystar.asia/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Result

```
f2819b5622ed0c003391643460464ff6dff5bda9@136.243.67.47:28356
fc93745fb6d05955f5d3e107d495c69d7d545828@91.99.196.242:26656
bb69adc6246d31899055c2da852ef5c3fd5bbfe3@51.195.60.23:28356
34e39405f02872a4a9403f241066cf0875a66ce2@65.108.7.249:28356
7db7f656d36c420f39a8eab76c50c41cff440fa9@65.109.58.158:28356
41e61c0441f0086caf17186950fc440d45870bf5@198.13.54.102:26656
7e8cf0f634173b2db50bae901f4de8d6af0bac3b@54.179.30.77:26656
2d712853b8aeca55161a71f1f5ca8bb27cc499d2@38.146.3.231:28356
e776df4c573785a3416da430fb9c90be72ea795e@23.129.20.120:28356
49be16d94c586f3ebd15f7cc7174d56765043b11@64.185.226.202:28356
13c1a5edd2e09c8ec3998fdc2c8ede330c6224bf@65.108.204.225:28356
```

```
SEEDS="2a30964e07118a0cb03bb1cae9185d37a967230a@a.validator.sunrise-1.sunriselayer.io:26656"
PEERS="41e61c0441f0086caf17186950fc440d45870bf5@a.consensus.sunrise-1.sunriselayer.io:26656,7e8cf0f634173b2db50bae901f4de8d6af0bac3b@b.consensus.sunrise-1.sunriselayer.io:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.sunrise/config/config.toml
```

#### Config Pruning & gas:

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.sunrise/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.sunrise/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.sunrise/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "19"|' $HOME/.sunrise/config/app.toml
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.0025urise\"|" $HOME/.sunrise/config/app.toml
```

#### Set Custom Port:

```
sed -i.bak -e "s%:1317%:${SUN_PORT}317%g;
s%:8080%:${SUN_PORT}080%g;
s%:9090%:${SUN_PORT}090%g;
s%:9091%:${SUN_PORT}091%g;
s%:8545%:${SUN_PORT}545%g;
s%:8546%:${SUN_PORT}546%g;
s%:6065%:${SUN_PORT}065%g" $HOME/.sunrise/config/app.toml
sed -i.bak -e "s%:26658%:${SUN_PORT}658%g;
s%:26657%:${SUN_PORT}657%g;
s%:6060%:${SUN_PORT}060%g;
s%:26656%:${SUN_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${SUN_PORT}656\"%;
s%:26660%:${SUN_PORT}660%g" $HOME/.sunrise/config/config.toml
```

#### Set Service File:

```
sudo tee /etc/systemd/system/sunrised.service > /dev/null <<EOF
[Unit]
Description=sunrise-mainnet
After=network-online.target

[Service]
User=$USER
ExecStart=$(which sunrised) start --home $HOME/.sunrise
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
sudo systemctl enable sunrised
sudo systemctl start sunrised && sudo journalctl -fu sunrised -o cat
```

```
sudo systemctl stop sunrised
sudo systemctl disable sunrised
```

#### Check Sync:

```
sunrised status 2>&1 | jq .sync_info
```

#### Check validator info

```
sunrised status 2>&1 | jq .validator_info
```

#### Create Validator

```
axoned tx staking create-validator --amount 1000000uatone --pubkey $(axoned comet show-validator) --moniker "Your-moniker" --identity "your-keybase-identity" --details "your-detail" --website "your-website" --chain-id atomone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from $WALLET --gas-adjustment 1.4 --gas auto --gas-prices 0.025uaxone -y
```

### Key management

#### Add new key

```
sunrised keys add $WALLET
```

#### Recover existing key

```
sunrised keys add $WALLET --recover
```

#### List all keys

```
sunrised keys list
```

#### Wallet balance

```
sunrised q bank balances $(sunrised keys show $WALLET -a)
```

### VALIDATOR:

#### Show pubkey:

```
sunrised comet show-validator
```

First, create validator config file&#x20;

```
nano $HOME/.sunrise/config/validator.json
```

```
{
  "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"ZQweivhEkT/akg5RT6RWkElt43rr5cf+qu/QQ5jOpmQ="},
  "amount": "1000000uvrise",
  "moniker": "your_validator's_name",
  "identity": "optional identity signature (ex. UPort or Keybase)",
  "website": "validator's (optional) website",
  "security": "validator's (optional) security contact email",
  "details": "validator's (optional) details",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

#### Create Validator

```
sunrised tx staking create-validator [path/to/validator.json] \
    --chain-id=$SUN_CHAIN_ID \
    --from=$WALLET \
    --keyring-backend=os \
    --gas-prices=0.025uusdrise --gas-adjustment 1.2 \
    --gas=auto \
    -y
```

### Remove node

_**- Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv\_validator\_key.json!**_

```
sudo systemctl stop sunrisedtest.service
sudo systemctl disable sunrisedtest.service
sudo rm -rf /etc/systemd/system/sunrisedtest.service
sudo systemctl daemon-reload
sudo rm -f $(which sunrised)
sudo rm -rf $HOME/.sunrise
sudo rm -rf $HOME/sunrise
```

## END
