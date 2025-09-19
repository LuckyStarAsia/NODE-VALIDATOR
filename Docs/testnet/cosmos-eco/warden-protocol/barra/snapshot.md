---
description: Snapshot every 24h
---

# Snapshot

## Stop the service

```
sudo systemctl stop wardend.service
```

## Reset the data and save validator state

```
cp $HOME/.warden/data/priv_validator_state.json $HOME/.warden/priv_validator_state.json.backup
rm -rf $HOME/.warden/data/*
```

## Download `latest` snapshot

```
curl -L https://bwarden-testnet-services.luckystar.asia/bwarden/oro_1336-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.warden/data
mv $HOME/.warden/priv_validator_state.json.backup $HOME/.warden/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start wardend.service && sudo journalctl -fu wardend.service -o cat
```
