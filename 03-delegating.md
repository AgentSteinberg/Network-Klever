# Klever Delegating

## About 

Delegating Klever ist similar to staking.
At first it is necessary to freeze a specific amount of KLV. After that you can delegate the frozen Klever to another Validator by using
the transaction id.

Note: All delegations are made from a seconds wallet, which is stored in the `delegate-wallet` directory.

## Delegating

### Freeze the KLV

Let's start with freezing 1000 KLV (which is the minimum amount).

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/delegate-wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance account freeze 1000
```    

### Get the BucketId

Use the transaction id of the freeze transaction to get the <BUCKET_ID>.

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/delegate-wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    tx-by-id \
    <FREEZE_ID>
```

### Delegate the frozen KLV

Delegate the frozen KLV to the selected <ACCOUNT> (e.g. `klv186vg5k3pqetmfuy04620htcvz3krugu7hqe4ukczdy48r222j78q9y8vm5`).
Hint: To find accounts to delegate to, you want to check the validator list in the explorer: https://testnet.kleverscan.org/validators

```
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/delegate-wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    account delegate \
    <ACCOUNT> \
    --bucketID=<BUCKET_ID>
```

The delegated amount can be checked in the transaction list.
        
Note: To redelegate, you just need to keep the same Bucket ID and type a new address to whom the delegation will be done.
