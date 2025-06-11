# Snapshot

## Stop the service

```
sudo systemctl stop emped.service
```

## Reset the data and save validator state

```
cp $HOME/.empe-chain/data/priv_validator_state.json $HOME/.empe-chain/priv_validator_state.json.backup
rm -rf $HOME/.empe-chain/data/*
rm -rf $HOME/.empe-chain/wasm/*
```

## Download `latest` snapshot

```
curl -L https://empeiria-services.luckystar.asia/empeiria/empe-testnet-2_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.empe-chain/data
curl -L https://empeiria-services.luckystar.asia/empeiria/wasm_empe-testnet-2_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.empe-chain/wasm
```

```
mv $HOME/.empe-chain/priv_validator_state.json.backup $HOME/.empe-chain/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start emped.service && sudo journalctl -fu emped.service -o cat
```
