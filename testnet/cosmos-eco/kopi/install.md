# install

## KOPI ![image](https://github.com/user-attachments/assets/b4719195-1938-45e5-b9c4-dac2dad333bf)

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
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

or just this to use go on other user:

```
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```

### Cloning Kopi repository and installing it locally:

#### test5 v0.6

```
rm -rf ${HOME}/kopi
git clone --quiet --depth 1 --branch v0.6 https://github.com/kopi-money/kopi.git ${HOME}/kopi
cd ${HOME}/kopi
make install
```

Check Kopi version

```
$HOME/go/bin/kopid version --long | tail
```

result

```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 5a1ae2172db7684a696b162ed2281ca68da891c2
cosmos_sdk_version: v0.50.7-0.5
go: go version go1.23.2 linux/amd64
name: kopi
server_name: kopid
version: v0.6
```

#### Configure Cosmovisor:

```
cd $HOME
mkdir -p $HOME/.kopid/cosmovisor/genesis/bin
mkdir -p $HOME/.kopid/cosmovisor/upgrades
cp $HOME/go/bin/kopid $HOME/.kopid/cosmovisor/genesis/bin/
```

#### Download and install Cosmovisor

```
cd $HOME
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest
```

### INITIA NODE:

#### Set var:

with port=`11`xxx

```
echo "export WALLET="wallet"" >> $HOME/.bash_profile
echo "export MONIKER="your-moniker"" >> $HOME/.bash_profile
echo "export KOPI_CHAIN_ID="kopi-test-5"" >> $HOME/.bash_profile
echo "export KOPI_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```

```
DAEMON_HOME="$HOME/.kopid/" DAEMON_NAME="kopid" cosmovisor init $HOME/go/bin/kopid
```

#### config and init app

```
kopid init $MONIKER && \
kopid config set client chain-id $KOPI_CHAIN_ID && \
kopid config set client keyring-backend test && \
sed -i -e "s|^node *=.*|node = \"tcp://localhost:${KOPI_PORT}657\"|" $HOME/.kopid/config/client.toml
```

### Custom Port:

#### set custom ports in `app.toml`

```
sed -i.bak -e "s%:1317%:${KOPI_PORT}317%g;
s%:8080%:${KOPI_PORT}080%g;
s%:9090%:${KOPI_PORT}090%g;
s%:9091%:${KOPI_PORT}091%g;
s%:8545%:${KOPI_PORT}545%g;
s%:8546%:${KOPI_PORT}546%g;
s%:6065%:${KOPI_PORT}065%g" $HOME/.kopid/config/app.toml
```

#### set custom ports in `config.toml` file

```
sed -i.bak -e "s%:26658%:${KOPI_PORT}658%g;
s%:26657%:${KOPI_PORT}657%g;
s%:6060%:${KOPI_PORT}060%g;
s%:26656%:${KOPI_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${KOPI_PORT}656\"%;
s%:26660%:${KOPI_PORT}660%g" $HOME/.kopid/config/config.toml
```

### Downloading genesis and adjust config files...:

```
IP=$(curl -s -4 icanhazip.com)
sed -i -e "s/external_address = \"\"/external_address = \"${IP}:26656\"/g" ~/.kopid/config/config.toml
sed -i '/^seeds/ c\seeds = "db38ce21eb11a9d9d45cfac6fa7694e79e7336ca@95.217.154.60:26656"' ~/.kopid/config/config.toml
sed -i -e 's/timeout_propose = "3s"/timeout_propose = "1s"/g' ~/.kopid/config/config.toml
sed -i -e 's/timeout_commit = "5s"/timeout_commit = "1s"/g' ~/.kopid/config/config.toml
sed -i -e 's/minimum-gas-prices = ""/minimum-gas-prices = "0ukopi"/g' ~/.kopid/config/app.toml
```

#### Setting pruning

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.kopid/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.kopid/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.kopid/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.kopid/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.kopid/config/config.toml
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.kopid/config/config.toml
```

#### Downloading chain data:

```
wget -q https://data.kopi.money/genesis-test-5.json -O ~/.kopid/config/genesis.json
```

#### Addrbook:

```
curl -Ls https://files.chaintools.tech/chains/kopi/testnet/addrbook.json > $HOME/.kopid/config/addrbook.json
```

### Create service:

```
sudo tee /etc/systemd/system/kopidtest.service > /dev/null << EOF
[Unit]
Description=Cosmovisor daemon
After=network-online.target

[Service]
Environment="DAEMON_NAME=kopid"
Environment="DAEMON_HOME=$HOME/.kopid"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_POLL_INTERVAL=300ms"
Environment="DAEMON_DATA_BACKUP_DIR=$HOME/.kopid"
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

#### Service:

```
sudo systemctl daemon-reload
sudo systemctl enable kopidtest.service
sudo systemctl start kopidtest
```

Stop service:

```
sudo systemctl stop kopidtest
sudo systemctl disable kopidtest
```

#### CHECK:

Check status:

```
sudo systemctl status kopidtest
```

Check log:

```
sudo journalctl -u kopidtest -f --no-hostname -o cat
```

Check Sync:

```
kopid status 2>&1 | jq .sync_info
```

Check validator info:

```
kopid status 2>&1 | jq .validator_info
```

### Delete node:

_**Make sure you have backup your validator before do this action:**_

```
sudo systemctl stop kopidtest
sudo systemctl disable kopidtest
sudo rm -rf /etc/systemd/system/kopidtest.service
sudo systemctl daemon-reload
sudo rm -f $(which kopid)
sudo rm -rf $HOME/.kopid
sudo rm -rf $HOME/kopi
```

## END
