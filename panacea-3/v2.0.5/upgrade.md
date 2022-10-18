# Panacea v2.0.5 Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade Panacea daemons from v2.0.3 to v2.0.5.


## Summary

- SoftwareUpgrade Governance Proposal: TODO: link
- The upgrade will be started after the block time `2022-10-24 07:00:00 UTC` if the voting period of the SoftwareUpgrade proposal ends with the `passed` status.
- Upgrade type: Soft-fork
    - The chain ID will not be changed (will be maintained as `panacea-3`).
    - The block height will not be reset.

| |Before|After|
|--------|-----------|-----------|
|Panacea|[v2.0.3](https://github.com/medibloc/panacea-core/releases/tag/v2.0.3)|[v2.0.5](https://github.com/medibloc/panacea-core/releases/tag/v2.0.5)|
|Cosmos SDK|[v0.42.11-panacea.1](https://github.com/medibloc/cosmos-sdk/releases/tag/v0.42.11-panacea.1)|[v0.45.9-panacea.1](https://github.com/medibloc/cosmos-sdk/releases/tag/v0.45.9-panacea.1)|
|Tendermint|[v0.34.14](https://github.com/tendermint/tendermint/releases/tag/v0.34.14)|[v0.34.21](https://github.com/tendermint/tendermint/releases/tag/v0.34.21)|


## Changes

- Upgrades Cosmos SDK from v0.42.11 to v0.45.9 which contains many new features, bug fixes, and especially a [IBC security fix](https://forum.cosmos.network/t/ibc-security-advisory-dragonberry/7702).
- Removes the `x/token` module.

## Guides for Validators and Full-Node Operators

If this SoftwareUpgrade proposal is approved, the state machine of the `panacead` will be stopped as soon as a proposed block time `2022-10-24 07:00:00 UTC` is reached.
It means that new blocks will not be produced until the `panacead` daemon is restarted with the new version.

When the state machine is stopped, you will be able to see the following error logs:
```
ERR UPGRADE "v2.0.5" NEEDED at time: 2022-10-24T07:00:00Z:
ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.0.5\" NEEDED at time: 2022-10-24T07:00:00Z: "
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
git checkout v2.0.5
make install

panacead version
# should print v2.0.5
```

(Recommended) Before starting the new `panacead` process, please back up the `~/.panacea` directory just in case.
```bash
cp -R ~/.panacea ~/panacea-3-v2.0.3-backup

# or, to the external storage
```

Then, you can start the `panacead` again.
```bash
panacead start

# or by other commands depending on your environment
```

NOTE:
If you are using the [Cosmovisor](https://medibloc.gitbook.io/panacea-core/guide/cosmovisor) process manager, please build the new `panacead` binary manually and put that under the `$HOME/.panacea/cosmovisor/upgrades/v2.0.5/bin/` by following the [guide](https://medibloc.gitbook.io/panacea-core/guide/cosmovisor#cosmovisor-setup). The [auto-download](https://github.com/cosmos/cosmos-sdk/tree/main/cosmovisor#auto-download) is not supported yet because the appropriate version of the [libwasmvm.so](https://github.com/CosmWasm/wasmvm/blob/v0.14.0/api/libwasmvm.so) must be installed as well. Instead of installing the `libwasmvm.so` separately, I would recommend you build the `panacea-core` by the guide above. Then, the `libwasmvm.so` will be installed automatically.

### Rollback Strategy

In the event of an issue at upgrade time, we should coordinate via the validators channel in discord to come to a quick emergency consensus and mitigate any further issues.

We will try to fix the issue by making a hotfix without rolling the chain back.
But, if there is no other way, we will roll the chain back to the v2.0.3 by following the steps below.

0. We assume that you already backed up your `~/.panacea` directory before upgrading your node, as explained above. For example:
    ```bash
    cp -R ~/.panacea ~/panacea-3-v2.0.3-bak
    ```
1. Stop the process running.
    ```bash
    pkill panacead

    # or by other commands depending on your environment
    ```
2. Delete the new state, and recover the previous state from the backup that you made.
    ```bash
    rm -rf ~/.panacea/data
    cp -R ~/panacea-3-v2.0.3-bak/data ~/.panacea/data
    ```
3. Prepare the previous `panacead` v2.0.3 binary.
    ```bash
    git clone https://github.com/medibloc/panacea-core
    cd panacea-core
    git checkout tags/v2.0.3
    make install

    panacead version
    # should print v2.0.3
    ```
4. Start the process with skipping the upgrade which was registered by the SoftwareUpgrade proposal.
    ```bash
    panacead start --unsafe-skip-upgrades <upgrade-height>

    # or by other commands depending on your environment
    ```
5. Check if your node produces new blocks successfully, after 2/3+ of validators complete the rollback.


## Note for Service Providers

If you are using the latest version of [panacea-java](https://github.com/medibloc/panacea-java) or [panacea-js](https://github.com/medibloc/panacea-js) library, you don't need to change the library that you are using. There is no new release related to this chain upgrade.

However, there are some API breaking changes from the Cosmos SDK side. You can find all API breaking changes between v0.42.x and v0.45.9 from the [Cosmos SDK change log](https://github.com/cosmos/cosmos-sdk/blob/v0.45.9/CHANGELOG.md).