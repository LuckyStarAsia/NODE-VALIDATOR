---
description: Intento
---

# Snapshot

![Intento](../../../.gitbook/assets/into.png)

## Stop the service

```
sudo systemctl stop intentod.service
```

## Reset the data and save validator state

```
cp $HOME/.intento/data/priv_validator_state.json $HOME/.intento/priv_validator_state.json.backup
rm -rf $HOME/.intento/data/*
rm -rf $HOME/.intento/wasm/*
```

## Download `latest` snapshot

```
curl -L https://intento-mainnet-services.luckystar.asia/intento/intento-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.intento/data && \
curl -L https://intento-mainnet-services.luckystar.asia/intento/wasm_intento-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.intento/wasm 
```

```
mv $HOME/.intento/priv_validator_state.json.backup $HOME/.intento/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start intentod.service && sudo journalctl -fu intentod.service -o cat
```
