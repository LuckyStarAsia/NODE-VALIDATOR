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
curl -L https://snapshots.luckystar.asia/terra/terra_data.tar.zst && \
tar -I zstd -xf $HOME/terra_data.tar.zst -C $HOME/.terra/
```

```
mv $HOME/.terra/priv_validator_state.json.backup $HOME/.terra/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start terrad.service && sudo journalctl -fu terrad.service -o cat
```
