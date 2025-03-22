# Snapshot

## Airchains | varanasi-1

<figure><img src="../../../.gitbook/assets/Airchains.jpg" alt="" width="200"><figcaption><p>Airchains</p></figcaption></figure>

## Stop the service

```
sudo systemctl stop airchains2test.service
```

## Reset the data and save validator state

```
cp $HOME/.junctiond/data/priv_validator_state.json $HOME/.junctiond/priv_validator_state.json.backup
rm -rf $HOME/.junctiond/data/*
rm -rf $HOME/.junctiond/wasm/*
```

## Download `latest` snapshot

```
curl -L https://airchains2-services.luckystar.asia/airchains2/varanasi-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.junctiond/data
curl -L https://airchains2-services.luckystar.asia/airchains2/wasm_varanasi-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.junctiond/wasm
mv $HOME/.junctiond/priv_validator_state.json.backup $HOME/.junctiond/data/priv_validator_state.json
```

## Restart the service and check the log

```
sudo systemctl start airchains2test.service && sudo journalctl -fu airchains2test.service -o cat
```
