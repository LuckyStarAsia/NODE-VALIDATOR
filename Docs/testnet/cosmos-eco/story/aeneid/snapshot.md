---
cover: ../../../../.gitbook/assets/logo-dark.svg
coverY: 0
---

# Snapshot

## Stop the service

```
sudo systemctl stop story.service story-geth.service
```

## Reset the data and save validator state

```
cp $HOME/.story/data/priv_validator_state.json $HOME/.sedad/priv_validator_state.json.backup
rm -rf $HOME/.story/story/data/*
rm -rf $HOME/.story/geth/aeneid/geth/chaindata/*
mkdir -p $HOME/.story/geth/aeneid/geth/chaindata
```

## Download `latest` snapshot

```
curl https://story-testnet-services.luckystar.asia/story/aeneid_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/story/data
curl https://story-testnet-services.luckystar.asia/story/geth_aeneid_latest.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.story/geth/aeneid/geth/chaindata
```

```
mv $HOME/.story/story/priv_validator_state.json.backup $HOME/.story/story/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl restart story-geth && sleep 5 && sudo systemctl restart story
sudo journalctl -u story -u story-geth -f
```
