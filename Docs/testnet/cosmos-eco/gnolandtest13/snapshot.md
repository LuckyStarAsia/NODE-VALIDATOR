# Snap shot

## SNAPSHOT for Gnoland test-13:

### Stop the service

```
sudo systemctl stop gnolandtest.service
```

### Reset the data and save validator state

```
cp $HOME/gnoland-data/secrets/priv_validator_state.json $HOME/gnoland-data/priv_validator_state.json.backup
rm -rf $HOME/gnoland-data/db/db $HOME/gnoland-data/db/wal
```

### Download latest snapshot

```
curl -L https://snapshots.luckystar.asia/gnolandtest/gnoland_data.tar.zst && \
tar -I zstd -xf $HOME/gnoland_data.tar.zst -C $HOME/gnoland-data/
```

```
mv $HOME/gnoland-data/priv_validator_state.json.backup $HOME/gnoland-data/secrets/priv_validator_state.json
```

### Restart the service and check the log

```
sudo systemctl restart gnolandtest.service && sudo journalctl -fu gnolandtest.service -o cat
```
