# Note for Service Providers

There are an unusual amount of API breakage between the Cosmos SDK `v0.37.14` and `v0.42.6`.
Because Panacea is based on Cosmos SDK, the service providers should update their source code
according to the new Cosmos SDK REST API or Panacea client SDKs.


## For Java SDK users

Please refer to the [guide](https://github.com/medibloc/panacea-java) in the `panacea-java` repo.

TODO: use the specific link.


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


## For REST API users

Please refer to the [Migrating to New REST Endpoints](https://docs.cosmos.network/v0.42/migrations/rest.html#migrating-to-new-rest-endpoints) guide of the Cosmos SDK.