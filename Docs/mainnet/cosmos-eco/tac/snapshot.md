---
cover: ../../../.gitbook/assets/681860db654f71faabe0ffc4_tac roadmap.png
coverY: 0
---

# Snapshot

## TAC Chain

## Stop the service

```
sudo systemctl stop tacchaind.service
```

## Reset the data and save validator state

```
cp $HOME/.tacchaind/data/priv_validator_state.json $HOME/.tacchaind/priv_validator_state.json.backup
rm -rf $HOME/.tacchaind/data/*
```

## Download `latest` snapshot

```
curl -L https://tac-mainnet-services.luckystar.asia/tac/tacchain_239-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.tacchaind/data
```

```
mv $HOME/.tacchaind/priv_validator_state.json.backup $HOME/.tacchaind/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start tacchaind.service && sudo journalctl -fu tacchaind.service -o cat
```
