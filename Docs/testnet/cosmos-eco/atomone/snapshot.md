# Snap shot

## SNAPSHOT for TESTNET:

### Stop the service

```
sudo systemctl stop atomonetestd.service
```

### Reset the data and save validator state

```
cp $HOME/.atomone/data/priv_validator_state.json $HOME/.atomone/priv_validator_state.json.backup
rm -rf $HOME/.atomone/data/*
```

### Download latest snapshot

```
curl -L https://atomone-testnet-services.luckystar.asia/atomone-testnet/atomone-testnet-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.atomone/data
```

```
mv $HOME/.atomone/priv_validator_state.json.backup $HOME/.atomone/data/priv_validator_state.json
```

### Restart the service and check the log

```
sudo systemctl start atomonetestd.service && sudo journalctl -fu atomonetestd.service -o cat
```
