---
hidden: true
---

# Command

## XRPLEVM

## Useful commands

### 🔑 Key management

#### Add new key

```
exrpd keys add $WALLET --key-type eth_secp256k1
```

#### Recover existing key

```
exrpd keys add $WALLET --recover
```

#### List all keys

```
exrpd keys list
```

#### Delete key

```
exrpd keys delete $WALLET
```

#### Export key to the file

```
exrpd keys export $WALLET
```

#### Import key from the file

```
exrpd keys import $WALLET wallet.backup
```

#### Query wallet balance

```
exrpd q bank balances $(exrpd keys show $WALLET -a)
```

### 👷 Validator management

Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

#### Create new validator

```
exrpd tx staking create-validator \
--amount 10000000000000000000axrp \
--pubkey $(exrpd tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id xrplevm_1449000-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 2000000000000000000axrp \
-y
```

#### Edit existing validator

```
exrpd tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id xrplevm_1449000-1 \
--commission-rate 0.05 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 2000000000000000000axrp \
-y
```

#### Unjail validator

```
exrpd tx slashing unjail --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Jail reason

```
exrpd query slashing signing-info $(exrpd tendermint show-validator)
```

#### List all active validators

```
exrpd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
exrpd q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
exrpd q staking validator $(exrpd keys show $WALLET --bech val -a)
```

### 💲 Token management

#### Withdraw rewards from all validators

```
exrpd tx distribution withdraw-all-rewards --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Withdraw commission and rewards from your validator

```
exrpd tx distribution withdraw-rewards $(exrpd keys show $WALLET --bech val -a) --commission --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Delegate tokens to yourself

```
exrpd tx staking delegate $(exrpd keys show $WALLET --bech val -a) 2000000000000000000axrp --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Delegate tokens to validator

```
exrpd tx staking delegate <TO_VALOPER_ADDRESS> 2000000000000000000axrp --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Redelegate tokens to another validator

```
exrpd tx staking redelegate $(exrpd keys show $WALLET --bech val -a) <TO_VALOPER_ADDRESS> 2000000000000000000axrp --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Unbond tokens from your validator

```
exrpd tx staking unbond $(exrpd keys show $WALLET --bech val -a) 2000000000000000000axrp --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Send tokens to the wallet

```
exrpd tx bank send $WALLET <TO_WALLET_ADDRESS> 2000000000000000000axrp --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

### 🗳 Governance

#### List all proposals

```
exrpd query gov proposals
```

#### View proposal by id

```
exrpd query gov proposal 1
```

#### Vote 'Yes'

```
exrpd tx gov vote 1 yes --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Vote 'No'

```
exrpd tx gov vote 1 no --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Vote 'Abstain'

```
exrpd tx gov vote 1 abstain --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

#### Vote 'NoWithVeto'

```
exrpd tx gov vote 1 NoWithVeto --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

### ⚡️ Utility

#### Update ports

```
CUSTOM_PORT=258
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.exrpd/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.exrpd/config/app.toml
```

#### Update Indexer

Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.exrpd/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.exrpd/config/config.toml
```

Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.exrpd/config/app.toml
```

### 🚨 Maintenance

#### Get validator info

```
exrpd status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
exrpd status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(exrpd tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.exrpd/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Check if validator key is correct

```
[[ $(exrpd q staking validator $(exrpd keys show $WALLET --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(exrpd status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### Get live peers

```
curl -sS http://localhost:19657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"2000000000000000000axrp\"/" $HOME/.exrpd/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.exrpd/config/config.toml
```

#### Reset chain data

```
exrpd tendermint unsafe-reset-all --home $HOME/.exrpd --keep-addr-book
```

#### Remove node

Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME
sudo systemctl stop exrpdtest
sudo systemctl disable exrpdtest
sudo rm /etc/systemd/system/exrpdtest.service
sudo systemctl daemon-reload
rm -f $(which exrpd)
rm -rf $HOME/.exrpd
rm -rf $HOME/xrp
```

### ⚙️ Service Management

#### Reload sedad configuration

```
sudo systemctl daemon-reload
```

#### Enable sedad

```
sudo systemctl enable exrpdtest
```

#### Disable sedad

```
sudo systemctl disable exrpdtest
```

#### Start sedad

```
sudo systemctl start exrpdtest
```

#### Stop sedad

```
sudo systemctl stop exrpdtest
```

#### Restart sedad

```
sudo systemctl restart exrpdtest
```

#### Check sedad status

```
sudo systemctl status exrpdtest
```

#### Check sedad logs

```
sudo journalctl -u exrpdtest -f --no-hostname -o cat
```

## END
