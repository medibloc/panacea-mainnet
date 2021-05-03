# `panacea-3` Upgrade Instructions (for non-validators)


## Abstraction

- From [v1.3.3](https://github.com/medibloc/panacea-core/releases/tag/v1.3.3) to v1.3.4 (TO BE RELEASE SOON)
	- New features that support more DID Document properties
	- Genesis parameter updates for the validator joining


## Preliminary

The following document describes the necessary steps involved that full-node operators (**that are non-validators**) must take in order to upgrade from the chain `panacea-2` to `panacea-3`.

Currently, all validator nodes are being operated by MediBloc. So, MediBloc is going to upgrade all validator nodes on `June 8th, 2021 at or around 02:00 UTC` (`11:00 KST`). After that, MediBloc will provide the new `genesis.json` with all full-node operators, so that they can upgrade their nodes based on it.

It will take about 2 hours conservatively to upgrade all validators. Thus, I would like to ask you to block deposits and withdrawals for about 6 hours, including the duration you upgrade your nodes and clients.


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

*NOTE:* It is assumed you are currently operating a full-node running Panacea `v1.3.3`.

- The commit hash of Panacea `v1.3.4`: TO BE RELEASED SOON
- The height will be reset to `0` after the upgrade.
  - All data will be preserved.

1. Verify you are currently runing the correct version of Panacea (`v1.3.3`).
    ```bash
    $ panacead version --long
    panacea-core: 1.3.3
    git commit: 2dfceffc647db15499c715e8833c5e379c04e028
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

3. Checkout and build the new Panacea `v1.3.4`.
    ```bash
    $ git clone https://github.com/medibloc/panacea-core.git
    $ cd panacea-core
    $ git checkout v1.3.4
    $ make install 
    ```

4. Verify you have the correct version of Panacea (`v1.3.4`).
	- TO BE DESCRIBED

5. Download the new `genesis.json`.
    ```bash
    # Backup the origin genesis file
    $ cp ~/.panacead/config/genesis.json ~/genesis.orig.json

    # Download the new one
    $ curl -o ~/.panacead/config/genesis.json https://raw.githubusercontent.com/medibloc/panacea-launch/master/panacea-3/genesis.json
    ```

6. Reset state
    ```bash
    $ panacead unsafe-reset-all
    ```

7. Start Panacea processes
    ```bash
    # Start the Panacea daemon
    $ panacead start

    # Start the REST API server. Note that the chain ID must be panacea-3.
    $ panaceacli rest-server --laddr tcp://0.0.0.0:1317 --chain-id panacea-3
    # If you have run the REST API server without the --chain-id option,
    # please modify the chain-id in the '~/.panaceacli/config/config.toml'.
    ```
