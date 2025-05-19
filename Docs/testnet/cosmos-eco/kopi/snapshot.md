# Snapshot

## Stop the service

```
sudo systemctl stop kopidtest.service
```

## Reset the data and save validator state

```
cp $HOME/.kopid/data/priv_validator_state.json $HOME/.kopid/priv_validator_state.json.backup
rm -rf $HOME/.kopid/data/*
rm -rf $HOME/.kopid/wasm/*
```

## Download `latest` snapshot

```
curl -L https://kopitest-services.luckystar.asia/kopitest/kopi-test-6_latest.tar.gz | tar -xzf - -C $HOME/.kopid/data
curl -L https://kopitest-services.luckystar.asia/kopitest/wasm_kopi-test-6_latest.tar.gz | tar -xzf - -C $HOME/.kopid/wasm
```

```
mv $HOME/.kopid/priv_validator_state.json.backup $HOME/.kopid/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start kopidtest.service && sudo journalctl -fu kopidtest.service -o cat
```
