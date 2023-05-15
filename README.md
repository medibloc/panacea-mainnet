# Panacea Mainnet

This repo contains genesis files of all historical chains of the Panacea Mainnet.

The current the Panacea mainnet version is [v2.0.7-1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.6).

From v2.0.5, panacea supports state sync for rapid syncing with the network, thus we do not provide snapshot data anymore.

We recommend to set state sync enabled on your node. Please refer to our [guide](https://medibloc.gitbook.io/panacea-core/for-validators/join-mainnet-testnet#state-sync).

## Intro

- Version: Panacea Core [v2.0.7-1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.7-1)
- Chain ID: `panacea-3`
- Genesis file: https://github.com/medibloc/panacea-mainnet/raw/master/panacea-3/genesis.json.gz


## Persistent Peers

These public nodes below are operated by MediBloc with [PEX](https://docs.tendermint.com/master/spec/p2p/messages/pex.html) enabled.

You can add them to the `persistent_peers` in your `config.toml`. For more details, please see the [Tendermint document](https://docs.tendermint.com/v0.34/tendermint-core/using-tendermint.html#peers).

```
c238f279c970764d6893ae44bdf5c949dc22b009@13.114.44.199:26656
395aead00e99f828e4af92531dcd8c8da1255a8f@3.36.50.133:26656
00c57e36559b49ce7d29fa4920b5132584994368@52.77.227.241:26656
```


## Node Operations Guide

See the [Join the Network](https://docs.gopanacea.org/for-validators/3-join-mainnet-testnet) guide.


## Public Endpoints

Several public endpoints are provided by MediBloc.
**But, we highly recommend to run your own full node for high availability and reliability.**

- Cosmos REST: https://api.gopanacea.org
- Cosmos gRPC: https://grpc.gopanacea.org
- Tendermint RPC: https://rpc.gopanacea.org


## Explorer

You can use an [explorer](https://explorer.gopanacea.org/) based on Big-Dipper
