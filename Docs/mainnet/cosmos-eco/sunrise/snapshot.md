---
cover: ../../../.gitbook/assets/1500x500.jfif
coverY: 0
---

# Snap shot

## SNAPSHOT for Mainnet:

### Stop the service

```
sudo systemctl stop sunrised.service
```

### Reset the data and save validator state

```
cp $HOME/.sunrise/data/priv_validator_state.json $HOME/.sunrise/priv_validator_state.json.backup
rm -rf $HOME/.sunrise/data/*
```

### Download latest snapshot

```
curl -L https://sunrise-mainnet-services.luckystar.asia/sunrise/sunrise-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.sunrise/data
```

```
mv $HOME/.sunrise/priv_validator_state.json.backup $HOME/.sunrise/data/priv_validator_state.json
```

### Restart the service and check the log

```
sudo systemctl start sunrised.service && sudo journalctl -fu sunrised.service -o cat
```
