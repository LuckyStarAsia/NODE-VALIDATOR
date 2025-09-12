# Snapshot

## Stop the service

```
sudo systemctl stop gaiad.service
```

## Reset the data and save validator state

```
cp $HOME/.gaia/data/priv_validator_state.json $HOME/.gaia/priv_validator_state.json.backup
rm -rf $HOME/.gaia/data/*
rm -rf $HOME/.gaia/wasm/*
```

## Download `latest` snapshot

```
curl -L https://gaia-services.luckystar.asia/gaia/cosmoshub-4_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.gaia/data
curl -L https://gaia-services.luckystar.asia/gaia/wasm_cosmoshub-4_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.gaia/wasm
```

```
mv $HOME/.gaia/priv_validator_state.json.backup $HOME/.gaia/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start gaiad.service && sudo journalctl -fu gaiad.service -o cat
```
