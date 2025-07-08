---
cover: >-
  ../../../.gitbook/assets/Copy-of-Black-white-Minimal-Corporate-Report-Cover-Page-Twitter-Header-1024x341.webp
coverY: 0
---

# Snapshot

## Lumera protocol

## Stop the service

```
sudo systemctl stop lumerad.service
```

## Reset the data and save validator state

```
cp $HOME/.lumera/data/priv_validator_state.json $HOME/.lumera/priv_validator_state.json.backup
rm -rf $HOME/.lumera/data/*
rm -rf $HOME/.lumera/wasm/*
```

## Download `latest` snapshot

```
curl -L https://lumera-mainnet-services.luckystar.asia/lumera/lumera-mainnet-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.lumera/data
curl -L https://lumera-mainnet-services.luckystar.asia/lumera/wasm_lumera-mainnet-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.lumera/wasm
```

```
mv $HOME/.lumera/priv_validator_state.json.backup $HOME/.lumera/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start lumerad.service && sudo journalctl -fu lumerad.service -o cat
```
