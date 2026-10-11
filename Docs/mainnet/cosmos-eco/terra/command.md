# Command

## TERRA

## Useful commands

### 🔑 Key management

#### Add new key

```
terrad keys add $WALLET
```

#### Recover existing key

```
terrad keys add $WALLET --recover
```

#### List all keys

```
terrad keys list
```

#### Delete key

```
terrad keys delete $WALLET
```

#### Export key to the file

```
terrad keys export $WALLET
```

#### Import key from the file

```
terrad keys import $WALLET wallet.backup
```

#### Query wallet balance

```
terrad q bank balances $(terrad keys show $WALLET -a)
```

### 👷 Validator management

Please make sure you have adjusted `moniker`, `identity`, `details` and `website` to match your values.

#### Create new validator

```
terrad tx staking create-validator <(cat <<EOF
{
  "pubkey": $(terrad comet show-validator),
  "amount": "1000000uluna",
  "moniker": "$MONIKER",
  "identity": "",
  "website": "",
  "security-contact": "",
  "details": "",
  "commission-rate": "0.05",
  "commission-max-rate": "1",
  "commission-max-change-rate": "0.5",
  "min-self-delegation": "1"
}
EOF
) \
  --from $WALLET \
  --chain-id phoenix-1 \
  --gas auto --gas-adjustment 1.4 \
  --fees 5000uluna \
  -y
```

#### Edit existing validator

```
terrad tx staking edit-validator \
--new-moniker "YOUR_MONIKER_NAME" \
--identity "YOUR_KEYBASE_ID" \
--details "YOUR_DETAILS" \
--website "YOUR_WEBSITE_URL" \
--chain-id phoenix-1 \
--commission-rate 0.05 \
--from $WALLET \
--gas-adjustment 1.4 \
--gas auto \
--fees 5000uluna \
-y
```

#### Unjail validator

```
terrad tx slashing unjail --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Jail reason

```
terrad query slashing signing-info $(terrad tendermint show-validator)
```

#### List all active validators

```
terrad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### List all inactive validators

```
terrad q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_UNBONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

#### View validator details

```
terrad q staking validator $(terrad keys show $WALLET --bech val -a)
```

### 💲 Token management

#### Withdraw rewards from all validators

```
terrad tx distribution withdraw-all-rewards --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Withdraw commission and rewards from your validator

```
terrad tx distribution withdraw-rewards $(terrad keys show $WALLET --bech val -a) --commission --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto ---fees 5000uluna -y
```

#### Delegate tokens to yourself

```
terrad tx staking delegate $(terrad keys show $WALLET --bech val -a) 10000000aseda --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Delegate tokens to validator

```
terrad tx staking delegate <TO_VALOPER_ADDRESS> 10000000000aseda --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Redelegate tokens to another validator

```
terrad tx staking redelegate $(terrad keys show $WALLET --bech val -a) <TO_VALOPER_ADDRESS> 10000000000uluna --from $WALLET --chain-id seda-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Unbond tokens from your validator

```
terrad tx staking unbond $(terrad keys show $WALLET --bech val -a) 10000000uluna --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Send tokens to the wallet

```
terrad tx bank send $WALLET <TO_WALLET_ADDRESS> 10000000uluna --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

### 🗳 Governance

#### List all proposals

```
terrad query gov proposals
```

#### View proposal by id

```
terrad query gov proposal 1
```

#### Vote 'Yes'

```
terrad tx gov vote 1 yes --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Vote 'No'

```
terrad tx gov vote 1 no --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Vote 'Abstain'

```
sedad tx gov vote 1 abstain --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

#### Vote 'NoWithVeto'

```
terrad tx gov vote 1 NoWithVeto --from $WALLET --chain-id phoenix-1 --gas-adjustment 1.4 --gas auto --fees 5000uluna -y
```

### ⚡️ Utility

#### Update ports

```
CUSTOM_PORT=258
sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${CUSTOM_PORT}58\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${CUSTOM_PORT}57\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${CUSTOM_PORT}60\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${CUSTOM_PORT}56\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${CUSTOM_PORT}66\"%" $HOME/.terra/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${CUSTOM_PORT}17\"%; s%^address = \":8080\"%address = \":${CUSTOM_PORT}80\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${CUSTOM_PORT}90\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${CUSTOM_PORT}91\"%" $HOME/.terra/config/app.toml
```

#### Update Indexer

Disable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "null"|' $HOME/.terra/config/config.toml
```

Enable indexer

```
sed -i -e 's|^indexer *=.*|indexer = "kv"|' $HOME/.terra/config/config.toml
```

Update pruning

```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "107"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "13"|' \
  $HOME/.terra/config/app.toml
```

### 🚨 Maintenance

#### Get validator info

```
terrad status 2>&1 | jq .validator_info
```

#### Get sync info

```
terrad status 2>&1 | jq .sync_info
```

#### Get node peer

```
echo $(terrad tendermint show-node-id)'@'$(curl -s ifconfig.me)':'$(cat $HOME/.terra/config/config.toml | sed -n '/Address to listen for incoming connection/{n;p;}' | sed 's/.*://; s/".*//')
```

#### Check if validator key is correct

```
[[ $(terrad q staking validator $(terrad keys show $WALLET --bech val -a) -oj | jq -r .consensus_pubkey.key) = $(sedad status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

#### Get live peers

```
curl -sS http://localhost:25857/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

#### Set minimum gas price

```
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.015uluna\"/" $HOME/.terra/config/app.toml
```

#### Enable prometheus

```
sed -i -e "s/prometheus = false/prometheus = true/" $HOME/.terra/config/config.toml
```

#### Reset chain data

```
terrad tendermint unsafe-reset-all --home $HOME/.terra --keep-addr-book
```

#### Remove node

Please, before proceeding with the next step! All chain data will be lost! Make sure you have backed up your `priv_validator_key.json`!

```
cd $HOME && \
sudo systemctl stop terrad && \
sudo systemctl disable terrad && \
sudo rm -rf /etc/systemd/system/terrad.service && \
sudo rm $(which terrad) && \
sudo rm -rf $HOME/.terra && \
sudo rm -rf $HOME/core
```

### ⚙️ Service Management

#### Reload sedad configuration

```
sudo systemctl daemon-reload
```

#### Enable sedad

```
sudo systemctl enable terrad
```

#### Disable sedad

```
sudo systemctl disable terrad
```

#### Start sedad

```
sudo systemctl start terrad
```

#### Stop sedad

```
sudo systemctl stop terrad
```

#### Restart sedad

```
sudo systemctl restart terrad
```

#### Check sedad status

```
sudo systemctl status sedad
```

#### Check sedad logs

```
sudo journalctl -u terrad -f --no-hostname -o cat
```

## END
