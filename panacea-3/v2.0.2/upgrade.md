# Panacea v2.0.2 Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade Panacea daemons from v2.0.1 to v2.0.2.


## Summary

- Governance Proposal: https://www.mintscan.io/medibloc/proposals/2
- The upgrade will be started after the block time `2021-10-01 07:00:00 UTC` if the voting period ends with the `passed` status.
- Upgrade type: Soft-fork
    - The chain ID will not be changed (will be maintained as `panacea-3`).
    - The block height will not be reset.
- No API breaking changes

| |Before|After|
|--------|-----------|-----------|
|Panacea|[v2.0.1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.1)|[v2.0.2](https://github.com/medibloc/panacea-core/releases/tag/v2.0.2)|
|Cosmos SDK|[v0.42.7](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.42.7)|[v0.42.9](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.42.9)|
|Tendermint|[v0.34.11](https://github.com/tendermint/tendermint/releases/tag/v0.34.11)|Same as before|


## Changes

The Cosmos team released the Cosmos SDK v0.42.9 which fixes a bug that prohibits IBC to create new channels 
(issue [#9800](https://github.com/cosmos/cosmos-sdk/issues/9800)) on Cosmos SDK v0.42.7 and v0.42.8.
Therefore, it is required to upgrade the Cosmos SDK to v0.42.9 in order to enable IBC on Panacea.

For more details, please see the v2.0.2 release note: https://github.com/medibloc/panacea-core/releases/tag/v2.0.2.


## Guides for Validators and Full-Node Operators

Since this upgrade will be proposed as a Governance Proposal on Panacea, the state machine of your `panacead` process will be stopped automatically as soon as it finalize a block which has the block time larger than `2021-10-01T07:00:00Z`. Then, no more blocks will not be created until the process is restarted with the new version.

You will be able to see the following error logs:
```
ERR UPGRADE "v2.0.2" NEEDED at time: 2021-10-01T07:00:00Z:
ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.0.2\" NEEDED at time: 2021-10-01T07:00:00Z: "
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
git checkout v2.0.2
make install

panacead version
```

(Recommended) Before starting the new `panacead` process, please back up the `~/.panacea` directory just in case.
```bash
cp -R ~/.panacea ~/panacea-3-v2.0.1-backup

# or, to the external storage
```

Then, you can start the `panacead` again.
```bash
panacead start

# or by other commands depending on your environment
```

NOTE:
If you are using the `cosmovisor`, please build the new `panacead` binary manually and put that under the `upgrade` directory. The auto-download is not supported yet because the appropriate version of the `libwasmvm.so` must be installed as well.


## Note for Service Providers

Service providers do not need to change their source code which communicates with Panacea nodes, because this upgrade
does not contain any API breaking changes.
