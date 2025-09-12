# Command

## CLI Cheatsheet

### Key management

#### Add new key

```
gaiad keys add $WALLET
```

#### Recover existing key

```
gaiad keys add $WALLET --recover
```

#### List all keys

```
gaiad keys list
```

#### Delete key

```
gaiad keys delete $WALLET
```

#### Export key to the file

```
gaiad keys export $WALLET
```

#### Import key from the file

```
gaiad keys import $WALLET wallet.backup
```

#### Wallet balance

```
gaiad q bank balances $(gaiad keys show $WALLET -a)
```

### Token management

#### Set withdraw address:

```
gaiad tx distribution set-withdraw-addr <New-withdraw-address> --from $WALLET
```

#### Withdraw rewards from all validators

```
gaiad tx distribution withdraw-all-rewards --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Withdraw rewards and commissions from your validator

```
gaiad tx distribution withdraw-rewards $(gaiad keys show $WALLET --bech val -a) --commission --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Delegate tokens to yourself

```
gaiad tx staking delegate $(gaiad keys show $WALLET --bech val -a) 1000000uatom --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Delegate tokens to validator

```
gaiad tx staking delegate <TO_VALOPER_ADDRESS> 1000000uatom --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Redelegate tokens to another validator

```
gaiad tx staking redelegate $(gaiad keys show $WALLET --bech val -a) <TO_VALOPER_ADDRESS> 1000000uatom --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Unbond tokens from your validator

```
gaiad tx staking unbond $(gaiad keys show $WALLET --bech val -a) 1000000uatom --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Send tokens to the wallet

```
gaiad tx bank send $WALLET <TO_WALLET_ADDRESS> 1000000uatom --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

### Validator management

#### Validator info

```
gaiad status 2>&1 | jq .validator_info
```

#### Validator details

```
gaiad q staking validator $(gaiad keys show $WALLET --bech val -a)
```

#### Check if validator key is correct

```
[[ $(gaiad q staking validator $(gaiad keys show $WALLET --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(gaiad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### List all active validators

```
gaiad q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
gaiad q staking validators -oj --limit=1000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### Edit existing validator

```
gaiad tx staking edit-validator \
  --new-moniker "YOUR_MONIKER_NAME" \
  --identity "YOUR_KEYBASE_ID" \
  --details "YOUR_DETAILS" \
  --website "YOUR_WEBSITE_URL" \
  --security-contact "YOUR_EMAIL_ADDRESS" \
  --chain-id $GAIA_CHAIN_ID \
  --commission-rate 0.05 \
  --from $WALLET \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 3000uatom \
  -y
```

#### Jail reason

```
gaiad query slashing signing-info $(gaiad tendermint show-validator)
```

#### Unjail validator

<pre><code><strong>gaiad tx slashing unjail --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
</strong></code></pre>

### Governance

#### Create a new offer

```
gaiad tx gov submit-proposal \
  --title "" \
  --description "" \
  --deposit 10000000uatom \
  --type Text
  --from $WALLET \
  --gas-adjustment 1.4 \
  --gas auto \
  --gas-prices 3000uatom \
  -y
```

#### List all proposals

```
gaiad query gov proposals
```

#### View proposal by ID

```
gaiad query gov proposal 1
```

#### Vote “YES”

```
gaiad tx gov vote 1 yes --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Vote “NO”

```
gaiad tx gov vote 1 no --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Vote “ABSTAIN”

```
gaiad tx gov vote 1 abstain --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

#### Vote “NOWITHVETO”

```
gaiad tx gov vote 1 NoWithVeto --from $WALLET --chain-id $GAIA_CHAIN_ID --gas-adjustment 1.4 --gas auto --gas-prices 3000uatom -y
```

### Maintenance

#### Get sync info

```
gaiad status 2>&1 | jq .sync_info
```

#### Get node peer

```
echo $(gaiad tendermint show-node-id)'@'$(wget -qO- eth0.me)':'$(cat $HOME/.gaia/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Get live peers

```
curl -sS http://localhost:19657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Enable Prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.gaia/config/config.toml
```

#### Set minimum gas price

```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.025uatom\"|" $HOME/.gaia/config/app.toml
```

#### Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.gaia/config/config.toml
```

#### Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.gaia/config/config.toml
```

#### Update pruning

```
sed -i -e 's|^pruning *=.*|pruning = "custom"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-keep-recent  *=.*|pruning-keep-recent = "100"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' $HOME/.gaia/config/app.toml
sed -i -e 's|^pruning-interval *=.*|pruning-interval = "10"|' $HOME/.gaia/config/app.toml
```

#### Filter peers and max peers

```
sed -i -e 's|^filter_peers *=.*|filter_peers = "true"|' $HOME/.gaia/config/config.toml
sed -i -e 's|^max_num_inbound_peers *=.*|max_num_inbound_peers = "50"|' $HOME/.gaia/config/config.toml
sed -i -e 's|^max_num_outbound_peers *=.*|max_num_outbound_peers = "20"|' $HOME/.gaia/config/config.toml
```

#### Update ports

```
CUSTOM_PORT=136
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.gaia/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.gaia/config/app.toml
```

#### Reset chain data

```
gaiad tendermint unsafe-reset-all --keep-addr-book --home $HOME/.gaia
```

#### Delete node

```
sudo systemctl stop gaiad
sudo systemctl disable gaiad
sudo rm -rf /etc/systemd/system/gaiad.service
sudo systemctl daemon-reload
sudo rm -f $(which gaiad)
sudo rm -rf $HOME/.gaia
sudo rm -rf $HOME/gaia
```

### Service Management

#### Status service

```
sudo systemctl status gaiad
```

#### Start service

```
sudo systemctl start gaiad
```

#### Stop service

```
sudo systemctl stop gaiad
```

#### Restart service

```
sudo systemctl restart gaiad
```

#### Logs service

```
sudo journalctl -u gaiad -f --no-hostname -o cat
```

#### Reload service

```
sudo systemctl daemon-reload
```

#### Enable service

```
sudo systemctl enable gaiad
```

#### Disable service

```
sudo systemctl disable gaiad
```

## END
