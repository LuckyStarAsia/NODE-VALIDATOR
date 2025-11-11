# Command

## ORAI

## Useful commands

### 🔑 Key management

#### Add new key

```
oraid keys add wallet
```

#### Recover existing key

```
oraid keys add wallet --recover
```

#### List all keys

```
oraid keys list
```

#### Delete key

```
oraid keys delete wallet
```

#### Export key to the file

```
oraid keys export wallet
```

#### Import key from the file

```
oraid keys import wallet wallet.backup
```

#### Query wallet balance

```
oraid q bank balances $(oraid keys show wallet -a)
```

### 👷 Validator management

Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

#### Create new validator

```
oraid tx staking create-validator \
--amount 1000000orai \
--pubkey $(oraid tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id Oraichain \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10orai \
-y
```

#### Edit existing validator

```
oraid tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id Oraichain \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10orai \
-y
```

#### Unjail validator

```
oraid tx slashing unjail --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Jail reason

```
oraid query slashing signing-info $(oraid tendermint show-validator)
```

#### List all active validators

```
oraid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
oraid q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
oraid q staking validator $(oraid keys show wallet --bech val -a)
```

### 💲 Token management

#### Withdraw rewards from all validators

```
oraid tx distribution withdraw-all-rewards --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Withdraw commission and rewards from your validator

```
oraid tx distribution withdraw-rewards $(oraid keys show wallet --bech val -a) --commission --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Delegate tokens to yourself

```
oraid tx staking delegate $(oraid keys show wallet --bech val -a) 10orai --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Delegate tokens to validator

```
oraid tx staking delegate <TO_VALOPER_ADDRESS> 1000000orai --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Redelegate tokens to another validator

```
oraid tx staking redelegate $(oraid keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 10orai --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Unbond tokens from your validator

```
oraid tx staking unbond $(oraid keys show wallet --bech val -a) 10orai --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Send tokens to the wallet

```
oraid tx bank send wallet <TO_WALLET_ADDRESS> 10orai --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

### 🗳 Governance

#### List all proposals

```
oraid query gov proposals
```

#### View proposal by id

```
oraid query gov proposal 1
```

#### Vote 'Yes'

```
oraid tx gov vote 1 yes --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Vote 'No'

```
oraid tx gov vote 1 no --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Vote 'Abstain'

```
oraid tx gov vote 1 abstain --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

#### Vote 'NoWithVeto'

```
oraid tx gov vote 1 NoWithVeto --from wallet --chain-id Oraichain --gas-adjustment 1.4 --gas auto --gas-prices 10orai -y
```

### ⚡️ Utility

#### Update ports

```
CUSTOM_PORT=258
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.oraid/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.oraid/config/app.toml
```

#### Update Indexer

Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.oraid/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.oraid/config/config.toml
```

Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.oraid/config/app.toml
```

### 🚨 Maintenance

#### Get validator info

```
oraid status 2>&1 | jq .validator_info
```

#### Get sync info

```
oraid status 2>&1 | jq .sync_info
```

#### Get node peer

```
echo $(oraid tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.oraid/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Check if validator key is correct

```
[[ $(oraid q staking validator $(oraid keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(oraid status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### Get live peers

```
curl -sS http://localhost:25857/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"10orai\"/" $HOME/.oraid/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.oraid/config/config.toml
```

#### Reset chain data

```
oraid tendermint unsafe-reset-all --home $HOME/.oraid --keep-addr-book
```

#### Remove node

Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME
sudo systemctl stop oraid
sudo systemctl disable oraid
sudo rm /etc/systemd/system/oraid.service
sudo systemctl daemon-reload
rm -f $(which oraid)
rm -rf $HOME/.oraid
rm -rf $HOME/wasmd
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
