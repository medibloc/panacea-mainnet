# Panacea Mainnet

This repo contains genesis files of all historical chains of the Panacea Mainnet.

The current the Panacea mainnet version is [v2.2.0-1](https://github.com/medibloc/panacea-core/releases/tag/v2.2.0-1).

From v2.0.5, panacea supports state sync for rapid syncing with the network, thus we do not provide snapshot data anymore.

We recommend to set state sync enabled on your node. Please refer to our [guide](https://docs.gopanacea.org/guide/join-the-network#state-sync).

## Intro

- Version: Panacea Core [v2.2.0-1](https://github.com/medibloc/panacea-core/releases/tag/v2.2.0-1)
- Chain ID: `panacea-3`
- Genesis file: https://github.com/medibloc/panacea-mainnet/raw/master/panacea-3/genesis.json.gz


## Persistent Peers

These public nodes below are operated by MediBloc with [PEX](https://github.com/cometbft/cometbft/blob/main/spec/p2p/implementation/pex.md) enabled.

You can add them to the `persistent_peers` in your `config.toml`. For more details, please see the [Tendermint document](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#peers).

```
395aead00e99f828e4af92531dcd8c8da1255a8f@3.36.50.133:26656
0e030ab48abc25ff2918ab019fd74d5447d7582e@15.165.127.151:26656
f808eb775180345c3b443d55afcc3c148dd19183@3.37.237.120:26656
```


## Node Operations Guide

See the [Join the Network](https://docs.gopanacea.org/guide/join-the-network) guide.


## Public Endpoints

Several public endpoints are provided by MediBloc.
**But, we highly recommend to run your own full node for high availability and reliability.**

- Cosmos REST: https://api.gopanacea.org
- Cosmos gRPC: https://grpc.gopanacea.org
- Tendermint RPC: https://rpc.gopanacea.org


## Explorer

You can use [Mintscan](https://www.mintscan.io/medibloc), provided by Cosmostation, for exploring the blockchain. 
Additionally, as a supplementary tool, you have the option to use an [explorer](https://explorer.gopanacea.org) based on Callisto.