# Snapshot

## Stop the service

```
sudo systemctl stop sedad.service
```

## Reset the data and save validator state

```
cp $HOME/.sedad/data/priv_validator_state.json $HOME/.sedad/priv_validator_state.json.backup
rm -rf $HOME/.sedad/data/*
rm -rf $HOME/.sedad/wasm/*
```

## Download `latest` snapshot

```
curl -L https://seda-mainnet-services.luckystar.asia/seda/seda-1_latest.tar.gz | tar -xzf - -C $HOME/.sedad/data
curl -L https://seda-mainnet-services.luckystar.asia/seda/wasm_seda-1_latest.tar.gz | tar -xzf - -C $HOME/.sedad/wasm
```

```
mv $HOME/.sedad/priv_validator_state.json.backup $HOME/.sedad/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start sedad.service && sudo journalctl -fu sedad.service -o cat
```
