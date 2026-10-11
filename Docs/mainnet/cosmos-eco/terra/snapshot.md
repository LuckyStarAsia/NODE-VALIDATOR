# Snapshot

## Stop the service

```
sudo systemctl stop terrad.service
```

## Reset the data and save validator state

```
cp $HOME/.terra/data/priv_validator_state.json $HOME/.terrad/priv_validator_state.json.backup
rm -rf $HOME/.terra/data/*
```

## Download `latest` snapshot

```
curl -L https://terra-mainnet-services.luckystar.asia/terra/phoenix-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.sedad/data
```

```
mv $HOME/.terra/priv_validator_state.json.backup $HOME/.terra/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start terrad.service && sudo journalctl -fu terrad.service -o cat
```
