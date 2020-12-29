# `panacea-2` Upgrade Instructions (for non-validators)

- [Preliminary](#preliminary)
- [Major Updates](#major-updates)
- [Expected Errors](#expected-errors)
- [Notes for Service Providers](#notes-for-service-providers)


## Preliminary

The following document describes the necessary steps involved that full-node operators (**that are non-validators**) must take in order to upgrade from the chain `panacea-1` to `panacea-2`.

Currently, all validator nodes are being operated by MediBloc. So, MediBloc is going to upgrade all validator nodes on `January 5, 2021 at or around 14:00 UTC` (`23:00 KST`). After that, MediBloc will provide the new `genesis.json` with all full-node operators, so that they can upgrade their nodes based on it.

It will take about 2 hours conservatively to upgrade all validators. Thus, I would like to ask you to block deposits and withdrawals for about 6 hours, including the duration you upgrade your nodes and clients.


## Major Updates 

- From [v1.2.5](https://github.com/medibloc/panacea-core/releases/tag/v1.2.5) to [v1.3.3](https://github.com/medibloc/panacea-core/releases/tag/v1.3.3)
- [DID](https://github.com/medibloc/panacea-core/blob/v1.3.3/docs/did.md) operations have been added.
- Cosmos SDK has been updated from [v0.35.0](https://github.com/cosmos/cosmos-sdk/blob/master/CHANGELOG.md#0350) to [v0.37.14](https://github.com/cosmos/cosmos-sdk/blob/master/CHANGELOG.md#v03714---2020-08-12).
  - **NOTE:** This change contains an unusual amount of REST API breakage from Cosmos SDK. Please update your client code by following the [Notes for Service Providers](#notes-for-service-providers).


## Expected Errors

After MediBloc upgrades all validators, before you upgrade your nodes, you may get the following error logs from the `panacead`.
```
Connection failed @ recvRoutine (reading byte) module=p2p peer=f9dfd5d962d38f1650e67bd1dbde48eb45ba8381@13.125.218.17:26656 conn=MConn{13.125.218.17:26656} err=EOF

Stopping peer for error                      module=p2p peer="Peer{MConn{13.125.218.17:26656} f9dfd5d962d38f1650e67bd1dbde48eb45ba8381 out}" err=EOF

dialing failed (attempts: 1): dial tcp 13.125.218.17:26656: connect: connection refused module=pex addr=f9dfd5d962d38f1650e67bd1dbde48eb45ba8381@13.125.218.17:26656

...

dialing failed (attempts: 9): incompatible: Peer is on a different network. Got panacea-2, expected panacea-1 module=pex addr=f9dfd5d962d38f1650e67bd1dbde48eb45ba8381@13.125.218.17:26656
```

These errors are normal, and they will be resolved once you upgrade your nodes by following the guide below.

Until then, the Panacea REST API server that you are running will work only for read operations.


## Upgrade Procedure

*NOTE:* It is assumed you are currently operating a full-node running Panacea `v1.2.5`.

- The commit hash of Panacea `v1.3.3`: [2dfceffc647db15499c715e8833c5e379c04e028](https://github.com/medibloc/panacea-core/releases/tag/v1.3.3)
- The height will be reset to `0` after the upgrade.
  - All data will be preserved.

1. Verify you are currently runing the correct version of Panacea (`v1.2.5`).
    ```bash
    $ panacead version --long
    panacea-core: 1.2.5
    git commit: 7aeecf3c413a87bf2f733efe14b66bcf6758b1f4
    go.sum hash:
    build tags:  ledger
    go version go1.15.3 darwin/amd64
    ```

2. Stop Panacea processes
    ```bash
    # Stop the REST API server
    $ pkill panaceacli

    # Stop the Panacea daemon
    $ pkill panacead
    ```

3. Checkout and build the new Panacea `v1.3.3`.

    **NOTE:** Go [1.13+](https://golang.org/dl/) is required.
    ```bash
    $ git clone https://github.com/medibloc/panacea-core.git
    $ cd panacea-core
    $ git checkout v1.3.3
    $ make install 
    ```

4. Verify you have the correct version of Panacea (`v1.3.3`).
    ```bash
    $ panacead version --long
    name: panacea-core
    server_name: panacead
    client_name: panaceacli
    version: 1.3.3
    commit: 2dfceffc647db15499c715e8833c5e379c04e028
    build_tags: ' ledger'
    go: go version go1.15.5 linux/amd64
    ```

5. Download the new `genesis.json`.
    ```bash
    # Backup the origin genesis file
    $ cp ~/.panacead/config/genesis.json ~/genesis.orig.json

    # Download the new one
    $ curl -o ~/.panacead/config/genesis.json https://raw.githubusercontent.com/medibloc/panacea-launch/master/panacea-2/genesis.json
    ```

6. Reset state
    ```bash
    $ panacead unsafe-reset-all
    ```

7. Make sure that the `db_backend` on `~/.panacead/config/config.toml` is `goleveldb`.
    ```bash
    db_backend = "goleveldb"
    ```

8. Start Panacea processes
    ```bash
    # Start the Panacea daemon
    $ panacead start

    # Start the REST API server. Note that the chain ID must be panacea-2.
    $ panaceacli rest-server --laddr tcp://0.0.0.0:1317 --chain-id panacea-2
    # If you have run the REST API server without the --chain-id option,
    # please modify the chain-id in the '~/.panaceacli/config/config.toml'.
    ```


## Notes for Service Providers

There are an unusual amount of API breakage between Cosmos SDK `v0.35.0` and `v0.37.14`.
Because Panacea is based on Cosmos SDK, the service providers should update their source code according to the new Cosmos SDK REST API.

HTTP request formats have not  been changed. Only HTTP response formats have been changed.

In short, there are two big changes:
1. Anyone running signing infrastructure(wallets and exchanges) should be conscious that the type: field on StdTx will have changed from `"type":"auth/StdTx","value":...` to `"type":"cosmos-sdk/StdTx","value":...`.
2. As mentioned in the Cosmos SDK [CHANGELOG](https://github.com/cosmos/cosmos-sdk/blob/master/CHANGELOG.md), most queries are wrapped with `height` fields now.

For details, please see the following sections. If you are using other APIs which are not listed in the following sections, please contact us.

### GET `/node_info`

Before:
```json
{
    "id": "...",
    "listen_addr": "...",
    ...
}
```
After:
```json
{
    "node_info": {
        "id": "...",
        "listen_addr": "...",
        ...
    },
    "application_version": {
        "name": "panacea-core",
        "version": "x.x.x",
        ...
    }
}
```

### GET `/txs/{tx_hash}`

Before:
```json
{
    "height": "1",
    "txHash": "....",
    "tx": {
        "type": "auth/StdTx",
        ...
    },
    ...
}
```
After:
```json
{
    "height": "1",
    "txHash": "....",
    "tx": {
        "type": "cosmos-sdk/StdTx",
        ...
    },
    ...
}
```

### GET `/txs?tx.height={height}`

Before:
```json
[
    {
        "height": 1,
        "txHash": "....",
        ...
    }
]
```
After:
```json
{
    "txs": [
        {
            "height": 1,
            "txHash": "....",
            ...
        }
    ]
}
```

### GET `/auth/accounts/{address}`

Before:
```json
{
    "type": "auth/Account",
    "value": {
        "address": "...",
        "account_number": "10",
        ...
    }
}
```
After:
```json
{
    "height": "12345",
    "result": {
        "type": "cosmos-sdk/Account",
        "value": {
            "address": "...",
            "account_number": "10",
            ...
        }
    }
}
```

### GET `/bank/balances/{address}`

Before:
```json
[
  {
    "denom": "umed",
    "amount": "29990000000"
  }
]
```
After:
```json
{
  "height": "840450",
  "result": [
    {
      "denom": "umed",
      "amount": "29990000000"
    }
  ]
}
```

### GET `/staking/validators`

Before:
```json
[
    {
       "operator_address": "..",
        ... 
    }
]
```
After:
```json
{
    "height": "123",
    "result": [
        {
           "operator_address": "..",
            ... 
        }
    ]
}
```

### GET `/staking/pool`

Before:
```json
{
  "not_bonded_tokens": "8342091269079211",
  "bonded_tokens": "21000000000000"
}
```
After:
```json
{
    "height": "123",
    "result": {
      "not_bonded_tokens": "8342091269079211",
      "bonded_tokens": "21000000000000"
    }
}
```
