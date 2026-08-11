# Panacea v2.3.0 Mainnet Upgrade

> [!WARNING]
> This is a draft upgrade guide. The proposal, upgrade height, schedule, and
> release artifacts are not final. Do not use it for mainnet operations yet.

This guide is for validators and full-node operators upgrading `panacea-3`
from v2.2.1 to v2.3.0.

## Summary

- Governance proposal: **TBD**
- Upgrade height: **TBD**
- Estimated time: **TBD** (block times vary; monitor the chain)
- Upgrade plan name: `v2.3.0`
- Release commit: **TBD**
- Chain ID: `panacea-3` (unchanged)

| Component | Before | After |
| --- | --- | --- |
| Panacea | v2.2.1 | v2.3.0 |
| Cosmos SDK | v0.47.10 | v0.50.15 |
| CometBFT | v0.37.18 | v0.38.23 |
| IBC-Go | v7.3.2 | v8.8.0 |

The release adds the Panacea NFT module, retires the legacy PNFT runtime, and
changes node-local configuration and CLI behavior. Read the
[v2.3.0 release notes](https://github.com/medibloc/panacea-core/releases/tag/v2.3.0)
before voting or upgrading.

## Before waiting for the upgrade height

Complete this section while v2.2.1 is running and fully synced. When Cosmovisor
is used, `PANACEA_HOME` below must be the same directory configured as
`DAEMON_HOME`. Stage the binary, back up the active configuration, and migrate
`app.toml` before waiting for the upgrade height:

```text
$PANACEA_HOME/cosmovisor/upgrades/v2.3.0/bin/panacead  # staged binary
$PANACEA_HOME/config/app.toml                          # active configuration
```

### 1. Prepare the binary

Download the official binary for your platform from the v2.3.0 release and
verify its published checksum.

```sh
export PANACEA_HOME="${PANACEA_HOME:-$HOME/.panacea}"
export PANACEAD_V230=/path/to/panacead-v2.3.0

"$PANACEAD_V230" version --long
```

The version must be `2.3.0`. Source builds require Go 1.26.5.

If you use Cosmovisor, stage the binary in advance:

```sh
mkdir -p "$PANACEA_HOME/cosmovisor/upgrades/v2.3.0/bin"
cp "$PANACEAD_V230" \
  "$PANACEA_HOME/cosmovisor/upgrades/v2.3.0/bin/panacead"
chmod +x "$PANACEA_HOME/cosmovisor/upgrades/v2.3.0/bin/panacead"

export PANACEAD_V230="$PANACEA_HOME/cosmovisor/upgrades/v2.3.0/bin/panacead"
"$PANACEAD_V230" version --long
```

Keep Cosmovisor automatic binary downloads disabled.

### 2. Back up the node

Follow your normal node backup procedure. Validator operators must preserve the
latest `data/priv_validator_state.json` and `config/priv_validator_key.json` to
avoid double-signing or losing the validator key.

Back up and restore the node data and validator signing state together from the
same recovery point.

Back up the local configuration before changing it:

```sh
cp -p "$PANACEA_HOME/config/app.toml" \
  "$PANACEA_HOME/config/app.toml.pre-v2.3.0"
cp -p "$PANACEA_HOME/config/client.toml" \
  "$PANACEA_HOME/config/client.toml.pre-v2.3.0"
cp -p "$PANACEA_HOME/config/config.toml" \
  "$PANACEA_HOME/config/config.toml.pre-v2.3.0"
```

### 3. Migrate `app.toml`

Generate a v0.50 preview without changing the active configuration:

```sh
"$PANACEAD_V230" config migrate v0.50 \
  --home "$PANACEA_HOME" \
  --stdout >"$PANACEA_HOME/config/app.toml.v0.50.preview"
diff -u \
  "$PANACEA_HOME/config/app.toml" \
  "$PANACEA_HOME/config/app.toml.v0.50.preview"
```

The migration updates only `$PANACEA_HOME/config/app.toml`. It does not modify
`client.toml` or CometBFT `config.toml`; keep those files and review
`config.toml` separately.

The preview diff is expected to add settings introduced in v0.50 and remove
obsolete settings. Before applying the migration, review any removed or changed
values and confirm that the newly added defaults are appropriate for the node.

After reviewing the diff, apply the migration and validate the result:

```sh
"$PANACEAD_V230" config migrate v0.50 --home "$PANACEA_HOME"
"$PANACEAD_V230" config view app --home "$PANACEA_HOME" >/dev/null
```

Confirm that the application DB resolves to `goleveldb`:

```sh
grep -nE '^(db_backend|app-db-backend) =' \
  "$PANACEA_HOME/config/config.toml" \
  "$PANACEA_HOME/config/app.toml"
```

The expected result is `db_backend = "goleveldb"` with either
`app-db-backend = ""` or `app-db-backend = "goleveldb"`.

Do not start v2.3.0 with `cleveldb`, `boltdb`, or `badgerdb` selected for the
application database.

`client.toml` remains in place, but automation that reads or writes it must use
the new command form: `panacead config get client <key>` or
`panacead config set client <key> <value>`.

### 4. Query endpoint compatibility

`--grpc-web.address` is not supported by v2.3.0. Remove it from the node service
command. Cosmovisor users must restart v2.2.1 before the upgrade so that the
automatic binary switch does not reuse it. If the node serves gRPC-Web, update
and verify its client or proxy route before that restart.

Nodes that do not expose public REST, gRPC, gRPC-Web, or application-query
endpoints can skip the rest of this section. These changes do not affect
consensus.

In v2.2.1, gRPC-Web uses a separate listener configured by
`grpc-web.address` (default `localhost:9091`). In v2.3.0, REST and gRPC-Web
share `[api].address` (default `tcp://localhost:1317`), while native gRPC
continues to use `[grpc].address` (default `localhost:9090`). Serving gRPC-Web
requires `[api].enable`, `[grpc].enable`, and `[grpc-web].enable` to be `true`.

Update clients to the API address, or keep the client-facing address stable
with a reverse proxy and switch its upstream at the upgrade height. Configure
browser CORS under `[api].enabled-unsafe-cors` or at the proxy.

Apply the Panacea serving defaults before waiting for the upgrade. The query gas
limit takes effect with v2.3.0; the API and gRPC limits are also understood by
v2.2.1.

```sh
"$PANACEAD_V230" config set app query-gas-limit 10000000 \
  --home "$PANACEA_HOME"
"$PANACEAD_V230" config set app api.rpc-write-timeout 10 \
  --home "$PANACEA_HOME"
"$PANACEAD_V230" config set app grpc.max-send-msg-size 10485760 \
  --home "$PANACEA_HOME"
```

## At the upgrade height

If the proposal passes, the v2.2.1 binary stops at the upgrade height.
Cosmovisor should switch to the staged `v2.3.0` binary automatically. Without
Cosmovisor, stop the service, replace the binary, and start the service again.

Do not use `--unsafe-skip-upgrades` unless validators have explicitly agreed on
a coordinated recovery plan.

## Verify

After restart, confirm:

```sh
"$PANACEAD_V230" version --long
"$PANACEAD_V230" status --node tcp://127.0.0.1:26657
```

- the reported version is `2.3.0`, `catching_up` is `false`, and block height
  continues to increase;
- logs contain no store migration, module migration, or consensus errors;
- REST, gRPC, gRPC-Web, RPC, and IBC relayers work where enabled.

## Recovery

If the chain does not resume, stop making independent changes and coordinate in
the official validator channel. Binary or chain-state rollback must be agreed
by validators; restoring only a local configuration backup is safe before the
v2.3.0 process starts.
