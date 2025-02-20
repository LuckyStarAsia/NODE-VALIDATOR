# NATIVE network
![image](https://github.com/user-attachments/assets/c2554fe3-6cc0-4f28-9593-77e0c5780ae9)


## Testnet details
```
Network Chain ID: native-t1
Day of the week:untiv
Binary: gonatived
Working directory: .gonative
```
RPC: https://t-native.rpc.utsa.tech/

API: https://t-native.api.utsa.tech/

Explorer: https://testnet.native.explorers.guru/

Docs: https://github.com/gonative-cc/network-docs/blob/master/validator.md

### _Native Transforms Bitcoin's Role in DeFi by Introducing Programmability, Interoperability, and Accessibility_

Native, Avail, and Nuffle Join Forces to Power BLISS - https://www.binance.com/en/square/post/14680468817418

Roadmap: 2025 - https://x.com/nativenetwork/status/1877357364820517202?s=46

`Testnet 1` is currently running with validator limits filled.

If you are interested in taking part in `testnet 2`, please create a ticket in discord

## check prevotes/precommits status
```
FOLDER = .gonative
```
### find out RPC port:
```
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

 ### Check `prevotes/precommits`. Useful for updates
```
PORT = < enter your port >
```
```
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array' && \
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].precommits_bit_array'
```

      
 
## UPD 🕊 (Update height: )
```
cd  $HOME /
```
### AFTER STOPPING THE NETWORK ON THE REQUIRED BLOCK!!!

## Preparing the server
### update repositories 
```
apt update && sudo apt upgrade -y
```

### install necessary utilities 
```
apt install curl build-essential git wget jq make gcc tmux htop nvme-cli pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### check hard drives 
```
curl -sL yabs.sh | bash -s — -ig
```

### check internet 
```
curl -sL yabs.sh | bash -s — -fg
```

## File2Ban - more details here

### install and copy the config, which will have a higher priority 
```
apt install fail2ban -y && \
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local && \
nano /etc/fail2ban/jail.local
```
### uncomment and add your IP: ignoreip = 127.0.0.1/8 ::1 <ip> 
```
systemctl restart fail2ban # check the status 
systemctl status fail2ban # check which jails are active (by default only sshd) 
fail2ban-client status # check sshd statistics 
fail2ban-client status sshd # look at the logs tail `/var/log/fail2ban.log`
```

## stop the work and remove from startup #systemctl stop fail2ban && systemctl disable fail2ban

### Installing Go
```
ver="1.23.3" && \
wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz" && \
sudo rm -rf /usr/local/go && \
sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz" && \
rm "go$ver.linux-amd64.tar.gz" && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile && \
source $HOME/.bash_profile && \
go version
```

## New node installation
**IMPORTANT**  - in the commands below, everything in  `<>` is changed to `your value` and `removed yourself` `<>`

### Installing binaries
```
git clone https://github.com/gonative-cc/gonative && cd gonative
git checkout v0.1.1
make build
mv $HOME/gonative/out/gonative $HOME/go/bin/gonatived
```

```
gonatived version --long | grep -e version -e commit
```

```
# version: 0.1.1
# commit: 5bcbb57e4e4689cf515e1c9e6326ae9078f3bdfa
# comet_server_version: v1.0.0-beta.1
# cosmos_sdk_version: v0.52.0-rc.1.0.20250106202622-c6327ea6e403
# runtime_version: v2.0.0-20241219154748-69025c556666
# stf_version: v1.0.0-beta.1
```

### Initialize the node to create the necessary configuration files
```
gonatived init <your-moniker> --chain-id native-t1
```

### Download Genesis

#wget -O $HOME/.gonative/config/genesis.json "https://share102.utsa.tech/native/genesis.json"
```
wget https://github.com/gonative-cc/network-docs/raw/refs/heads/master/genesis/genesis-testnet-1.json.gz
gzip -d genesis-testnet-1.json.gz
mv genesis-testnet-1.json  $HOME/.gonative/config/genesis.json
```

### Checksum
```
sha256sum ~/.gonative/config/genesis.json
```
Result: `6c7c7ee70b89850dfe9b9c64b24debe67e36ddf5457c8555d4b979146b99e1b0`


### Download Addr book
```
wget -O $HOME/.gonative/config/addrbook.json "https://share102.utsa.tech/native/addrbook.json"
```

## Setting up the node configuration
### edit the config so we can no longer use the chain-id flag for each CLI command in client.toml 
```
gonatived config set client chain-id native-t1 # configure keyring-backend in client.toml if necessary 
gonatived config set client keyring-backend os # configure the minimum gas price in app.toml sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \" 0.1untiv \" /;" ~/.gonative/config/app.toml # add seeds/bpeers/peers to config.toml external_address = $( wget -qO- eth0.me ) sed -i.bak -e "s/^external_address *=.*/external_address = \" $external_address :26656 \" /" $HOME /.gonative/config/config.toml peers = "2e2f0def6453e67a5d5872da7f73002caf55a010@195.3.221.110:52656,612e6279e528c3fadfe0bb9916fd5532bc9be2cd@164. 132.247.253:56406,f2e45e48f55f649dee0e8dc4adfa445308ae8018@167.235.247.51:26656,a7577f50cdefd9a7a5e4a673278d 9004df9b4bb4@103.219.169.97:56406,236946946eacbf6ab8a6f15c99dac1c80db6f8a5@65.108.203.61:52656,13b49060d7ae1 375a04a8eaec8602920f47b5a55@37.142.120.99:27656,48bd9b42755de1a3039089c4cb66b9dd01561001@57.129.53.32:40656, 0c2926cdac1e31e39ea137ec3438be6485fd58d7@116.202.85.179:10007,b80d0042f7096759ae6aada870b52edf0dcd74af@65.1 09.58.158:26056,f0e48f295e0d7c7d03943c82ac01bfc54969320b@78.46.185.199:26656,8eb8024d28b070d21844a90c7c2d34e f4b731365@193.34.213.77:15007,b5f52d67223c875947161ea9b3a95dbec30041cb@116.202.42.156:32107,5be5b41a6aef28a7 779002f2af0989c7a7da5cfe@165.154.245.110:26656,b07e0482f43a2add3531e9aa7946609386b2e7e7@65.109.113.219:30656 ,6edabc54e76245af9f8f739303c788bfb23bd09a@95.216.12.106:24096,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@176. 9.82.221:30656,d0e0d80be68cec942ad46b36419f0ba76d35d134@94.130.138.41:26444,be5b6092815df2e0b2c190b3deef8831 159bb9a2@64.225.109.119:26656,49784fe6a1b812fd45f4ac7e5cf953c2a3630cef@136.243.17.170:38656,40cbf43553e7ee35 07814e3110a749a7e638aa83@194.163.169.182:24656,1e9a3f3615ec98ea66c41b7e37a724889067bc05@193.24.209.155:12056 ,7567880ef17ce8488c55c3256c76809b37659cce@161.35.157.54:26656,d856c6c6f195b791c54c18407a8ad4391bd30b99@142.1 32.156.99:24096,2dacf537748388df80a927f6af6c4b976b7274cb@148.251.44.42:26656,2c1e6b6b54daa7646339fa9abede159 519ca7cae@37.252.186.248:26656,fbc51b668eb84ae14d430a3db11a5d90fc30f318@65.108.13.154:52656,48d0fdcc642690ed e0ad774f3ba4dce6e549b4db@142.132.215.124:26656,24d51644ea1a6b91bf745f6767a9079d4b35e3d2@37.27.202.188:26656" sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"
```

## Gas:
```
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.1untiv\"/;" ~/.gonative/config/app.toml
```

### Seed & peeer:
```
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $HOME/.gonative/config/config.toml

peers="2e2f0def6453e67a5d5872da7f73002caf55a010@195.3.221.110:52656,612e6279e528c3fadfe0bb9916fd5532bc9be2cd@164.132.247.253:56406,f2e45e48f55f649dee0e8dc4adfa445308ae8018@167.235.247.51:26656,a7577f50cdefd9a7a5e4a673278d9004df9b4bb4@103.219.169.97:56406,236946946eacbf6ab8a6f15c99dac1c80db6f8a5@65.108.203.61:52656,13b49060d7ae1375a04a8eaec8602920f47b5a55@37.142.120.99:27656,48bd9b42755de1a3039089c4cb66b9dd01561001@57.129.53.32:40656,0c2926cdac1e31e39ea137ec3438be6485fd58d7@116.202.85.179:10007,b80d0042f7096759ae6aada870b52edf0dcd74af@65.109.58.158:26056,f0e48f295e0d7c7d03943c82ac01bfc54969320b@78.46.185.199:26656,8eb8024d28b070d21844a90c7c2d34ef4b731365@193.34.213.77:15007,b5f52d67223c875947161ea9b3a95dbec30041cb@116.202.42.156:32107,5be5b41a6aef28a7779002f2af0989c7a7da5cfe@165.154.245.110:26656,b07e0482f43a2add3531e9aa7946609386b2e7e7@65.109.113.219:30656,6edabc54e76245af9f8f739303c788bfb23bd09a@95.216.12.106:24096,ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@176.9.82.221:30656,d0e0d80be68cec942ad46b36419f0ba76d35d134@94.130.138.41:26444,be5b6092815df2e0b2c190b3deef8831159bb9a2@64.225.109.119:26656,49784fe6a1b812fd45f4ac7e5cf953c2a3630cef@136.243.17.170:38656,40cbf43553e7ee3507814e3110a749a7e638aa83@194.163.169.182:24656,1e9a3f3615ec98ea66c41b7e37a724889067bc05@193.24.209.155:12056,7567880ef17ce8488c55c3256c76809b37659cce@161.35.157.54:26656,d856c6c6f195b791c54c18407a8ad4391bd30b99@142.132.156.99:24096,2dacf537748388df80a927f6af6c4b976b7274cb@148.251.44.42:26656,2c1e6b6b54daa7646339fa9abede159519ca7cae@37.252.186.248:26656,fbc51b668eb84ae14d430a3db11a5d90fc30f318@65.108.13.154:52656,48d0fdcc642690ede0ad774f3ba4dce6e549b4db@142.132.215.124:26656,24d51644ea1a6b91bf745f6767a9079d4b35e3d2@37.27.202.188:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.gonative/config/config.toml
seeds="ade4d8bc8cbe014af6ebdf3cb7b1e9ad36f412c0@testnet-seeds.polkachu.com:30656"
sed -i.bak -e "s/^seeds =.*/seeds = \"$seeds\"/" $HOME/.gonative/config/config.toml
```

### Recommended Database Backend
```
sed -i -e "s/^app-db-backend *=.*/app-db-backend = \"goleveldb\"/;" $HOME/.gonative/config/app.toml
sed -i -e "s/^db_backend *=.*/db_backend = \"pebbledb\"/" $HOME/.gonative/config/config.toml
```
### app.toml / base configuration options
```
app-db-backend = "goleveldb"
```
### `app.toml` / base configuration options
```
app-db-backend = "goleveldb"
```
### `config.toml` / base configuration options
```
db_backend = "pebbledb"
```

### (OPTIONAL) Set up `pruning` in `app.toml`
```
pruning="nothing"
pruning_keep_recent="0"
pruning_interval="0"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.gonative/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.gonative/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.gonative/config/app.toml
```

### (OPTIONAL) Disable `indexing` in `config.toml`
```
indexer="null"
sed -i -e "s/^indexer *=.*/indexer = \"$indexer\"/" $HOME/.gonative/config/config.toml
```

### (OPTIONAL) Turn `snapshots` on/off `app.toml`
```
snapshot_interval=1000
sed -i.bak -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" ~/.gonative/config/app.toml
```

## State sync
### add peer
```
peers="f82c497af58874eac7c862c362df9659d95f5332@5.9.87.231:60756"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.gonative/config/config.toml
```

```
SNAP_RPC=https://t-native.rpc.utsa.tech:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 100)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.gonative/config/config.toml
```
```
systemctl restart gonatived && journalctl -u gonatived -f -o cat
```

## Snapshot
time: every 24 hours | indexer: kv | app.toml: goleveldb | config.toml: pebbledb

🌐 https://share102.utsa.tech/native/
```
cd $HOME
systemctl stop gonatived
```

```
cp $HOME/.gonative/data/priv_validator_state.json $HOME/.gonative/priv_validator_state.json.backup
```

### reset
```
rm -rf $HOME/.gonative/data/{application.db,evidence.db,snapshots,tx_index.db,blockstore.db,state.db,cs.wal}
```

### download snapshot
```
curl -o - -L https://share102.utsa.tech/native/native_testnet.tar.lz4 | lz4 -c -d - | tar -x -C $HOME/.gonative/
```
```
mv $HOME/.gonative/priv_validator_state.json.backup $HOME/.gonative/data/priv_validator_state.json
```

```
systemctl restart gonatived && journalctl -u gonatived -f -o cat
```

### Create a service file
```
tee /etc/systemd/system/gonatived.service > /dev/null <<EOF
[Unit]
Description=gonatived
After=network-online.target

[Service]
User=$USER
ExecStart=$(which gonatived) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
```
systemctl daemon-reload
systemctl enable gonatived
systemctl restart gonatived && journalctl -u gonatived -f -o cat
```

## Create or restore a wallet and save the output

### create wallet 
```
gonatived keys add <name_wallet> --keyring-backend os # restore wallet (after the command insert seed) 
gonatived keys add <name_wallet> --recover --keyring-backend os # connect ledger wallet 
gonatived keys add <name_wallet> --ledger 
```

## Don't forget to save the seed!!!

## Create a validator
### 1. Get your pubkey
```
gonatived comet show-validator
# {"@type":"/cosmos.crypto.ed25519.PubKey","key":"ZXONS7NNjLWH4HePBOoHKDAYeLXQO5iUwpCRQSi1poI="}
```

### 2. Create `validator.json`
```
nano $HOME/.gonative/validator.json
```

### 3. Insert `your config`
```
{
  "pubkey": {"#pubkey"},
  "amount": "1000000untiv",
  "moniker": "your-moniker",
  "identity": "",
  "website": "",
  "security": "",
  "details": "",
  "commission-rate": "0.05",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.2",
  "min-self-delegation": "1"
}
```

### 4. Send the transaction
```
gonatived tx staking create-validator $HOME/.gonative/validator.json \
    --from=<key-name> \
    --chain-id=native-t1 \
    --fees 35000untiv --gas 350000
```

### _Don't forget to save `priv_validator_key.json`_ !!!

You can read more about creating/editing a validator here: https://teletype.in/@lesnik13utsa/RPLJpWXIoDQ


## Useful commands
### Information

check logs 
```
sudo journalctl -u gonatived -f -o cat
```

check status
```
curl localhost:$PORT/status | jq
```

check balance 
```
gonatived q bank balances <address>
```
check pubkey of validator 
```
gonatived tendermint show-validator
```
check validator 
```
gonatived query staking validator <valoper_address>
gonatived query staking validators --limit 1000000 -o json | jq '.validators[] | select(.description.moniker=="<name_moniker>")' | jq
```

Check TX_HASH information 
```
gonatived query tx <TX_HASH>
```

network parameters 
```
gonatived q staking params
gonatived q slashing params
```

check how many blocks were skipped by the validator and from which block the asset
```
gonatived q slashing signing-info $(gonatived tendermint show-validator)
```
check slashing
```
gonatived q slashing signing-info $(gonatived tendermint show-validator)
```

find out the validator creation transaction (replace your valoper_address) 
```
gonatived query txs --events='create_validator.validator=<your_valoper_address>' -o=json | jq .txs[0].txhash -r
```

view the active set
```
gonatived q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```

view inactive set
```
gonatived q staking validators -o json --limit=1000 \
| jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' \
| jq -r '.tokens + " - " + .description.moniker' \
| sort -gr | nl
```
 
## Transactions

### collect commissions + rewards
```
gonatived tx distribution withdraw-rewards <valoper_address> --from <name_wallet> --fees 5000untiv --commission -y
```

### delegate another coin to your stake (this is how 1 coin is sent) 
```
gonatived tx staking delegate <valoper_address> 1000000untiv --from <name_wallet> --fees 5000untiv -y
```

### redelegate to another validator 
```
gonatived tx staking redelegate <src-validator-addr> <dst-validator-addr> 1000000untiv --from <name_wallet> --fees 5000untiv -y
```

### unbond 
```
gonatived tx staking unbond <addr_valoper> 1000000untiv --from <name_wallet> --fees 5000untiv -y
```

### send coins to another address 
```
gonatived tx bank send <name_wallet> <address> 1000000untiv --fees 5000untiv -y
```
### break out of jail 
```
gonatived tx slashing unjail --from <name_wallet> --fees 5000untiv -y
```

## Working with wallets

### list wallets 
```
gonatived keys list
```

### show account key
```
gonatived keys show <name_wallet> --bech acc
```
### show validator key 
```
gonatived keys show <name_wallet> --bech val
```
### show consensus key 
```
gonatived keys show <name_wallet> --bech cons
```
### query account 
```
gonatived q auth account $(gonatived keys show <name_wallet> -a) -o text
```
### delete wallet 
```
gonatived keys delete <name_wallet>
```
  

## Delete node
```
systemctl stop gonatived && \
systemctl disable gonatived && \
rm /etc/systemd/system/gonatived.service && \
systemctl daemon-reload && \
cd $HOME && \
rm -rf .gonative gonative && \
rm -rf $(which gonatived)
```

## GOVERNANCE

### list proposals
```
gonatived q gov proposals
```
### view voting results 
```
gonatived q gov proposals --voter <ADDRESS>
```
### vote for proposal 
```
gonatived tx gov vote 1 yes --from <name_wallet> --fees 5000untiv
```
### deposit proposal 
```
gonatived tx gov deposit 1 1000000untiv --from <name_wallet> --fees 5000untiv
```
### create proposal 
```
gonatived tx gov submit-proposal --title="Randomly reward" --description="Reward 10 testnet participants who completed more than 3 tasks" --type="Text" --deposit="11000000grain" --from=<name_wallet> --fees 5000untiv
```
 

     
## Peers and RPC
```
FOLDER=.gonative
```
### Get peer
```
PORTR=$(grep -A 3 "\[p2p\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+") && \
echo $(gonatived tendermint show-node-id)@$(wget -qO- eth0.me)$PORTR
```
### find RPC port
```
echo -e "\033[0;32m$(grep -A 3 "\[rpc\]" ~/$FOLDER/config/config.toml | egrep -o ":[0-9]+")\033[0m"
```

### Check number of peer
```
PORT=
curl -s http://localhost:$PORT/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr | split(":")[2])"' | wc -l
```

### List moniker conect peer
```
curl -s http://localhost:$PORT/net_info | jq '.result.peers[].node_info.moniker'
```

### Check `prevotes/precommits`. for upgrade
```
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].prevotes_bit_array' && \
curl -s localhost:$PORT/consensus_state | jq '.result.round_state.height_vote_set[0].precommits_bit_array'
```

### check prevote of your validator
```
curl -s localhost:$PORT/consensus_state -s | grep $(curl -s localhost:26657/status | jq -r .result.validator_info.address[:12])
```


        



     


 


   
 


     
