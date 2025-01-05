

# COMMAND

## WALLET MANAGEMENT


Add Wallet Specify the value <wallet> with your own wallet name


### Add new wallet:

```
emped keys add wallet
```


### Recover Wallet:

```
emped keys add wallet --recover
```


### List Wallet:

```
emped keys list
```


### Delete Wallet:
```
emped keys delete wallet
```



### Check Wallet Balance:
```
emped q bank balances $(emped keys show wallet -a)
```


## VALIDATOR MANAGEMENT

Please adjust , `MONIKER` , `YOUR_KEYBASE_ID` , `YOUR_DETAILS` , `YOUR_WEBSITE_URL`


### Create Validator:

```
emped tx staking create-validator \
--amount=1000000uempe \
--pubkey=$(emped tendermint show-validator) \
--moniker=$MONIKER \
--identity="YOUR_KEYBASE_ID" \
--details="YOUR_DETAILS" \
--website="YOUR_WEBSITE_URL" \
--chain-id=$EMPE_CHAIN_ID \
--commission-rate=0.10 \
--commission-max-rate=0.20 \
--commission-max-change-rate=0.01 \
--min-self-delegation=1000 \
--from=wallet \
--gas-adjustment=1.5 \
--gas="auto" \
--gas-prices=1uempe\ 
-y
```


### Edit Validator
```
emped tx staking edit-validator \
--new-moniker="YOUR MONIKER" \
--identity="IDENTITY KEYBASE" \
--details="DETAILS VALIDATOR" \
--website="LINK WEBSITE" \
--chain-id=$EMPE_CHAIN_ID \
--from=wallet \
--gas-adjustment=1.5 \
--gas="auto" \
--gas-prices=1uempe\ 
-y
```


### Unjail Validator:

```
emped tx slashing unjail --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```


### Check Jailed Reason:
```
emped query slashing signing-info $(emped tendermint show-validator)
```

## TOKEN MANAGEMENT


### Withdraw Rewards:
```
emped tx distribution withdraw-all-rewards --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```

### Withdraw Rewards with Comission:
```
emped tx distribution withdraw-rewards $(emped keys show wallet --bech val -a) --commission --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```

### Delegate Token to your own validator:

```
emped tx staking delegate $(emped keys show wallet --bech val -a) 100000uempe --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```


### Delegate Token to other validator:

```
emped tx staking redelegate $(emped keys show wallet --bech val -a) <TO_VALOPER_ADDRESS> 100000uempe --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```


### Unbond Token from your validator:

```
emped tx staking unbond $(emped keys show wallet --bech val -a) 100000uempe --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```


### Send Token to another wallet

```
emped tx bank send wallet <TO_WALLET_ADDRESS> 100000uempe --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```


## GOVERNANCE



### Vote, You can change the value of yes to no,abstain,nowithveto
```
emped tx gov vote 1 yes --from wallet --chain-id $EMPE_CHAIN_ID --gas-adjustment 1.5 --gas auto --gas-prices 1uempe -y
```

### List all proposals
```
emped query gov proposals
```


### Create new text proposal
```
emped tx gov submit-proposal \
--title="Title" \
--description="Description" \
--deposit=10000000uempe \
--type="Text" \
--from=wallet \
--gas-adjustment 1.5 \
--gas "auto" \
--gas-prices=1uempe\ 
-y 
```


## SERVICES MANAGEMENT



### Reload Service
```
sudo systemctl daemon-reload
```

### Enable Service
```
sudo systemctl enable emped
```

### Disable Service
```
sudo systemctl disable emped
```

### Start Service
```
sudo systemctl start emped
```

### Stop Service
```
sudo systemctl stop emped
```

### Restart Service
```
sudo systemctl restart emped
```

### Check Service Status
```
sudo systemctl status emped
```

### Check Service Logs
```
sudo journalctl -u emped -f --no-hostname -o cat
```


## UTILITY


### Set Indexer NULL / KV
```
sed -i 's|^indexer *=.*|indexer = "null"|' $HOME/.empe-chain/config/config.toml
```

### Get Validator info
```
emped status 2>&1 | jq .ValidatorInfo
```

### Get denom info
```
emped q bank denom-metadata -oj | jq
```

### Get sync status
```
emped status 2>&1 | jq .SyncInfo.catching_up
```


### Get latest height
```
emped status 2>&1 | jq .SyncInfo.latest_block_height
```

### Reset Node
```
emped tendermint unsafe-reset-all --home $HOME/.empe-chain --keep-addr-book
```


## DELETE NODE
WARNING! Use this command wisely Backup your key first it will remove node from your system

**`SAVE YOUR PRIVATE KEYS VALIDATOR & MNEMONIC PHRASE`**

```
sudo systemctl stop emped
sudo systemctl disable emped
sudo rm -rf /etc/systemd/system/emped.service
sudo systemctl daemon-reload
sudo rm -f $(which emped)
sudo rm -rf $HOME/.empe-chain
```


```
sudo systemctl stop emped && sudo systemctl disable emped && sudo rm /etc/systemd/system/emped.service && sudo systemctl daemon-reload && sudo rm -rf $(which emped) && rm -rf $HOME/.empe-chain && rm -rf $HOME/go/bin/emped
```


