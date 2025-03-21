# Snapshot

## Stop the service

```
sudo systemctl stop kiidtest.service
```

## Reset the data and save validator state

```
cp $HOME/.kiichain3/data/priv_validator_state.json $HOME/.kiichain3/priv_validator_state.json.backup
rm -rf $HOME/.kiichain3/data/*
rm -rf $HOME/.kiichain3/wasm/*
```

## Download `latest` snapshot

```
curl -L https://kii-testnet-services.luckystar.asia/kii/kiichain3_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain3/data
curl -L https://kii-testnet-services.luckystar.asia/kii/wasm_kiichain3_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.kiichain3/wasm
mv $HOME/.kiichain3/priv_validator_state.json.backup $HOME/.kiichain3/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start kiidtest.service && sudo journalctl -fu kiidtest.service -o cat
```
