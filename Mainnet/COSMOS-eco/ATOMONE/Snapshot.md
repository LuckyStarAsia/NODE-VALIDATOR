# Snap shot

## SNAPSHOT for Mainnet:

### Stop the service

```
sudo systemctl stop atomoned.service
```

### Reset the data and save validator state

```
cp $HOME/.atomone/data/priv_validator_state.json $HOME/.atomone/priv_validator_state.json.backup
rm -rf $HOME/.atomone/data/*
```

### Download latest snapshot

```
curl -L https://atomone-mainnet-services.luckystar.asia/atomonemainnet/atomone-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.atomone/data
```

```
mv $HOME/.atomone/priv_validator_state.json.backup $HOME/.atomone/data/priv_validator_state.json
```

### Restart the service and check the log

```
sudo systemctl start atomoned.service && sudo journalctl -fu atomoned.service -o cat
```
