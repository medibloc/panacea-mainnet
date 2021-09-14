# Panacea v2.0.2 Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade Panacea daemons from v2.0.1 to v2.0.2.


## Summary

- Governance Proposal: TBD
- The upgrade will be started on `TBD` (`October 1, 2021` probably).
  - Block height: `TBD`
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

For more details, please see the v2.0.2 release note (TBD).


## Guides for Validators and Full-Node Operators

Since this upgrade will be proposed as a Governance Proposal on Panacea, your `panacead start` process will be shut down
automatically at the time that was proposed on the Governance Proposal (TBD).
When the process is stopped, it will print the following error logs:
```
ERR UPGRADE "v2.0.2" NEEDED at height: <TBD>:
ERR CONSENSUS FAILURE!!! err="UPGRADE \"v2.0.2\" NEEDED at height: <TBD>: "
```

After that, you should replace the old `panacead` binary with the new one.

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
```


## Note for Service Providers

Service providers do not need to change their source code which communicates with Panacea nodes, because this upgrade
does not contain any API breaking changes.
