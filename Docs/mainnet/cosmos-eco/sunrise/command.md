---
cover: ../../../.gitbook/assets/1500x500.jfif
coverY: 0
---

# Command

## COMMAND

### Key management 🔑

#### Add new key

```
sunrised keys add wallet
```

#### Recover existing key

```
sunrised keys add wallet --recover
```

#### List all keys

```
sunrised keys list
```

#### Delete key

```
sunrised keys delete wallet
```

#### Export key to the file

```
sunrised keys export wallet
```

#### Import key from the file

```
sunrised keys import wallet wallet.backup
```

#### Query wallet balance

```
sunrised q bank balances $(sunrised keys show wallet -a)
```

### Validator management 👷

* Please make sure you have adjusted moniker, identity, details and website to match your values.

#### Create new validator

```
sunrised tx staking create-validator --amount 1000000uvrise --pubkey $(sunrised tendermint show-validator) --moniker "YOUR_MONIKER_NAME" --identity "YOUR_KEYBASE_ID" --details "YOUR_DETAILS" --website "YOUR_WEBSITE_URL" --chain-id sunrise-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Edit existing validator

```
sunrised tx staking edit-validator --new-moniker "YOUR_MONIKER_NAME" --identity "YOUR_KEYBASE_ID" --details "YOUR_DETAILS" --website "YOUR_WEBSITE_URL" --chain-id sunrise-1 --commission-rate 0.05 --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Unjail validator

```
sunrised tx slashing unjail --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Jail reason

```
sunrised query slashing signing-info $(sunrised tendermint show-validator)
```

#### List all active validators

```
sunrised q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " 	 " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
sunrised q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " 	 " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
sunrised q staking validator $(sunrised keys show wallet --bech val -a)
```

### Token management 💲

#### Withdraw rewards from all validators

```
sunrised tx distribution withdraw-all-rewards --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Withdraw commission and rewards from your validator

```
sunrised tx distribution withdraw-rewards $(sunrised keys show wallet --bech val -a) --commission --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Delegate tokens to yourself

```
sunrised tx staking delegate $(sunrised keys show wallet --bech val -a) 1000000uvrise --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Delegate tokens to validator

```
sunrised tx staking delegate TO_VALOPER_ADDRESS 1000000uvrise --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Redelegate tokens to another validator

```
sunrised tx staking redelegate $(sunrised keys show wallet --bech val -a) TO_VALOPER_ADDRESS 1000000uvrise --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Unbond tokens from your validator

```
sunrised tx staking unbond $(sunrised keys show wallet --bech val -a) 1000000uvrise --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Send tokens to the wallet

```
sunrised tx bank send wallet TO_WALLET_ADDRESS 1000000urise --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

### Governance 🗳

#### List all proposals

```
sunrised query gov proposals
```

#### View proposal by id

```
sunrised query gov proposal 1
```

#### Vote 'Yes'

```
sunrised tx gov vote 1 yes --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Vote 'No'

```
sunrised tx gov vote 1 no --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Vote 'Abstain'

```
sunrised tx gov vote 1 abstain --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

#### Vote 'NoWithVeto'

```
sunrised tx gov vote 1 NoWithVeto --from wallet --chain-id sunrise-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uusdrise -y
```

### Utility ⚡️

#### Update Indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.sunrise/config/config.toml
```

#### Update pruning

```
sed -i   -e 's|^pruning *=.*|pruning = "custom"|'   -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|'   -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|'   -e 's|^pruning-interval *=.*|pruning-interval = "19"|'   $HOME/.sunrise/config/app.toml
```

### Maintenance 🚨

#### Get validator info

```
sunrised status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
sunrised status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(sunrised tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.sunrise/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Get live peers

```
curl -sS http://localhost:11657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = "0.025urise"/" $HOME/.sunrise/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sunrise/config/config.toml
```

#### Reset chain data

```
sunrised tendermint unsafe-reset-all --home $HOME/.sunrise --keep-addr-book
```

#### Remove node

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

### Service Management ⚙️

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable sunrised
```

#### Disable service

```
sudo systemctl disable sunrised
```

#### Start service

```
sudo systemctl start sunrised
```

#### Stop service

```
sudo systemctl stop sunrised
```

#### Restart service

```
sudo systemctl restart sunrised
```

#### Check service status

```
sudo systemctl status sunrised
```

#### Check service logs

```
sudo journalctl -u sunrised -f --no-hostname -o cat
```

## END
