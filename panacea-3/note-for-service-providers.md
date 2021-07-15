# Note for Service Providers

There are an unusual amount of API breakage between the Cosmos SDK `v0.37.14` and `v0.42.7`.
Because Panacea is based on Cosmos SDK, the service providers should update their source code
according to the new Cosmos SDK REST API or Panacea client SDKs.


## For Java SDK users

The `panacea-java` [v2.0.0](https://github.com/medibloc/panacea-java/releases/tag/v2.0.0) has been released.

The SDK interface has been changed because the underlying Cosmos SDK contains many breaking changes including the protocol migration from Amino to Protobuf.
But, we have tried to design the new SDK interface as similar as possible to the previous one.

Whereas the previous `panacea-java` v1.x communicates with the Panacea REST API, the v2.x communicates with the [Panacea gRPC endpoint](https://docs.cosmos.network/master/run-node/interact-node.html#using-grpc) (default port: `9090`). 
For more details, please refer to the [examples](https://github.com/medibloc/panacea-java#feature).


## For Javascript SDK users

The new Panacea core is compatible with [CosmJS](https://github.com/cosmos/cosmjs) (tested with CosmJS v0.25.4).
Please refer to the [CosmJS guide](https://gist.github.com/webmaster128/8444d42a7eceeda2544c8a59fbd7e1d9) about how to communicate with Cosmos-based blockchains.
```ts
import { DirectSecp256k1HdWallet } from "@cosmjs/proto-signing";
import { stringToPath } from "@cosmjs/crypto";

const mnemonic =
  "surround thank ...";
const wallet = await DirectSecp256k1HdWallet.fromMnemonic(
  mnemonic,
  {
    hdPaths: [stringToPath("m/44'/371'/0'/0/0")],
    prefix: "panacea",
  },
);
const [firstAccount] = await wallet.getAccounts();
```

If you want to use Panacea-specific transactions such as AOL or DID, please use the [panacea-js](https://github.com/medibloc/panacea-js) which wraps CosmJS.

Whereas the previous `panacea-js` v1.x communicates with the Panacea REST API, but the v2.x communicates with the [Tendermint RPC endpoint](https://docs.cosmos.network/master/core/grpc_rest.html#tendermint-rpc) (default port: `26657`).


## For REST API users

Panacea provides the [Cosmos REST API](https://docs.cosmos.network/master/run-node/interact-node.html#using-the-rest-endpoints) as it is (default port: `1317`).
However, numerous breaking changes have been made in the Cosmos REST API.
Please refer to the [Migrating to New REST Endpoints](https://docs.cosmos.network/v0.42/migrations/rest.html#migrating-to-new-rest-endpoints) guide of the Cosmos SDK.
