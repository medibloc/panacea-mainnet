# Panacea Mainnet

This repo contains genesis files of all historical chains of the Panacea Mainnet.


## Intro

- Version: Panacea Core [v2.0.1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.1)
- Chain ID: `panacea-3`
- Genesis file: https://github.com/medibloc/panacea-mainnet/raw/master/panacea-3/genesis.json.gz


## Persistent Peers

These public nodes below are operated by MediBloc with [PEX](https://docs.tendermint.com/master/spec/p2p/messages/pex.html) enabled.

You can add them to the `persistent_peers` in your `config.toml`. For more details, please see the [Tendermint document](https://docs.tendermint.com/master/tendermint-core/using-tendermint.html#peers).

```
e93f5df69cc1b9bda230b3efcf162d4672293ded@3.35.82.40:26656
0e0db1c7ab1e37c76f2ceced0bb72e1c60ef17a6@13.124.96.254:26656
a83232db3a40e486b54b1463a43431e8490b808b@52.79.108.35:26656
```

**These 3 nodes below will be deprecated soon.**
If you connect to these nodes, please change them to the new nodes above.
```
8c41cc8a6fc59f05138ae6c16a9eec05d601ef71@13.209.177.91:26656
cc0285c4d9cec8489f8bfed0a749dd8636406a0d@54.180.169.37:26656
1fc4a41660986ee22106445b67444ec094221e76@52.78.132.151:26656
```


## Node Operations Guide

See the [Join the Network](https://medibloc.gitbook.io/panacea-core/guide/join-the-network) guide.


## Public Endpoints

Several public endpoints are provided by MediBloc.
**But, we highly recommend to run your own full node for high availability and reliability.**

- Cosmos REST: https://api.gopanacea.org
- Cosmos gRPC: https://grpc.gopanacea.org
- Tendermint RPC: https://rpc.gopanacea.org


## Explorer

You can use an [explorer](https://explorer.gopanacea.org/) based on Big-Dipper
