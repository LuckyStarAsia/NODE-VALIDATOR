---
cover: ../../../.gitbook/assets/681860db654f71faabe0ffc4_tac roadmap.png
coverY: 0
---

# Command

![TAC Chain](../../../.gitbook/assets/tac.jpg)

## TAC chain CLI Cheatsheet

### Bank:

#### Send

```
tacchaind tx bank send <KEY RECEIVER_ADDRESS> 1000000utac \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000utac --gas 300000 \
  --from $WALLET
```

### Distribution:

#### Withdraw Rewards including Commission

```
tacchaind tx distribution withdraw-rewards VALIDATOR_OPERATOR \
  --commission \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000utac --gas 300000 \
  --from $WALLET
```

### Gov:

#### Query Proposal

```
tacchaind query gov proposal PROPOSAL_NUMBER \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 \
  --output json | jq
```

#### Vote

VOTE\_OTION: `yes, no, no_with_veto and abstain.`

```
tacchaind  tx gov vote PROPOSAL_NUMBER VOTE_OPTION \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000utac --gas 300000 \
  --from $WALLET
```

### Slashing:

#### Unjail

```
tacchaind tx slashing unjail \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000utac --gas 300000 \
  --from $WALLET
```

### Staking:

#### Create Validator

Note: replace your validator info

```
tacchaind tx staking create-validator \
  --amount 1000000000000000000utac \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.10" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(tacchaind tendermint show-validator) \
  --moniker 'your-moniker' \
  --website "your-website" \
  --identity "your-identity" \
  --details "your-detail" \
  --security-contact="your-contact" \
  --chain-id $TAC_CHAIN_ID \
  --node http://tac-mainnet-rpc.luckystar.asia:443 --fees 10000000000000000utac --gas 300000 \
  --from $WALLET
```
