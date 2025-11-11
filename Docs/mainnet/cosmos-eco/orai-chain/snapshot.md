# Snapshot

## Stop the service

```
sudo systemctl stop oraid.service
```

## Reset the data and save validator state

```
cp $HOME/.oraid/data/priv_validator_state.json $HOME/.oraid/priv_validator_state.json.backup
rm -rf $HOME/.oraid/data/*
rm -rf $HOME/.oraid/wasm/*
```

## Download `latest` snapshot

```
curl -L https://orai-mainnet-services.luckystar.asia/orai/Oraichain_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.oraid/data
curl -L https://orai-mainnet-services.luckystar.asia/orai/wasm_Oraichain_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.oraid/wasm
```

```
mv $HOME/.oraid/priv_validator_state.json.backup $HOME/.oraid/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start oraid.service && sudo journalctl -fu oraid.service -o cat
```
