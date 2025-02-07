# CHEAT SHEET
## 🔑 Key management
### Add new key
```
sided keys add $WALLET
```

### Recover existing key
```
sided keys add $WALLET --recover
```

### List all keys
```
sided keys list
```

### Delete key
```
sided keys delete $WALLET
```

### Export key to a file
```
sided keys export $WALLET
```

### Import key from a file
```
sided keys import $WALLET wallet.backup
```

### Query wallet balance
```
sided q bank balances $(sided keys show $WALLET -a)
```

## 👷 Validator management
Please make sure you have adjusted moniker, identity, details and website to match your values.

### Create new validator
```
sided tx staking create-validator \
--amount 1000000uside \
--pubkey $(sided tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id sidechain-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--min-self-delegation 1 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.0006uside \
-y
```

### Edit existing validator
```
sided tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id sidechain-1 \
--commission-rate 0.05 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 0.0006uside \
-y
```

### Unjail validator
```
sided tx slashing unjail --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Jail reason
```
sided query slashing signing-info $(sided tendermint show-validator)
```

### List all active validators
```
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

### List all inactive validators
```
sided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

### View validator details
```
sided q staking validator $(sided keys show $WALLET --bech val -a)
```

## 💲 Token management
### Withdraw rewards from all validators
```
sided tx distribution withdraw-all-rewards --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Withdraw commission and rewards from your validator
```
sided tx distribution withdraw-rewards $(sided keys show $WALLET --bech val -a) --commission --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Delegate tokens to yourself
```
sided tx staking delegate $(sided keys show $WALLET --bech val -a) 1000000uside --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Delegate tokens to validator
```
sided tx staking delegate <TO_VALOPER_ADDRESS> 1000000uside --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Redelegate tokens to another validator
```
sided tx staking redelegate $(sided keys show $WALLET --bech val -a) <TO_VALOPER_ADDRESS> 1000000uside --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Unbond tokens from your validator
```
sided tx staking unbond $(sided keys show $WALLET --bech val -a) 1000000uside --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Send tokens to the wallet
```
sided tx bank send $WALLET <TO_WALLET_ADDRESS> 1000000uside --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

## 🗳 Governance
### List all proposals
```
sided query gov proposals
```

### View proposal by id
```
sided query gov proposal 1
```

### Vote ‘Yes’
```
sided tx gov vote 1 yes --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Vote ‘No’
```
sided tx gov vote 1 no --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Vote ‘Abstain’
```
sided tx gov vote 1 abstain --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

### Vote ‘NoWithVeto’
```
sided tx gov vote 1 NoWithVeto --from $WALLET --chain-id sidechain-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.0006uside -y
```

## ⚡️ Utility
### Update ports
```
CUSTOM_PORT=110
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.side/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.side/config/app.toml
```

### Update Indexer
Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.side/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.side/config/config.toml
```

### Update pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.side/config/app.toml
```

## 🚨 Maintenance
### Get validator info
```
sided status 2>&1 | jq .ValidatorInfo
```

### Get sync info
```
sided status 2>&1 | jq .SyncInfo
```

### Get node peer
```
echo $(sided tendermint show-node-id)'@'$(curl -4s ifconfig.me)':'$(cat $HOME/.side/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

### Check if validator key is correct
```
[[ $(sided q staking validator $(sided keys show $WALLET --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(sided status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Get live peers
```
curl -sS http://localhost:17457/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0006uside,0.000001sat\"/" $HOME/.side/config/app.toml
```

### Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.side/config/config.toml
```

### Reset chain data
```
sided tendermint unsafe-reset-all --keep-addr-book --home $HOME/.side --keep-addr-book
```

### Remove node
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv_validator_key.json!

```
cd $HOME
sudo systemctl stop sided.service
sudo systemctl disable sided.service
sudo rm /etc/systemd/system/sided.service
sudo systemctl daemon-reload
rm -f $(which sided)
rm -rf $HOME/.side
rm -rf $HOME/side
```

## ⚙️ Service Management
### Reload service configuration
```
sudo systemctl daemon-reload
```

### Enable service
```
sudo systemctl enable sided.service
```

### Disable service
```
sudo systemctl disable sided.service
```

### Start service
```
sudo systemctl start sided.service
```

### Stop service
```
sudo systemctl stop sided.service
```

### Restart service
```
sudo systemctl restart sided.service
```

### Check service status
```
sudo systemctl status sided.service
```

### Check service logs
```
sudo journalctl -u sidedd.service -f --no-hostname -o cat
```
https://services.kjnodes.com/mainnet/side/useful-commands/
