![image](https://github.com/user-attachments/assets/6784525c-4f21-4756-a918-fad64a2d353b)

# ATOMONE 
https://cosmos.directory/atomone

https://explorer.allinbits.services/wallet/suggest

https://github.com/atomone-hub/atomone.git

https://github.com/atomone-hub/genesis

https://github.com/atomone-hub/registry

## INSTALL

```
sudo apt update && apt upgrade -y
sudo apt install curl iptables build-essential git wget jq make gcc nano tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```



### Install GO

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




### Install Binary

```
cd $HOME
rm -rf atomone
git clone https://github.com/atomone-hub/atomone.git
cd atomone
git checkout v1.0.0
```

```
GOROOT=$(go1.21.13 env GOROOT) PATH=$GOROOT/bin:$PATH make install
```

```
atomoned version --long | tail
```

Result
```
- pgregory.net/rapid@v1.1.0
- sigs.k8s.io/yaml@v1.4.0
build_tags: netgo,ledger
commit: 2d6996e6f7e87330b40e945978778708bb9651d3
cosmos_sdk_version: v0.47.13
go: go version go1.21.13 linux/amd64
name: atomone
server_name: atomoned
version: v1.0.0
```



### Set Vars:
```
MONIKER=Your-Moniker
echo "export MONIKER=$MONIKER" >> $HOME/.bash_profile
echo "export ATONE_CHAIN_ID="atomone-1"" >> $HOME/.bash_profile
echo "export ATONE_PORT="11"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```


### Config and Init App:
```
atomoned config node tcp://localhost:${ATONE_PORT}657
atomoned config keyring-backend os
atomoned config chain-id $ATONE_CHAIN_ID
atomoned init $MONIKER --chain-id $ATONE_CHAIN_ID
```

### Add Genesis File and Addrbook:

```
wget -O $HOME/.atomone/config/genesis.json https://atomone.fra1.digitaloceanspaces.com/atomone-1/genesis.json
```


### Configure Seeds and Peers:

```
curl -sS https://atomone-rpc.allinbits.services/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

Result
```
2ff3c369e3acdabbc371ee462cdf5c9d45a0c582@178.79.157.65:26656,d4fedcd6918becd7804e7ccaad3d71237edfbb46@144.76.92.22:10656,608b9ddafe16715e05563d8cd40d703b71a6e6a5@162.55.103.20:43656,6a5b68893b69da2b0672ee0f7fec9b76663fb144@82.113.25.144:26656,0b209dd07b07e4754b8763a2cde80eb02a87bee5@65.109.97.51:26656,6d659f88556185baa8b8aab70659ff8489bc71d8@184.107.110.20:17900,9e6916423eaa4302127a0b7cb518ead4f8b98fd8@89.109.112.42:30656,8189bbd3888f1b963a1de6399a16bd1186760912@15.235.115.156:17900,9a2104250620518e94650bda45ca4bf7a40eae14@92.255.196.146:26656,5ad3d484730844e66f15926c4fcc006c77b53ddd@88.99.137.151:26656,2c02f0e92e00a7fdacdfafb1919b3424047b1701@45.87.107.24:27656,0d54eb13f795a07cfacfc5fe963b8dc27b84fc94@135.181.237.86:26656,caa0b10601fc6f2b974c5111c54820ac44b356de@65.109.123.86:26656,1728955056b6aa8ee8d9c4cd41cd1eeeb1474462@136.38.55.33:26656,4a89ad49b77cb751f02825f21b95c77b7bdb8e27@107.155.98.206:60856,a31d85900f6562b3a8b275617359643a5607ed40@146.70.243.163:26656,352a74c5d34bf10d78bc1a4c2d0cde9ad7de30e3@136.52.63.107:26656,35ecbcf9d8377ca2298cbe7a81eb57e520eb2154@152.53.33.96:26656
```


```
SEEDS=""
PEERS="2ff3c369e3acdabbc371ee462cdf5c9d45a0c582@178.79.157.65:26656,d4fedcd6918becd7804e7ccaad3d71237edfbb46@144.76.92.22:10656,608b9ddafe16715e05563d8cd40d703b71a6e6a5@162.55.103.20:43656,6a5b68893b69da2b0672ee0f7fec9b76663fb144@82.113.25.144:26656,0b209dd07b07e4754b8763a2cde80eb02a87bee5@65.109.97.51:26656,6d659f88556185baa8b8aab70659ff8489bc71d8@184.107.110.20:17900,9e6916423eaa4302127a0b7cb518ead4f8b98fd8@89.109.112.42:30656,8189bbd3888f1b963a1de6399a16bd1186760912@15.235.115.156:17900,9a2104250620518e94650bda45ca4bf7a40eae14@92.255.196.146:26656,5ad3d484730844e66f15926c4fcc006c77b53ddd@88.99.137.151:26656,2c02f0e92e00a7fdacdfafb1919b3424047b1701@45.87.107.24:27656,0d54eb13f795a07cfacfc5fe963b8dc27b84fc94@135.181.237.86:26656,caa0b10601fc6f2b974c5111c54820ac44b356de@65.109.123.86:26656,1728955056b6aa8ee8d9c4cd41cd1eeeb1474462@136.38.55.33:26656,4a89ad49b77cb751f02825f21b95c77b7bdb8e27@107.155.98.206:60856,a31d85900f6562b3a8b275617359643a5607ed40@146.70.243.163:26656,352a74c5d34bf10d78bc1a4c2d0cde9ad7de30e3@136.52.63.107:26656,35ecbcf9d8377ca2298cbe7a81eb57e520eb2154@152.53.33.96:26656"
sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.atomone/config/config.toml
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.001uatone\"/" $HOME/.atomone/config/app.toml
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.atomone/config/config.toml
sed -i 's|^prometheus *=.*|prometheus = true|' $HOME/.atomone/config/config.toml
```

### Config Pruning:
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.atomone/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.atomone/config/app.toml
```


### Set Custom Port:

```
sed -i.bak -e "s%:1317%:${ATONE_PORT}317%g;
s%:8080%:${ATONE_PORT}080%g;
s%:9090%:${ATONE_PORT}090%g;
s%:9091%:${ATONE_PORT}091%g;
s%:8545%:${ATONE_PORT}545%g;
s%:8546%:${ATONE_PORT}546%g;
s%:6065%:${ATONE_PORT}065%g" $HOME/.atomone/config/app.toml
sed -i.bak -e "s%:26658%:${ATONE_PORT}658%g;
s%:26657%:${ATONE_PORT}657%g;
s%:6060%:${ATONE_PORT}060%g;
s%:26656%:${ATONE_PORT}656%g;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${ATONE_PORT}656\"%;
s%:26660%:${ATONE_PORT}660%g" $HOME/.atomone/config/config.toml
```

### Set Service File:

```
sudo tee /etc/systemd/system/atomoned.service > /dev/null <<EOF
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

# Enable and Start Service:

```
sudo systemctl daemon-reload
sudo systemctl enable atomoned
sudo systemctl start atomoned && sudo journalctl -fu atomoned -o cat
```

```
sudo systemctl stop atomoned
sudo systemctl disable atomoned
```


### Check Sync:
```
atomoned status 2>&1 | jq .SyncInfo
```

### Check validator info
```
atomoned status 2>&1 | jq .ValidatorInfo
```

### Create Validator
```
atomoned tx staking create-validator --amount 1000000uatone --pubkey $(atomoned comet show-validator) --moniker "Your-moniker" --identity "your-keybase-identity" --details "your-detail" --website "your-website" --chain-id atomone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from atone01 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```


## Key management
### Add new key
```
atomoned keys add wallet
```

### Recover existing key

```
atomoned keys add wallet --recover
```

### List all keys

```
atomoned keys list
```

### Wallet balance
```
atomoned q bank balances $(atomoned keys show wallet -a)
``` 

## VALIDATOR:

### Show pubkey:
```
atomoned comet show-validator
```

### Create Validator
```
atomoned tx staking create-validator --amount 1000000uatone --pubkey $(atomoned comet show-validator) --moniker "Your-moniker" --identity "your-keybase-identity" --details "your-detail" --website "your-website" --chain-id atomone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from atone01 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

## Remove node
_**- Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!**_

```
cd $HOME
sudo systemctl stop atomoned
sudo systemctl disable atomoned
sudo rm /etc/systemd/system/atomoned.service
sudo systemctl daemon-reload
rm -f $(which atomoned)
rm -rf $HOME/.atomone
rm -rf $HOME/atomone
```

# END
