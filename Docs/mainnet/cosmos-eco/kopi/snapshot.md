# Snapshot

## Stop the service

```
sudo systemctl stop kopid.service
```

## Reset the data and save validator state

```
cp $HOME/.kopid/data/priv_validator_state.json $HOME/.kopid/priv_validator_state.json.backup
rm -rf $HOME/.kopid/data/*
rm -rf $HOME/.kopid/wasm/*
```

## Download `latest` snapshot

```
curl -L https://kopi-services.luckystar.asia/kopi/luwak-1_latest.tar.gz | tar -xzf - -C $HOME/.kopid/data
curl -L https://kopi-services.luckystar.asia/kopi/wasm_luwak-1_latest.tar.gz | tar -xzf - -C $HOME/.kopid/wasm
```

```
mv $HOME/.kopid/priv_validator_state.json.backup $HOME/.kopid/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start kopid.service && sudo journalctl -fu kopid.service -o cat
```
