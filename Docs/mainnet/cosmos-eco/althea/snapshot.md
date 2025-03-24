---
cover: ../../../.gitbook/assets/AltheaCover.JPG
coverY: 0
---

# Snapshot

## ALTHEA Network

## Stop the service

```
sudo systemctl stop althead.service
```

## Reset the data and save validator state

```
cp $HOME/.althea/data/priv_validator_state.json $HOME/.althea/priv_validator_state.json.backup
rm -rf $HOME/.althea/data/*
```

## Download `latest` snapshot

```
curl -L https://althea-mainnet-services.luckystar.asia/althea/althea_258432-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.althea/data
```

```
mv $HOME/.althea/priv_validator_state.json.backup $HOME/.althea/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start althead.service && sudo journalctl -fu althead.service -o cat
```
