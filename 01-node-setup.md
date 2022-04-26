# Klever Node Setup

## About

This guideline holds instructions on how to install a Validator for the [Klever Blockchain](https://klever.finance/kleverchain/).
The guide is based on the [official documentation](https://docs.klever.finance/klever-blockchain/how-to-run-a-node) but adds additional hints and instructions on the setup.

## Prerequisite

Hardware Requirements:

* at least 4 CPU cores
* at least 8 GB of RAM
* at least 200 GB storage
* at least 100 MBit internet speed

Although, the Klever node is able to run on MacOS and a wide range of linux distributions, this guide recommends the usage of **Ubuntu 20.04** as the operating system.

## Install

### Prepare the host Operating System

1. Get a Virtual Private Server (VPS) with Ubuntu 20.04 as operating system.
2. Make sure to update and secure your VPS. A good guideline, that was originally written by my buddy cryptonaut gordonfreeman can be found at <https://github.com/AgentSteinberg/VPS-First-Setup-Ubuntu>.
3. Install Docker, which is needed to run the Klever node software. Follow instructions from <https://docs.docker.com/engine/install/>.
4. Configure Docker to manage Docker as a non-root user as described at <https://docs.docker.com/engine/install/linux-postinstall/>.

### Install the Klever Node

Installation and update of the Klever node is done as a docker image via:

```shell
docker pull kleverapp/klever-go-testnet:latest
```

### Setup a Wallet

```shell
mkdir wallet
```

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest account create
```

### Run the Node

```shell
mkdir -p $(pwd)/node/config $(pwd)/node/db $(pwd)/node/logs
```

```shell
curl -k https://backup.testnet.klever.finance/config.testnet.100009.tar.gz \
    | tar -xz -C ./node
```

```shell
docker run -it --rm -v $(pwd)/node/config:/opt/klever-blockchain \
    --user "$(id -u):$(id -g)" \
    --entrypoint='' kleverapp/klever-go-testnet:latest keygenerator
```

```shell
curl -k https://backup.testnet.klever.finance/kleverchain.latest.tar.gz \
    | tar -xz -C ./node
```

```shell
docker run -it --rm \
    --user "$(id -u):$(id -g)" \
    --name klever-node \
    -v $(pwd)/node/config:/opt/klever-blockchain/config/node \
    -v $(pwd)/node/db:/opt/klever-blockchain/db \
    -v $(pwd)/node/logs:/opt/klever-blockchain/logs \
    --network=host \
    --entrypoint=/usr/local/bin/validator \
    kleverapp/klever-go-testnet:latest \
    '--log-save' '--rest-api-interface=0.0.0.0:8080'
```

You should see an ASCII dashboard in you console if the installation was successfull.
Cancel the Node with `CTRL+C` and start the node again, but this time let it run in the background. Even if you close the console.

```shell
docker run -it -d --restart unless-stopped \
    --user "$(id -u):$(id -g)" \
    --name klever-node \
    -v $(pwd)/node/config:/opt/klever-blockchain/config/node \
    -v $(pwd)/node/db:/opt/klever-blockchain/db \
    -v $(pwd)/node/logs:/opt/klever-blockchain/logs \
    --network=host \
    --entrypoint=/usr/local/bin/validator \
    kleverapp/klever-go-testnet:latest \
    '--log-save' '--use-log-view' '--rest-api-interface=0.0.0.0:8080'
```

### Becoming a Validator

In order to run a validator you need a certain amount of KLV

* 1,500,000 KLV as initiale stake
* 100,000 KLV which get burned during activation of your validator

For the Testnet you can get the TKLV via registration at the testnet program at <https://klever.finance/klever-blockchain-validators-program/>

Once you got the necessary (T)KLV you can proceed with the next step. To check if you received the TKLV you can run

```shell
docker run -it --rm --user "$(id -u):$(id -g)" -v $(pwd)/wallet:/opt/klever-blockchain --entrypoint=/usr/local/bin/operator kleverapp/klever-go-testnet:latest account balance
```

or check your account in the [Klever Explorer](https://testnet.kleverscan.org/accounts).

### Request Validator

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    validator create \
    <YOUR_ACCOUNT> \
    --bls=<YOUR BLS> \
    --rewards=<YOUR_ACCOUNT> \
    --name=<YOUR_VALIDATOR_NAME> \
    --logo="https://klever.finance/wp-content/uploads/2021/09/logo.svg" \
    --commission=5 \
    --maxDelegation=12000000 \
    --uris="github=github.com/klever-io" \
    --canDelegate=true
```

You need to replace:

* `<YOUR_ACCOUNT>` with your wallet key
* `<YOUR_BLS>` with your bls key from `validatorkey.pem` file you created above
* `<YOUR_VALIDATOR_NAME>` with the desired name of your validator

### Freeze KLV for staking

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance account freeze 1500000
```

### Get the BucketId

Use the transaction id of the freeze transaction to get the `<BUCKET_ID>`.

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    tx-by-id \
    <FREEZE_ID>
```

### Delegate frozen KLV to node

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    account delegate \
    klv186vg5k3pqetmfuy04620htcvz3krugu7hqe4ukczdy48r222j78q9y8vm5 \
    --bucketID=<BUCKET_ID>
```

Hint: You can find the `<BUCKET_ID>` also in the transaction using the [Klever Explorer](https://testnet.kleverscan.org/accounts).

Done! You are now successfully staking with your node and producing blocks.

Hint for the testnet: This is where you need to wait for additional delegation of 8.5M TKLV to be send to you by the Klever admins.







---

## Command Reference

Here are some important commands for reference

### How to check wallet certificate and address

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest account address
```

### How to show logs

```shell
docker logs -f --tail 5 klever-node
```

### How to stop a node

```shell
docker stop klever-node
```

### How to backup node data

Stop node first.

```shell
tar -czvf filename.tar.gz $(pwd)/node/db
```

### How to restart the node

```shell
docker restart klever-node
```

### How to backup node data

Stop node first.

```shell
docker pull kleverapp/klever-go-testnet:latest
```

```shell
docker rm klever-node
```

### How to edit the validator settings once created

```shell
docker run -it --rm --user "$(id -u):$(id -g)" \
    -v $(pwd)/wallet:/opt/klever-blockchain \
    --network=host \
    --entrypoint=/usr/local/bin/operator \
    kleverapp/klever-go-testnet:latest \
    --key-file=./walletKey.pem \
    --node=https://node.testnet.klever.finance \
    validator config \
    <YOUR_ACCOUNT> \
    --bls=<YOUR_BLS> \
    --rewards=<YOUR_ACCOUNT> \
    --name=MyValidatorName \
    --logo="https://klever.finance/wp-content/uploads/2021/09/logo.svg" \
    --commission=5 \
    --maxDelegation=12000000 \
    --uris="github=github.com/klever-io"
```

