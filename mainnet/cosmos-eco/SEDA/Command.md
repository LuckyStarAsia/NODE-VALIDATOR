# SEDA

# Useful commands

## 🔑 Key management
### Add new key
```
sedad keys add wallet
```
### Recover existing key
```
sedad keys add wallet --recover
```
### List all keys
```
sedad keys list
```
### Delete key
```
sedad keys delete wallet
```
### Export key to the file
```
sedad keys export wallet
```
### Import key from the file
```
sedad keys import wallet wallet.backup
```
### Query wallet balance
```
sedad q bank balances $(sedad keys show wallet -a)
```
## 👷 Validator management
Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

### Create new validator
```
sedad tx staking create-validator \
--amount 10000000000aseda \
--pubkey $(sedad tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id seda-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10000000000aseda \
-y
```
### Edit existing validator
```
sedad tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id seda-1 \
--commission-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 10000000000aseda \
-y
```
### Unjail validator
```
sedad tx slashing unjail --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Jail reason
```
sedad query slashing signing-info $(sedad tendermint show-validator)
```
### List all active validators
```
sedad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
### List all inactive validators
```
sedad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```
### View validator details
```
sedad q staking validator $(sedad keys show wallet --bech val -a)
```
## 💲 Token management
### Withdraw rewards from all validators
```
sedad tx distribution withdraw-all-rewards --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Withdraw commission and rewards from your validator
```
sedad tx distribution withdraw-rewards $(sedad keys show wallet --bech val -a) --commission --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Delegate tokens to yourself
```
sedad tx staking delegate $(sedad keys show wallet --bech val -a) 10000000000aseda --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Delegate tokens to validator
```
sedad tx staking delegate <TO_VALOPER_ADDRESS> 10000000000aseda --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Redelegate tokens to another validator
```
sedad tx staking redelegate $(sedad keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 10000000000aseda --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Unbond tokens from your validator
```
sedad tx staking unbond $(sedad keys show wallet --bech val -a) 10000000000aseda --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Send tokens to the wallet
```
sedad tx bank send wallet <TO_WALLET_ADDRESS> 10000000000aseda --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
## 🗳 Governance
### List all proposals
```
sedad query gov proposals
```
### View proposal by id
```
sedad query gov proposal 1
```
### Vote 'Yes'
```
sedad tx gov vote 1 yes --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Vote 'No'
```
sedad tx gov vote 1 no --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Vote 'Abstain'
```
sedad tx gov vote 1 abstain --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
### Vote 'NoWithVeto'
```
sedad tx gov vote 1 NoWithVeto --from wallet --chain-id seda-1 --gas-adjustment 1.4 --gas auto --gas-prices 10000000000aseda -y
```
## ⚡️ Utility
### Update ports
```
CUSTOM_PORT=258
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.sedad/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.sedad/config/app.toml
```
### Update Indexer
Disable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.sedad/config/config.toml
```
Enable indexer
```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.sedad/config/config.toml
```
Update pruning
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.sedad/config/app.toml
```
## 🚨 Maintenance
### Get validator info
```
sedad status 2>&1 | jq .ValidatorInfo
```
### Get sync info
```
sedad status 2>&1 | jq .SyncInfo
```
### Get node peer
```
echo $(sedad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.sedad/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```
### Check if validator key is correct
```
[[ $(sedad q staking validator $(sedad keys show wallet --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(sedad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```
### Get live peers
```
curl -sS http://localhost:25857/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```
### Set minimum gas price
```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"10000000000aseda\"/" $HOME/.sedad/config/app.toml
```
### Enable prometheus
```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.sedad/config/config.toml
```
### Reset chain data
```
sedad tendermint unsafe-reset-all --home $HOME/.sedad --keep-addr-book
```
### Remove node
Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME
sudo systemctl stop sedad
sudo systemctl disable sedad
sudo rm /etc/systemd/system/sedad.sedad
sudo systemctl daemon-reload
rm -f $(which sedad)
rm -rf $HOME/.sedad
rm -rf $HOME/osmosis
```
## ⚙️ Service Management
### Reload sedad configuration
```
sudo systemctl daemon-reload
```
### Enable sedad
```
sudo systemctl enable sedad
```
### Disable sedad
```
sudo systemctl disable sedad
```
### Start sedad
```
sudo systemctl start sedad
```
### Stop sedad
```
sudo systemctl stop sedad
```
### Restart sedad
```
sudo systemctl restart sedad
```
### Check sedad status
```
sudo systemctl status sedad
```
### Check sedad logs
```
sudo journalctl -u sedad -f --no-hostname -o cat
```


# END
