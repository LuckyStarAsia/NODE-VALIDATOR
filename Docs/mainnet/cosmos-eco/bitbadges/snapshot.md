---
description: Bitbadges
---

# Snapshot

![Bitbadges](../../../.gitbook/assets/badge_logo.png)

## Stop the service

```
sudo systemctl stop bitbadgeschaind.service
```

## Reset the data and save validator state

```
cp $HOME/.bitbadgeschain/data/priv_validator_state.json $HOME/.bitbadgeschain/priv_validator_state.json.backup
rm -rf $HOME/.bitbadgeschain/data/*
rm -rf $HOME/.bitbadgeschain/wasm/*
```

## Download `latest` snapshot

```
curl -L https://bitbadges-mainnet-services.luckystar.asia/bitbadges/bitbadges-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.bitbadgeschain/data && \
curl -L https://bitbadges-mainnet-services.luckystar.asia/bitbadges/wasm_bitbadges-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.bitbadgeschain/wasm 
```

```
mv $HOME/.bitbadgeschain/priv_validator_state.json.backup $HOME/.bitbadgeschain/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start bitbadgeschaind.service && sudo journalctl -fu bitbadgeschaind.service -o cat
```
