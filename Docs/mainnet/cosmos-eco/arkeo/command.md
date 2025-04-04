---
cover: ../../../.gitbook/assets/arkeo-top-bg.jpeg
coverY: 0
---

# Command

![ARKEO Network](../../../.gitbook/assets/ARKEO.jpg)

## Arkeo CLI Cheatsheet

### Bank:

#### Send

```
arkeod tx bank send <KEY RECEIVER_ADDRESS> 1000000uarkeo \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 --fees 500uarkeo --gas auto \
  --from $WALLET
```

### Distribution:

#### Withdraw Rewards including Commission

```
arkeod tx distribution withdraw-rewards VALIDATOR_OPERATOR \
  --commission \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 --fees 500uarkeo --gas auto \
  --from $WALLET
```

### Gov:

#### Query Proposal

```
arkeod query gov proposal PROPOSAL_NUMBER \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 \
  --output json | jq
```

#### Vote

VOTE\_OTION: `yes, no, no_with_veto and abstain.`

```
arkeod tx gov vote PROPOSAL_NUMBER VOTE_OPTION \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 --fees 500uarkeo --gas auto \
  --from $WALLET
```

### Slashing:

#### Unjail

```
arkeod tx slashing unjail \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 --fees 500uarkeo --gas auto \
  --from $WALLET
```

### Staking:

#### Create Validator

Note: replace your validator info

```
arkeod tx staking create-validator \
  --amount 100000000uarkeo \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.10" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(arkeod tendermint show-validator) \
  --moniker 'your-moniker' \
  --website "your-website" \
  --identity "your-identity" \
  --details "your-detail" \
  --security-contact="your-contact" \
  --chain-id $ARKEO_CHAIN_ID \
  --node https://arkeo-mainnet-rpc.luckystar.asia:443 --fees 500uarkeo --gas auto \
  --from $WALLET
```
