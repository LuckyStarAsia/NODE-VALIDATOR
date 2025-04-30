# Snapshot

## Stop the service

```
sudo systemctl stop kiidtest.service
```

## Reset the data and save validator state

```
cp $HOME/.kiichain/data/priv_validator_state.json $HOME/.kiichain/priv_validator_state.json.backup
rm -rf $HOME/.kiichain/data/*
rm -rf $HOME/.kiichain/wasm/*
```

## Download `latest` snapshot

```
curl -L https://kii-testnet-services.luckystar.asia/kii/oro_1336-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain/data
curl -L https://kii-testnet-services.luckystar.asia/kii/wasm_oro_1336-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain/wasm
mv $HOME/.kiichain/priv_validator_state.json.backup $HOME/.kiichain/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start kiidtest.service && sudo journalctl -fu kiidtest.service -o cat
```
