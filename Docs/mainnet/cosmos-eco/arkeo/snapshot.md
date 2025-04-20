---
cover: ../../../.gitbook/assets/arkeo-top-bg.jpeg
coverY: 0
---

# Snapshot

## ARKEO Network

## Stop the service

```
sudo systemctl stop arkeod.service
```

## Reset the data and save validator state

```
cp $HOME/.arkeo/data/priv_validator_state.json $HOME/.arkeo/priv_validator_state.json.backup
rm -rf $HOME/.arkeo/data/*
```

## Download `latest` snapshot

```
curl -L https://arkeo-mainnet-services.luckystar.asia/arkeo/arkeo-main-v1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.arkeo/data
```

```
mv $HOME/.arkeo/priv_validator_state.json.backup $HOME/.arkeo/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start arkeod.service && sudo journalctl -fu arkeod.service -o cat
```
