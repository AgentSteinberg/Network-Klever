# Klever Claiming

## About 

When you stake or delegate you KLV, you will earn addition KLV as incentive. The actually get those KLV you need to "claim" thos.

## Claiming

You can claim rewards either from Staking (with no delegation) or Allowance (from delegation):

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/delegate-wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \ 
    account claim <CLAIM_TYPE> --id=KLV
```

For the `<CLAIM_TYPE>` you need to set:

* `0` = StakingClaim
* `1` = AllowanceClaim

