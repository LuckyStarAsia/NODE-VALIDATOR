# Validator

## VALIDATOR:

#### Run your own validator

Show PubKey:

```
gaiad comet show-validator
```

OR:

```
gaiad tendermint show-validator
```

We assume operators understand how to create keys using the CLI. Below is how to register your validator to the Kopi blockchain. First you need to create a text file contain your validator’s configufation:

```
cd $HOME && \
nano $HOME/.gaia/config/validator.json
```

```
{
  "pubkey": {
    "@type":"/cosmos.crypto.ed25519.PubKey",
    "key":"<replace with value from:priv_validator_key.json>"
  },
  "amount": "1000000ukopi",
  "moniker": "YOUR_MONKIER",
  "identity": "YOUR_KEYBASE_ID",
  "website": "YOUR_WEBSITE_URL",
  "security": "YOUR_EMAIL_ADDRESS",
  "details": "Wiv-Sugar",
  "commission-rate": "0.1",
  "commission-max-rate": "0.2",
  "commission-max-change-rate": "0.01",
  "min-self-delegation": "1"
}
```

#### Then create your validator by passing the created text file:

```
gaiad tx staking create-validator $HOME/.gaia/config/validator.json \
  --from $WALLET \
  --chain-id $GAIA_CHAIN_ID \
  -y
```

## END VALIDATOR
