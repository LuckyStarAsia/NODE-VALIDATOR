# Command

## COMMAND

### Key management 🔑

#### Add new key

```
atomoned keys add wallet
```

#### Recover existing key

```
atomoned keys add wallet --recover
```

#### List all keys

```
atomoned keys list
```

#### Delete key

```
atomoned keys delete wallet
```

#### Export key to the file

```
atomoned keys export wallet
```

#### Import key from the file

```
atomoned keys import wallet wallet.backup
```

#### Query wallet balance

```
atomoned q bank balances $(atomoned keys show wallet -a)
```

### Validator management 👷

* Please make sure you have adjusted moniker, identity, details and website to match your values.

#### Create new validator

```
atomoned tx staking create-validator --amount 1000000uatone --pubkey $(atomoned tendermint show-validator) --moniker "YOUR_MONIKER_NAME" --identity "YOUR_KEYBASE_ID" --details "YOUR_DETAILS" --website "YOUR_WEBSITE_URL" --chain-id atomone-1 --commission-rate 0.05 --commission-max-rate 0.20 --commission-max-change-rate 0.01 --min-self-delegation 1 --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Edit existing validator

```
atomoned tx staking edit-validator --new-moniker "YOUR_MONIKER_NAME" --identity "YOUR_KEYBASE_ID" --details "YOUR_DETAILS" --website "YOUR_WEBSITE_URL" --chain-id atomone-1 --commission-rate 0.05 --from wallet --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Unjail validator

```
atomoned tx slashing unjail --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Jail reason

```
atomoned query slashing signing-info $(atomoned tendermint show-validator)
```

#### List all active validators

```
atomoned q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " 	 " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
atomoned q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " 	 " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
atomoned q staking validator $(atomoned keys show wallet --bech val -a)
```

### Token management 💲

#### Withdraw rewards from all validators

```
atomoned tx distribution withdraw-all-rewards --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Withdraw commission and rewards from your validator

```
atomoned tx distribution withdraw-rewards $(atomoned keys show wallet --bech val -a) --commission --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Delegate tokens to yourself

```
atomoned tx staking delegate $(atomoned keys show wallet --bech val -a) 1000000uatone --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Delegate tokens to validator

```
atomoned tx staking delegate TO_VALOPER_ADDRESS 1000000uatone --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Redelegate tokens to another validator

```
atomoned tx staking redelegate $(atomoned keys show wallet --bech val -a) TO_VALOPER_ADDRESS 1000000uatone --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Unbond tokens from your validator

```
atomoned tx staking unbond $(atomoned keys show wallet --bech val -a) 1000000uatone --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Send tokens to the wallet

```
atomoned tx bank send wallet TO_WALLET_ADDRESS 1000000uatone --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

### Governance 🗳

#### List all proposals

```
atomoned query gov proposals
```

#### View proposal by id

```
atomoned query gov proposal 1
```

#### Vote 'Yes'

```
atomoned tx gov vote 1 yes --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Vote 'No'

```
atomoned tx gov vote 1 no --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Vote 'Abstain'

```
atomoned tx gov vote 1 abstain --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

#### Vote 'NoWithVeto'

```
atomoned tx gov vote 1 NoWithVeto --from wallet --chain-id atomone-1 --gas-adjustment 1.4 --gas auto --gas-prices 0.025uatone -y
```

### Utility ⚡️

#### Update Indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.atomone/config/config.toml
```

#### Update pruning

```
sed -i   -e 's|^pruning *=.*|pruning = "custom"|'   -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|'   -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|'   -e 's|^pruning-interval *=.*|pruning-interval = "19"|'   $HOME/.atomone/config/app.toml
```

### Maintenance 🚨

#### Get validator info

```
atomoned status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
atomoned status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(atomoned tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.atomone/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Get live peers

```
curl -sS http://localhost:11657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = "0.025uatone"/" $HOME/.atomone/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.atomone/config/config.toml
```

#### Reset chain data

```
atomoned tendermint unsafe-reset-all --home $HOME/.atomone --keep-addr-book
```

#### Remove node

_**- Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your priv\_validator\_key.json!**_

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

### Service Management ⚙️

#### Reload service configuration

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable atomoned
```

#### Disable service

```
sudo systemctl disable atomoned
```

#### Start service

```
sudo systemctl start atomoned
```

#### Stop service

```
sudo systemctl stop atomoned
```

#### Restart service

```
sudo systemctl restart atomoned
```

#### Check service status

```
sudo systemctl status atomoned
```

#### Check service logs

```
sudo journalctl -u atomoned -f --no-hostname -o cat
```

## END
