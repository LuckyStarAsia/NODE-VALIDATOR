
![image](https://github.com/user-attachments/assets/9763ee83-ea45-42b7-a3ed-e2edbc16f921)


# Althea CLI Cheatsheet

## Bank: 

### Send
```
althea tx bank send <KEY RECEIVER_ADDRESS> 1000000aalthea \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000aalthea --gas 300000 \
  --from wallet
```

## Distribution: 

### Withdraw Rewards including Commission
```
althea tx distribution withdraw-rewards VALIDATOR_OPERATOR \
  --commission \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000aalthea --gas 300000 \
  --from wallet
```

## Gov: 
### Query Proposal
```
althea query gov proposal PROPOSAL_NUMBER \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 \
  --output json | jq
```

### Vote
VOTE_OTION: `yes, no, no_with_veto and abstain.`
```
althea tx gov vote PROPOSAL_NUMBER VOTE_OPTION \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000aalthea --gas 300000 \
  --from wallet
```

## Slashing: 

### Unjail
```
althea tx slashing unjail \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000aalthea --gas 300000 \
  --from wallet
```

## Staking: 

### Create Validator
Note: replace your validator info
```
althea tx staking create-validator \
  --amount 1000000000000000000aalthea \
  --commission-max-change-rate "0.05" \
  --commission-max-rate "0.10" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey=$(althea tendermint show-validator) \
  --moniker 'your-moniker' \
  --website "your-website" \
  --identity "your-identity" \
  --details "your-detail" \
  --security-contact="your-contact" \
  --chain-id althea_258432-1 \
  --node https://althea-mainnet-rpc.luckystar.asia:443 --fees 30000000000000000aalthea --gas 300000 \
  --from wallet
```


