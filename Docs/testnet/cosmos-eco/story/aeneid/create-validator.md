---
cover: ../../../../.gitbook/assets/logo-dark.svg
coverY: 0
---

# Create-Validator

## Create validator

### Export private key

By default, when you run `story init` a `validator key` is created for you. To view `your validator key`, run the following command:

```
story validator export
```

In addition, if you want to export the derived `EVM private key` of your validator into the default data config directory, please run the following:

```
story validator export --export-evm-key
```

\***Important: Please keep your private key in a safe place**

Your EVM Private Key saved to: `/root/.story/story/config/private_key.txt`

Note that to participate in consensus, at least `1024 IP` must be staked (equivalent to `1024000000000000000000 wei` )!

Faucet link:

[https://cloud.google.com/application/web3/faucet/story/aeneid](https://cloud.google.com/application/web3/faucet/story/aeneid)

```
story validator create \
  --stake 1024000000000000000000 \
  --moniker "your moniker" \
  --rpc https://aeneid.storyrpc.io \
  --chain-id 1315
```

#### Backup the validator key:

File location: `/root/.story/story/config/priv_validator_key.json`

**Copy this file to your local machine.**

Store it carefully; this is the most crucial key for your validator.

## END
