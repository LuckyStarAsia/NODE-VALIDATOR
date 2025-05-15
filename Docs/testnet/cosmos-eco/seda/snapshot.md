# Snapshot

## Stop the service

```
sudo systemctl stop sedadtest.service
```

## Reset the data and save validator state

```
cp $HOME/.sedad/data/priv_validator_state.json $HOME/.sedad/priv_validator_state.json.backup
rm -rf $HOME/.sedad/data/*
rm -rf $HOME/.sedad/wasm/*
```

## Download `latest` snapshot

```
curl -L https://seda-testnet-services.luckystar.asia/sedatest/seda-1-testnet_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.sedad/data
curl -L https://seda-testnet-services.luckystar.asia/sedatest/wasm_seda-1-testnet_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.sedad/wasm
```

```
mv $HOME/.sedad/priv_validator_state.json.backup $HOME/.sedad/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start sedadtest.service && sudo journalctl -fu sedadtest.service -o cat
```
