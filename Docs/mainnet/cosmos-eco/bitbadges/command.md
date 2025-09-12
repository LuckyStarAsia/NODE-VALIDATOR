---
hidden: true
---

# Command

## SEDA

## Useful commands

### 🔑 Key management

#### Add new key

```
bitwayd keys add wallet
```

#### Recover existing key

```
bitwayd keys add wallet --recover
```

#### List all keys

```
bitwayd keys list
```

#### Delete key

```
bitwayd keys delete wallet
```

#### Export key to the file

```
bitwayd keys export wallet
```

#### Import key from the file

```
bitwayd keys import wallet wallet.backup
```

#### Query wallet balance

```
bitwayd q bank balances $(bitwayd keys show wallet -a)
```

### 👷 Validator management

Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

#### Create new validator

```
bitwayd tx staking create-validator \
--amount 1000000ubtw \
--pubkey $(bitwayd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id bitwayd -1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10000ubtw \
-y
```

#### Edit existing validator

```
bitwayd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id seda-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10000ubtw \
-y
```

#### Unjail validator

```
bitwayd tx slashing unjail --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```

#### Jail reason

```
bitwayd query slashing signing-info $(bitwayd tendermint show-validator)
```

#### List all active validators

```
bitwayd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
bitwayd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
bitwayd q staking validator $(bitwayd keys show wallet --bech val -a)
```

### 💲 Token management

#### Withdraw rewards from all validators

```
bitwayd tx distribution withdraw-all-rewards --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Withdraw commission and rewards from your validator

```
bitwayd tx distribution withdraw-rewards $(bitwayd keys show wallet --bech val -a) --commission --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Delegate tokens to yourself

```
bitwayd tx staking delegate $(bitwayd keys show wallet --bech val -a) 10000ubtw --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Delegate tokens to validator

```
bitwayd tx staking delegate <TO_VALOPER_ADDRESS> 10000ubtw --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Redelegate tokens to another validator

```
bitwayd tx staking redelegate $(bitwayd keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 10000ubtw --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Unbond tokens from your validator

```
bitwayd tx staking unbond $(bitwayd keys show wallet --bech val -a) 10000ubtw --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Send tokens to the wallet

```
bitwayd tx bank send wallet <TO_WALLET_ADDRESS> 10000ubtw --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

### 🗳 Governance

#### List all proposals

```
bitwayd query gov proposals
```

#### View proposal by id

```
bitwayd query gov proposal 1
```

#### Vote 'Yes'

```
bitwayd tx gov vote 1 yes --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Vote 'No'

```
bitwayd tx gov vote 1 no --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Vote 'Abstain'

```
bitwayd tx gov vote 1 abstain --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

#### Vote 'NoWithVeto'

```
bitwayd tx gov vote 1 NoWithVeto --from wallet --chain-id bitway-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000ubtw -y
```

### ⚡️ Utility

#### Update ports

```
CUSTOM_PORT=258
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.bitway/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.bitway/config/app.toml
```

#### Update Indexer

Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.bitway/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.bitway/config/config.toml
```

Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.bitway/config/app.toml
```

### 🚨 Maintenance

#### Get validator info

```
bitwayd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
bitwayd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(bitwayd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.bitway/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Check if validator key is correct

```
[[ $(bitwayd q staking validator $(bitwayd keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(sedad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### Get live peers

```
curl -sS http://localhost:15857/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"10000ubtw\"/" $HOME/.bitway/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.bitway/config/config.toml
```

#### Reset chain data

```
bitwayd tendermint unsafe-reset-all --home $HOME/.bitway--keep-addr-book
```

#### Remove node

Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME
sudo systemctl stop bitwayd
sudo systemctl disable bitwayd
sudo rm /etc/systemd/system/bitwayd.sedad
sudo systemctl daemon-reload
rm -f $(which bitwayd)
rm -rf $HOME/.bitway
rm -rf $HOME/bitway
```

### ⚙️ Service Management

#### Reload sedad configuration

```
sudo systemctl daemon-reload
```

#### Enable sedad

```
sudo systemctl enable sedad
```

#### Disable sedad

```
sudo systemctl disable sedad
```

#### Start sedad

```
sudo systemctl start sedad
```

#### Stop sedad

```
sudo systemctl stop sedad
```

#### Restart sedad

```
sudo systemctl restart sedad
```

#### Check sedad status

```
sudo systemctl status sedad
```

#### Check sedad logs

```
sudo journalctl -u sedad -f --no-hostname -o cat
```

## END
