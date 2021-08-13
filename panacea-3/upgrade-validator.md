# Guidance for Validators

**NOTE**:
Currently, all validator nodes are operated by MediBloc. So, this guide is only for MediBloc engineers.
If you are a full node operator, please see the [Guidance for Full Node Operators](upgrade-fullnode.md).


## Risks

As a validator, performing the upgrade procedure on your consensus nodes carries a heightened risk of double-signing and being slashed.
The most important piece of this procedure is verifying your software version and genesis file hash before starting your validator and signing.

The riskiest thing that a validator can do is discovering that they made a mistake and repeating the upgrade procedure again during the network startup.
If you discover a mistake, the best thing to do is wait for the network to start by other validators before correcting it.
If the network is halted, and if you have started with a different genesis file than the expected one,
seek advice from the Panacea team before resetting your validator.

## Recovery

Prior to exporting `panacea-2` state, validators are encouraged to take a full data snapshot at the export height.
Snapshotting depends heavily on infrastructure, but this can be done generally by backing up the `~/.panacead` directory.

It is critically important to back up the `~/.panacead/data/priv_validator_state.json` file after stopping your `panacead` process.
This file is updated every block as your validator participates in a consensus round.
It is a critical file needed to prevent double-signing, in case the upgrade fails and the previous chain needs to be restarted.

In the event that the upgrade does not succeed, validators and full node operators must downgrade back to `panacea-2` (Panacea v1.3.3-internal)
and restore to their latest snapshot before restarting their nodes.

## Upgrade Procedure

**NOTE**: It is assumed that you are currently operating a validator node running Panacea v1.3.3-internal with the Cosmos SDK v0.37.14.

- The commit hash of Panacea [v2.0.1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.1): `b36d1dac432c75a6d865e75767fe227a4ca125ca`
- The previous `panacea-2` chain will be halted at the block height: `12574375`.
- The new `panacea-3` chain will be started with the block height: `0`. But, all data will be preserved.

1. Verify that you are currently running the correct version (v1.3.3-internal) of `panacead`.
```bash
panacead version --long

name: panacea-core
server_name: panacead
client_name: panaceacli
version: 1.3.3-internal
commit: cbce38a79f740b9bf5d56571417b4206e5218d63
build_tags: ' ledger'
go: go version go1.15.6 linux/amd64
```

2. Restart the `panacead` process with the `--halt-height=12574375` option.
   If you are running multiple validator nodes, please perform a rolling restart to minimize risks.
```bash
sudo systemctl stop panacead

# Don't use systemctl because the process shouldn't be restarted automatically after being halted.
panacead start --halt-height=12574375
```

3. After the chain is halted, make a backup of your `~/.panacead` and `~/.panaceacli` directories.

Note that the new Panacea v2.0 will use only one home directory: `~/.panacea`.
So, the `~/.panacead` and `~/.panaceacli` are not used anymore.
```bash
mkdir -p ~/panacea-2-backup
mv ~/.panacead ~/panacea-2-backup/
mv ~/.panaceacli ~/panacea-2-backup/
```

For safety, it is also recommended to back up those directories to your cloud or external devices.
For instance, you can take an AWS EBS snapshot if your node is on AWS.
Alternatively, you can backup directories to AWS S3.
```bash
tar cvzf - ~/.panacead | aws s3 cp - s3://panacea-snapshot/panacea-2-2021xxxx-v1.3.3.tar.gz
```

4. Export state.
```bash
panacead export --home=$HOME/panacea-2-backup/.panacead --for-zero-height --height=<H> > ~/panacea-2-export.json
```

5. Prepare the new Panacea v2.0.1 binary.
```bash
git clone https://github.com/medibloc/panacea-core
cd panacea-core
git checkout v2.0.1
make install

panacead version --long
```

6. Migrate the exported state to the genesis file which is compatible with the new Panacea version.

Unfortunately, the `panacead migrate` command doesn't handle Tendermint consensus parameters. You need to update these parameters manually.
```bash
cat ~/panacea-2-export.json | \
  jq -c '.consensus_params.evidence.max_age_duration = "1814400000000000"' | \
  jq -c '.consensus_params.evidence.max_age_num_blocks = "1814400"' | \
  jq -c '.consensus_params.evidence.max_bytes = "50000"' | \
  jq -c 'del(.consensus_params.evidence.max_age)' | \
  jq > ~/panacea-2-export-manually-migrated.json
```
Then, you can run `panacead migrate` commands.
```bash
panacead migrate v0.38 ~/panacea-2-export-manually-migrated.json --chain-id panacea-3 | jq > ~/genesis.38.json
panacead migrate v0.39 ~/genesis.38.json --chain-id panacea-3 | jq > ~/genesis.39.json
panacead migrate v0.40 ~/genesis.39.json --chain-id panacea-3 | jq > ~/genesis.40.json
```

7. Change validator-related parameters and new parameters for IBC and Wasm.
```bash
cat ~/genesis.40.json | \
  jq -c '.app_state.staking.params.max_validators = 50' | \
  jq -c '.app_state.mint.minter.inflation = "0.070000000000000000"' | \
  jq -c '.app_state.mint.params.inflation_min = "0.070000000000000000"' | \
  jq -c '.app_state.mint.params.inflation_max = "0.100000000000000000"' | \
  jq -c '.app_state.mint.params.inflation_rate_change = "0.030000000000000000"' | \
  jq -c '.app_state.slashing.params.slash_fraction_downtime = "0.000100000000000000"' | \
  jq -c '.app_state.ibc.client_genesis.clients = []' | \
  jq -c '.app_state.ibc.client_genesis.clients_consensus = []' | \
  jq -c '.app_state.ibc.client_genesis.create_localhost = false' | \
  jq -c '.app_state.ibc.connection_genesis.connections = []' | \
  jq -c '.app_state.ibc.connection_genesis.client_connection_paths = []' | \
  jq -c '.app_state.ibc.channel_genesis.channels = []' | \
  jq -c '.app_state.ibc.channel_genesis.acknowledgements = []' | \
  jq -c '.app_state.ibc.channel_genesis.commitments = []' | \
  jq -c '.app_state.ibc.channel_genesis.receipts = []' | \
  jq -c '.app_state.ibc.channel_genesis.send_sequences = []' | \
  jq -c '.app_state.ibc.channel_genesis.recv_sequences = []' | \
  jq -c '.app_state.ibc.channel_genesis.ack_sequences = []' | \
  jq -c '.app_state.transfer.port_id = "transfer"' | \
  jq -c '.app_state.transfer.denom_traces = []' | \
  jq -c '.app_state.transfer.params.send_enabled = true' | \
  jq -c '.app_state.transfer.params.receive_enabled = true' | \
  jq -c '.app_state.capability.index = "1"' | \
  jq -c '.app_state.capability.owners = []' | \
  jq -c '.app_state.wasm.codes = []' | \
  jq -c '.app_state.wasm.contracts = []' | \
  jq -c '.app_state.wasm.gen_msgs = []' | \
  jq -c '.app_state.wasm.params.code_upload_access.address = ""' | \
  jq -c '.app_state.wasm.params.code_upload_access.permission = "Everybody"' | \
  jq -c '.app_state.wasm.params.instantiate_default_permission = "Everybody"' | \
  jq -c '.app_state.wasm.params.max_wasm_code_size = "614400"' | \
  jq -c '.app_state.wasm.sequences += [{"id_key":"BGxhc3RDb2RlSWQ=","value":"1"},{"id_key":"BGxhc3RDb250cmFjdElk","value":"1"}]' | \
  jq > ~/genesis.panacea-3.json
````

8. Verify that the SHA-256 hash of the migrated file is the same as the hash of the genesis file
   on the [panacea-launch](https://github.com/medibloc/panacea-launch/panacea-3/genesis.json) GitHub.
```bash
jq -S -c -M '' ~/genesis.panacea-3.json | shasum -a 256
```

9. Create a new `~/.panacea` directory and copy previous config files.
```bash
# Get your node's moniker from the previous 'config.toml'.
MONIKER=$(grep "^moniker = " ~/panacea-2-backup/.panacead/config/config.toml | awk '{print $3}' | sed 's|"||g')
panacead init ${MONIKER} --chain-id panacea-3

cp ~/panacea-2-backup/.panacead/config/priv_validator_key.json ~/.panacea/config/
cp ~/panacea-2-backup/.panacead/config/node_key.json ~/.panacea/config/

# Copy some parameters manually from previous config files
# Don't copy entire files, because new config files already contain some parameters that are newly introduced.
#
# NOTE: Make sure that timeout_commit = "1s"
vimdiff ~/panacea-2-backup/.panacead/config/config.toml ~/.panacea/config/config.toml
vimdiff ~/panacea-2-backup/.panacead/config/app.toml ~/.panacea/config/app.toml
```

10. Move the new genesis file to the config directory.
```bash
cp ~/genesis.panacea-3.json ~/.panacea/config/genesis.json
```

11. Start the daemon
```bash
panacead start
# or, sudo systemctl start panacead
```
