# Snap shot

## SNAPSHOT for Mainnet:

### Stop the service

```
sudo systemctl stop axoned.service
```

### Reset the data and save validator state

<pre><code>cp $HOME/.axoned/data/priv_validator_state.json $HOME/.axoned/priv_validator_state.json.backup
rm -rf $HOME/.axoned/data/*
<strong>rm -rf $HOME/.axoned/wasm/*
</strong></code></pre>

### Download latest snapshot

```
curl -L https://axone-mainnet-services.luckystar.asia/axone/axone-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.axoned/data
curl -L https://axone-mainnet-services.luckystar.asia/axone/wasm_axone-1_latest.tar.lz4 | lz4 -d - | tar -xf - -C $HOME/.axoned/wasm
```

```
mv $HOME/.axoned/priv_validator_state.json.backup $HOME/.axoned/data/priv_validator_state.json
```

### Restart the service and check the log

```
sudo systemctl start axoned.service && sudo journalctl -fu axoned.service -o cat
```
