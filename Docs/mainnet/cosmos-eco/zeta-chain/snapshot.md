# Snapshot

## Stop the service

```
sudo systemctl stop zetacored.service
```

## Reset the data and save validator state

```
cp $HOME/.zetacored/data/priv_validator_state.json $HOME/.zetacored/priv_validator_state.json.backup
rm -rf $HOME/.zetacored/data/*
```

## Download `latest` snapshot

```
curl -L https://zeta-mainnet-services.luckystar.asia/zeta/zetachain_7000-1_latest.tar.gz | tar -xzf - -C $HOME/.zetacored/data
```

```
mv $HOME/.zetacored/priv_validator_state.json.backup $HOME/.zetacored/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start zetacored.service && sudo journalctl -fu zetacored.service -o cat
```
