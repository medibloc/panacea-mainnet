# Panacea v2.2.0 Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade Panacea daemons from v2.0.7-2 to v2.2.0.

- [Summary](#summary)
- [Changes](#changes)
- [Guides for Validators and Full-Node Operators](#guides-for-validators-and-full-node-operators)
    - [Rollback Strategy](#rollback-strategy)
- [Note for Service Providers](#note-for-service-providers)


## Summary

- SoftwareUpgrade Governance Proposal: https://www.mintscan.io/medibloc/proposals/16
- The upgrade will be started on the block height 15281100 (approximately, on Thursday March 21 2024, UTC 08:00), if the
  voting period of the SoftwareUpgrade proposal ends with the `passed` status.
    - See countdown [here](https://www.mintscan.io/medibloc/blocks/15281100).
- Upgrade type: Soft-fork
    - The chain ID will not be changed (will be maintained as `panacea-3`).
    - The block height will not be reset.

|                        | Before                                                                                          | After                                                                  |
|------------------------|-------------------------------------------------------------------------------------------------|------------------------------------------------------------------------|
| Panacea                | [v2.0.7-2](https://github.com/medibloc/panacea-core/releases/tag/v2.0.7-2)                      | [v2.2.0](https://github.com/medibloc/panacea-core/releases/tag/v2.2.0) |
| Cosmos SDK             | [v0.45.12-panacea.1](https://github.com/medibloc/cosmos-sdk/releases/tag/v0.45.12-panacea.1)    | [v0.47.10](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.47.10) |
| Tendermint to Cometbft | [v0.34.24-informalsystems](https://github.com/informalsystems/tendermint/releases/tag/v0.34.24) | [v0.37.4](https://github.com/cometbft/cometbft/releases/tag/v0.37.4)                                                            |


## Changes

- The Cosmos SDK version has been updated. Additionally, Tendermint has been replaced with Cometbft, which also includes a version update.
- NFT functionalities have been enabled.


## Guides for Validators and Full-Node Operators

**IMPORTANT** This version requires an upgrade to Go (version 1.22 or higher). 
Please install the latest version by following this [official guide](https://go.dev/doc/install).

If this SoftwareUpgrade proposal is approved, the state machine of the `panacead` will be stopped as soon as the chain
reaches to the block height 15281100.
It means that new blocks will not be produced until the `panacead` daemon is restarted with the new version.

When the state machine is stopped, you will be able to see the following error logs:

```
ERR UPGRADE "v2.2.0" NEEDED at height: 15281100:
ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.2.0\" NEEDED at height: 15281100: "
```

Then, stop your process.

```bash
pkill panacead

# or by other commands depending on your environment
```

After that, you should build a new binary as below. And, please replace the old `panacead` binary with the new one.

```bash
git clone https://github.com/medibloc/panacea-core
cd panacea-core
git checkout tags/v2.2.0
make install

panacead version
# should print 2.2.0
```

Prior to the upgrade, validators are encouraged to take a full data snapshot. Snapshotting depends heavily on
infrastructure, but generally this can be done by backing up the `~/.panacea` directory as below.<br>

**IMPORTANT**: It is critically important for validator operators to back-up
the `~/.panacea/data/priv_validator_state.json` file after stopping the `panacead` process. This file is updated every
block as your validator participates in consensus rounds. It is a critical file needed to prevent double-signing, in
case the upgrade fails and the previous chain needs to be restarted.

The `~/.panacea/config/priv_validator_key.json` file is also crucial, especially for validators. If this file is lost, validators must register anew as a validator.

Note: Before the upgrade, we plan to back up the state information of the last block height and store it on S3. However, as this process is time-consuming, we recommend node operators to back up the data themselves.

```bash
cp -R ~/.panacea ~/panacea-3-v2.0.7-backup

# or, to the external storage
```

Then, you can start the `panacead` again.

```bash
panacead start

# or by other commands depending on your environment
```

**NOTE**:
If you are using the [Cosmovisor](https://medibloc.gitbook.io/panacea-core/guide/cosmovisor) process manager, please
build the new `panacead` binary manually and put that under the `$HOME/.panacea/cosmovisor/upgrades/v2.2.0/bin/` by
following the [guide](https://medibloc.gitbook.io/panacea-core/guide/cosmovisor#cosmovisor-setup). We do not support
automatic downloads for security reasons.

### Rollback Strategy

In the event of an issue at upgrade time, we should coordinate via the validators channel in discord to come to a quick
emergency consensus and mitigate any further issues.

We will try to fix the issue by making a hotfix without rolling the chain back.
But, if there is no other way, we will roll the chain back to the v2.0.7-2 by following the steps below.

NOTE: We assume that you already backed up your `~/.panacea` directory before upgrading your node, as explained above.

1. Stop the process running.
    ```bash
    pkill panacead

    # or by other commands depending on your environment
    ```
2. Delete the new state, and recover the previous state from the backup that you made.
    ```bash
    rm -rf ~/.panacea/data
    cp -R ~/panacea-3-v2.0.7-backup/data ~/.panacea/data
    ```
3. Prepare the previous `panacead` v2.0.7-2 binary.
    ```bash
    git clone https://github.com/medibloc/panacea-core
    cd panacea-core
    git checkout tags/v2.0.7-2
    make install

    panacead version
    # should print v2.0.7-2
    ```
4. Start the process with skipping the upgrade which was registered by the SoftwareUpgrade proposal.
    ```bash
    panacead start --unsafe-skip-upgrades 15281100

    # or by other commands depending on your environment
    ```
5. Check if your node produces new blocks from the height 10196857 successfully, after 2/3+ of validators complete the
   rollback.
 

## Note for Service Providers

The service provider has introduced new features, requiring an upgrade to utilize these new functionalities. 
We provide the following SDKs for this purpose:

- For Java: panacea-java - [v2.2.0](https://github.com/medibloc/panacea-java/releases/tag/v2.2.0)
- For JavaScript: panacea-js - [v2.2.1](https://github.com/medibloc/panacea-js/releases/tag/v2.2.1)

