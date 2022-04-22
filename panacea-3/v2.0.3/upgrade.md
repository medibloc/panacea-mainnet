# Panacea v2.0.3 Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade Panacea daemons from v2.0.2 to v2.0.3.


## Summary

- Governance Proposals
	- Signaling proposal: https://www.mintscan.io/medibloc/proposals/7
	- SoftwareUpgrade proposal: https://www.mintscan.io/medibloc/proposals/8
- The upgrade will be started after the block time `2022-05-11 07:00:00 UTC` if the voting period of the SoftwareUpgrade proposal ends with the `passed` status.
- Upgrade type: Soft-fork
    - The chain ID will not be changed (will be maintained as `panacea-3`).
    - The block height will not be reset.
- No API breaking changes

| |Before|After|
|--------|-----------|-----------|
|Panacea|[v2.0.2](https://github.com/medibloc/panacea-core/releases/tag/v2.0.2)|[v2.0.3](https://github.com/medibloc/panacea-core/releases/tag/v2.0.3)|
|Cosmos SDK|[v0.42.9](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.42.9)|[v0.42.11-panacea.1](https://github.com/medibloc/cosmos-sdk/releases/tag/v0.42.11-panacea.1)|
|Tendermint|[v0.34.11](https://github.com/tendermint/tendermint/releases/tag/v0.34.11)|[v0.34.14](https://github.com/tendermint/tendermint/releases/tag/v0.34.14)|


## Changes

This upgrade introduces a new staking parameter `min_commission_rate` and set it to 3%.

For more details, please see a release note: https://github.com/medibloc/panacea-core/releases/tag/v2.0.3.


## Guides for Validators and Full-Node Operators

If this SoftwareUpgrade proposal is approved, the state machine of the `panacead` will be stopped as soon as a proposed block time '2022-05-11 07:00:00 UTC' is reached.
It means that new blocks will not be produced until the `panacead` daemon is restarted with the new version.

When the state machine is stopped, you will be able to see the following error logs:
```
ERR UPGRADE "v2.0.3" NEEDED at time: 2022-05-11T07:00:00Z:
ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.0.3\" NEEDED at time: 2022-05-11T07:00:00Z: "
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
git checkout v2.0.3
make install

panacead version
```

(Recommended) Before starting the new `panacead` process, please back up the `~/.panacea` directory just in case.
```bash
cp -R ~/.panacea ~/panacea-3-v2.0.2-backup

# or, to the external storage
```

Then, you can start the `panacead` again.
```bash
panacead start

# or by other commands depending on your environment
```

NOTE:
If you are using the [cosmovisor](https://docs.cosmos.network/master/run-node/cosmovisor) process manager, please build the new `panacead` binary manually and put that under the [upgrade](https://docs.cosmos.network/master/run-node/cosmovisor.html#folder-layout) directory. The [auto-download](https://docs.cosmos.network/master/run-node/cosmovisor.html#auto-download) is not supported yet because the appropriate version of the [libwasmvm.so](https://github.com/CosmWasm/wasmvm/blob/v0.14.0/api/libwasmvm.so) must be installed as well. Instead of installing the `libwasmvm.so` seperately, I would recommend you build the `panacea-core` by the guide above. Then, the `libwasmvm.so` will be installed automatically.


## Note for Service Providers

Service providers do not need to change their source code which communicates with Panacea nodes, because this upgrade does not contain any API breaking changes.
