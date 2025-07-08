# Snapshot

## Stop the service

```
sudo systemctl stop exrpd.service
```

## Reset the data and save validator state

```
cp $HOME/.exrpd/data/priv_validator_state.json $HOME/.exrpd/priv_validator_state.json.backup
rm -rf $HOME/.exrpd/data/*
```

## Download `latest` snapshot

```
curl -L https://xrpl-mainnet-services.luckystar.asia/xrpl/xrplevm_1440000-1_latest.tar.gz | tar -xzf - -C $HOME/.exrpd/data
```

```
mv $HOME/.exrpd/priv_validator_state.json.backup $HOME/.exrpd/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start exrpd.service && sudo journalctl -fu exrpd.service -o cat
```
