# Snapshot

## Stop the service

```
sudo systemctl stop bitwayd.service
```

## Reset the data and save validator state

```
cp $HOME/.bitway/data/priv_validator_state.json $HOME/.bitway/priv_validator_state.json.backup
rm -rf $HOME/.bitway/data/*
rm -rf $HOME/.bitway/wasm/*
```

## Download `latest` snapshot

```
curl -L https://bitway-mainnet-services.luckystar.asia/bitway/bitway-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.bitway/data
curl -L https://bitway-mainnet-services.luckystar.asia/bitway/wasm_bitway-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.bitway/wasm
```

```
mv $HOME/.bitway/priv_validator_state.json.backup $HOME/.bitway/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start bitwayd.service && sudo journalctl -fu bitwayd.service -o cat
```
