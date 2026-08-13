# Panacea v2.3.0 Mainnet Upgrade

This guide covers upgrading a node running v2.2.0 or v2.2.1 on `panacea-3`,
using either Cosmovisor or a manual restart. The commands assume that the
Panacea home is `$HOME/.panacea`. Replace that path if your node uses a
different home.

## Release

- Release: https://github.com/medibloc/panacea-core/releases/tag/v2.3.0
- Commit: `91c74f66aaeb0b2fc37282175eee400d0767e37f`
- AMD64 SHA256: `833c2d3c7dca815692e2050f2180a65736669c654965dae89175982c3139d499`
- ARM64 SHA256: `c2f7d6dfd97fe2091abe3d5a61eeef4930c4ca1bc5d7e3d9c8156c320283bfa6`
- Upgrade height: `28,074,600`
- Estimated time: `2026-08-20 16:00 KST` (`07:00 UTC`)
- Countdown: [Mintscan](https://www.mintscan.io/medibloc/block/28074600)

The estimated time and countdown may shift with block production. The upgrade
will occur at the specified block height.

## Upgrade overview

This upgrade requires more than a binary restart: it includes a configuration
migration and narrows the supported application database backends to
`goleveldb`.

Before the upgrade height, verify the release binary, back up the essential
files, and migrate `app.toml` to the SDK v0.50 format. Most importantly,
confirm that `db_backend` is `goleveldb` and `app-db-backend` is either empty
or `goleveldb`. Then prepare the verified binary for either Cosmovisor or a
manual restart at the approved height.

## What changes in v2.3.0

- Upgrades Cosmos SDK to v0.50.15, CometBFT to v0.38.23, and IBC-Go to
  v8.8.0.
- Adds the v2.3.0 upgrade handler and Panacea NFT module while preserving
  support for legacy AOL and DID signing.
- Moves gRPC-Web to the API listener, updates the client configuration
  commands, and removes application DB support for backends other than
  `goleveldb`.

## Before the upgrade

### 1. Check the node architecture

```sh
uname -m
```

Follow the section that matches your node architecture:

| `uname -m` output | Use |
| --- | --- |
| `x86_64` or `amd64` | AMD64 |
| `aarch64` or `arm64` | ARM64 |

### 2. Download and verify v2.3.0

The final `cp` command creates the common `panacead` path used by the remaining
steps.

#### AMD64

```sh
mkdir -p $HOME/.panacea/releases/v2.3.0
cd $HOME/.panacea/releases/v2.3.0

curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-amd64
curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-amd64.sha256

sha256sum -c panacead-linux-amd64.sha256

chmod +x panacead-linux-amd64
./panacead-linux-amd64 version --long | grep -E '^(version|commit):'

cp -p panacead-linux-amd64 panacead
```

The checksum verification must report `panacead-linux-amd64: OK`. The version
output must contain `version: 2.3.0` and commit
`91c74f66aaeb0b2fc37282175eee400d0767e37f`.

#### ARM64

```sh
mkdir -p $HOME/.panacea/releases/v2.3.0
cd $HOME/.panacea/releases/v2.3.0

curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-arm64
curl -fL -O https://github.com/medibloc/panacea-core/releases/download/v2.3.0/panacead-linux-arm64.sha256

sha256sum -c panacead-linux-arm64.sha256

chmod +x panacead-linux-arm64
./panacead-linux-arm64 version --long | grep -E '^(version|commit):'

cp -p panacead-linux-arm64 panacead
```

The checksum verification must report `panacead-linux-arm64: OK`. The version
output must contain `version: 2.3.0` and commit
`91c74f66aaeb0b2fc37282175eee400d0767e37f`.

### 3. Back up the essential files

Create a timestamped backup containing the configuration, validator key, and
validator signing state:

```sh
BACKUP_DIR="$HOME/.panacea/backups/pre-v2.3.0-files-$(date -u +%Y%m%dT%H%M%SZ)"
mkdir -m 0700 -p "$BACKUP_DIR"

tar -C $HOME/.panacea -czf "$BACKUP_DIR/config.tar.gz" config
cp -p $HOME/.panacea/data/priv_validator_state.json "$BACKUP_DIR/"
echo "$BACKUP_DIR"
```

Verify that the validator key is in the archive, then create and verify the
backup checksums:

```sh
tar -tzf "$BACKUP_DIR/config.tar.gz" |
  grep '^config/priv_validator_key.json$'

cd "$BACKUP_DIR"
sha256sum config.tar.gz priv_validator_state.json >SHA256SUMS
sha256sum -c SHA256SUMS
du -sh "$BACKUP_DIR"
```

The key check must print `config/priv_validator_key.json`, and every checksum
must report `OK`.

### 4. Migrate `app.toml`

Generate a preview and review it before changing the active file:

```sh
$HOME/.panacea/releases/v2.3.0/panacead config migrate v0.50 \
  --home $HOME/.panacea \
  --stdout >$HOME/.panacea/config/app.toml.v0.50.preview

diff -u \
  $HOME/.panacea/config/app.toml \
  $HOME/.panacea/config/app.toml.v0.50.preview
```

Most changes should be new defaults or the removal of obsolete settings.
Confirm that node-specific values remain correct.

After reviewing the diff, apply the migration and set the recommended query
limits:

```sh
$HOME/.panacea/releases/v2.3.0/panacead config migrate v0.50 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app query-gas-limit 10000000 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app api.rpc-write-timeout 10 \
  --home $HOME/.panacea

$HOME/.panacea/releases/v2.3.0/panacead config set app grpc.max-send-msg-size 10485760 \
  --home $HOME/.panacea
```

Validate `app.toml` and check both database backend settings:

```sh
$HOME/.panacea/releases/v2.3.0/panacead config view app \
  --home $HOME/.panacea >/dev/null && echo 'app.toml: OK'

grep -nE '^(db_backend|app-db-backend) =' \
  $HOME/.panacea/config/config.toml \
  $HOME/.panacea/config/app.toml
```

The first command must print `app.toml: OK`. `db_backend` must be `goleveldb`,
and `app-db-backend` must be empty or `goleveldb`. Do not start v2.3.0 with
`cleveldb`, `boltdb`, or `badgerdb`. If validation fails or either backend has
another value, do not start v2.3.0; report it in the official validator
channel.

### 5. Check the startup command

Skip this step if the node's startup command does not include
`--grpc-web.address`. If it does, remove the option before the upgrade because
v2.3.0 no longer supports it. Then restart the node with the same v2.2.0 or
v2.2.1 binary using your normal process manager and confirm that it resumes
normally.

### 6. Choose an upgrade method

Use one method only:

| Method | Before the upgrade height | At the upgrade height |
| --- | --- | --- |
| Cosmovisor | Stage the binary in the upgrade directory | Cosmovisor switches binaries automatically |
| Manual restart | Keep the verified release binary ready | Start v2.3.0 after the current binary halts |

#### Cosmovisor

Stage the verified binary and confirm that the copy is identical:

```sh
readlink -f $HOME/.panacea/cosmovisor/current

mkdir -p $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin

cp -p \
  $HOME/.panacea/releases/v2.3.0/panacead \
  $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead
chmod +x $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead

cmp \
  $HOME/.panacea/releases/v2.3.0/panacead \
  $HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead \
  && echo 'staged binary: OK'

$HOME/.panacea/cosmovisor/upgrades/v2.3.0/bin/panacead version --long |
  grep -E '^(version|commit):'
```

The comparison must print `staged binary: OK`. The version must be `2.3.0`
with the release commit listed above. Before the upgrade height,
`cosmovisor/current` must still select the currently running v2.2.0 or v2.2.1
binary.

Do not change `cosmovisor/current` manually. Keep Cosmovisor automatic binary
downloads disabled.

#### Manual restart

Keep the verified binary at
`$HOME/.panacea/releases/v2.3.0/panacead`. Confirm that your normal process
manager can be updated to use this path, but do not start v2.3.0 before the
upgrade height.

## At the upgrade height

**Cosmovisor:** Watch the node logs. Cosmovisor should select
`upgrades/v2.3.0/bin/panacead`, run the migration, and resume the chain.

**Manual restart:** Wait for the current v2.2.0 or v2.2.1 binary to halt at the
approved height. Update your process manager to use
`$HOME/.panacea/releases/v2.3.0/panacead`, then start the node.

After v2.3.0 starts, allow the migration to finish. Do not interrupt the
process unless it exits with an error.

## After the upgrade

Confirm that the running process uses the selected binary:

- **Cosmovisor:** `readlink -f $HOME/.panacea/cosmovisor/current` must point
  to `upgrades/v2.3.0`.
- **Manual restart:** the process manager must use
  `$HOME/.panacea/releases/v2.3.0/panacead`.

Then check the version, applied upgrade, and sync status:

```sh
$HOME/.panacea/releases/v2.3.0/panacead version --long |
  grep -E '^(version|commit):'

$HOME/.panacea/releases/v2.3.0/panacead query upgrade applied v2.3.0 \
  --home $HOME/.panacea \
  --node tcp://127.0.0.1:26657 \
  --output json | jq

curl -fsS --max-time 5 http://127.0.0.1:26657/status |
  jq '.result.sync_info | {latest_block_height, latest_block_time, catching_up}'
```

The version and commit must match the release, `catching_up` must be `false`,
and the block height must continue increasing. Also review the node logs for
migration or consensus errors.

## Public API and CLI command changes

This section applies only if the node exposes REST or gRPC-Web, or if you use
the `panacead` CLI to read or change client settings. Skip this section
otherwise. These changes do not affect consensus.

In v2.3.0, gRPC-Web no longer has its own listener on port `9091`; it is
served through the API listener, typically on port `1317`. Native gRPC
continues to use the address configured under `[grpc]`. If clients currently
use port `9091`, update them to the API address or keep the public address
stable and switch the reverse proxy upstream at the upgrade height. If browser
clients connect directly, configure CORS on the API listener or proxy.

The CLI commands for reading or changing `client.toml` now use this form:

- `panacead config get client <key>`
- `panacead config set client <key> <value>`

## Recovery

If the chain does not resume, preserve the logs and coordinate in the official
validator channel. Do not independently use `--unsafe-skip-upgrades`, restore
signing state, or roll back chain data.
