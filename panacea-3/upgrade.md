# `panacea-3` Upgrade Instructions

The following document describes the necessary steps involved that validators and
full node operators must take in order to upgrade from `panacea-2` to `panacea-3`.

Currently, all validator nodes are being operated by MediBloc.
So, MediBloc is going to upgrade all validator nodes on `August 11, 2021 at 04:00 UTC (13:00 KST)`.
After that, MediBloc will post an official `panacea-3` genesis file to the [panacea-launch](https://github.com/medibloc/panacea-launch/panacea-3/genesis.json) GitHub repo,
so that full node operators can upgrade their nodes based on it.


## Summary

|Chain ID|`panacea-2`|`panacea-3`|
|--------|-----------|-----------|
|Panacea|[v1.3.3](https://github.com/medibloc/panacea-core/releases/tag/v1.3.3)|v2.0.0 (TODO: to be released)|
|Cosmos SDK|[v0.37.14](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.37.14)|[v0.42.6](https://github.com/cosmos/cosmos-sdk/releases/tag/v0.42.6)|
|Tendermint|[v0.32.13](https://github.com/tendermint/tendermint/releases/tag/v0.32.13)|[v0.34.11](https://github.com/tendermint/tendermint/releases/tag/v0.34.11)|

Specific instructions for validators are available in [Guidance for Validators](upgrade-validator.md),
and specific instructions for full node operators are available in [Guidance for Full Node Operators](upgrade-fullnode.md).

Since the new Cosmos SDK contains many API breaking changes, Panacea client SDKs and REST API have been totally changed.
Please refer to the [Note for Service Providers](note-for-service-providers.md).


## Major Updates

Many changes have occurred to the Panacea, because the Cosmos SDK contains many new features and protocol breaking changes.
Many notable changes in the `panacea-3` can be found at the [CHANGELOG](https://github.com/medibloc/panacea-core/blob/master/CHANGELOG.md).

Some of the biggest changes are the following:

- **Protocol Buffers**: Initially, the Cosmos SDK used Amino codecs for nearly all encoding and decoding.
In this version, a major upgrade to Protocol Buffers have been integrated for better speed, readability, convenience and interoperability with many programming languages.
- **CLI**: `panacead` and `panaceacli` binaries have been merged into one `panacead` which now supports the `panaceacli` previously supported.
- **Node Configuration**: Previously, the blockchain data and configs were stored in `~/.panacead`. These files will now reside in `~/.panacea`.
  
Some of the biggest new features are the following:

- **Smart Contracts**: Panacea now provides smart contracts based on the [CosmWasm](https://cosmwasm.com/).
- **IBC**: TODO: to be described


## Guidance for Validators

[link](upgrade-validator.md)


## Guidance for Full Node Operators

[link](upgrade-fullnode.md)


## Note for Service Providers

[link](note-for-service-providers.md)