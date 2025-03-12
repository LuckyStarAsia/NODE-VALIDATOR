# Command

<figure><img src="../../../.gitbook/assets/image (3).png" alt="" width="100"><figcaption><p>ZETA CHAIN</p></figcaption></figure>

## ZETA

## Useful commands

### 🔑 Key management

#### Add new key

```
zetacored keys add $WALLET
```

#### Recover existing key

```
zetacored keys add $WALLET --recover
```

#### List all keys

```
zetacored keys list
```

#### Delete key

```
zetacored keys delete $WALLET
```

#### Export key to the file

```
zetacored keys export $WALLET
```

#### Import key from the file

```
zetacored keys import $WALLET wallet.backup
```

#### Query wallet balance

```
zetacored q bank balances $(zetacored keys show $WALLET -a)
```

### 👷 Validator management

Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

#### Create new validator

```
zetacored tx staking create-validator \
--amount 10000000000000000000azeta \
--pubkey $(zetacored tendermint show-validator) \
--moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id zetachain_7000-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 2000000000000000000azeta \
-y
```

#### Edit existing validator

```
zetacored tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id zetachain_7000-1 \
--commission-rate 0.05 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--gas-prices 2000000000000000000azeta \
-y
```

#### Unjail validator

```
zetacored tx slashing unjail --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Jail reason

<pre><code><strong>zetacored query slashing signing-info $(zetacored tendermint show-validator)
</strong></code></pre>

#### List all active validators

```
zetacored q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
zetacored q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
zetacored q staking validator $(zetacored keys show $WALLET --bech val -a)
```

### 💲 Token management

#### Withdraw rewards from all validators

```
zetacored tx distribution withdraw-all-rewards --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Withdraw commission and rewards from your validator

```
zetacored tx distribution withdraw-rewards $(zetacored keys show $WALLET --bech val -a) --commission --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Delegate tokens to yourself

```
zetacored tx staking delegate $(zetacored keys show $WALLET --bech val -a) 2000000000000000000azeta --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Delegate tokens to validator

```
zetacored tx staking delegate <TO_VALOPER_ADDRESS> 2000000000000000000azeta --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Redelegate tokens to another validator

```
zetacored tx staking redelegate $(zetacored keys show $WALLET --bech val -a) <TO_VALOPER_ADDRESS> 2000000000000000000azeta --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Unbond tokens from your validator

```
zetacored tx staking unbond $(zetacored keys show $WALLET --bech val -a) 2000000000000000000azeta --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Send tokens to the wallet

```
zetacored tx bank send $WALLET <TO_WALLET_ADDRESS> 2000000000000000000axrp --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

### 🗳 Governance

#### List all proposals

```
zetacored query gov proposals
```

#### View proposal by id

```
zetacored query gov proposal 1
```

#### Vote 'Yes'

```
zetacored tx gov vote 1 yes --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Vote 'No'

```
zetacored tx gov vote 1 no --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Vote 'Abstain'

```
zetacored tx gov vote 1 abstain --from $WALLET --chain-id zetachain_7000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000azeta -y
```

#### Vote 'NoWithVeto'

```
zetacored tx gov vote 1 NoWithVeto --from $WALLET --chain-id xrplevm_1449000-1 --gas-adjustment 1.4 --gas auto --gas-prices 2000000000000000000axrp -y
```

### ⚡️ Utility

#### Update ports

```
CUSTOM_PORT=13
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.zetacored/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.zetacored/config/app.toml
```

#### Update Indexer

Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.zetacored/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.zetacored/config/config.toml
```

Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.zetacored/config/app.toml
```

### 🚨 Maintenance

#### Get validator info

```
zetacored status 2>&1 | jq .ValidatorInfo
```

#### Get sync info

```
zetacored status 2>&1 | jq .SyncInfo
```

#### Get node peer

```
echo $(zetacored tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.zetacored/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Check if validator key is correct

```
[[ $(zetacored q staking validator $(zetacored keys show $WALLET --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(zetacored status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### Get live peers

```
curl -sS http://localhost:13657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0azeta\"/" $HOME/.zetacored/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.zetacored/config/config.toml
```

#### Reset chain data

```
zetacored tendermint unsafe-reset-all --home $HOME/.zetacored --keep-addr-book
```

#### Remove node

Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME
sudo systemctl stop zetacored
sudo systemctl disable zetacored
sudo rm /etc/systemd/system/zetacored.service
sudo systemctl daemon-reload
rm -f $(which zetacored)
rm -rf $HOME/.zetacored
```

### ⚙️ Service Management

#### Reload sedad configuration

```
sudo systemctl daemon-reload
```

#### Enable sedad

```
sudo systemctl enable zetacored
```

#### Disable sedad

```
sudo systemctl disable zetacored
```

#### Start sedad

```
sudo systemctl start zetacored
```

#### Stop sedad

```
sudo systemctl stop zetacored
```

#### Restart sedad

```
sudo systemctl restart zetacored
```

#### Check sedad status

```
sudo systemctl status zetacored
```

#### Check sedad logs

```
sudo journalctl -u zetacored -f --no-hostname -o cat
```

## END
