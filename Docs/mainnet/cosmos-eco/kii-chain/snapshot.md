# Snapshot

## Stop the service

```
sudo systemctl stop kiichaind.service
```

## Reset the data and save validator state

```
cp $HOME/.kiichain/data/priv_validator_state.json $HOME/.kiichain/priv_validator_state.json.backup
rm -rf $HOME/.kiichain/data/*
rm -rf $HOME/.kiichain/wasm/*
```

## Download `latest` snapshot

```
curl -L https://kiichain-mainnet-services.luckystar.asia/kii/kiichain_1783-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain/data
curl -L https://kiichain-mainnet-services.luckystar.asia/kii/wasm_kiichain_1783-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain/wasm
mv $HOME/.kiichain/priv_validator_state.json.backup $HOME/.kiichain/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl restart kiichaind.service && sudo journalctl -fu kiichaind.service -o cat
```
