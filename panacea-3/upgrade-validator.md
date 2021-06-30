# Guidance for Validators

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
It is a critical file needed to prevent double-signing, in case the upgrade fails and the previous chain needfs to be restarted.

In the event that the upgrade does not succeed, validators and full node operators must downgrade back to `panacea-2` (Panacea v1.3.3)
and restore to their latest snapshot before restarting their nodes.

## Upgrade Procedure

**NOTE**: It is assumed that you are currently operating a validator node running Panacea v1.3.3-internal with the Cosmos SDK v0.37.14.

- The commit hash of Panacea v2.0.0: TODO: to be described
- The height will be reset to 0 after the upgrade. All data will be preserved.

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

2. Restart the `panacead` process with the `--halt-height=<H>` option. TODO: `<H>` to be described.
```bash
sudo systemctl stop panacead

# Don't use systemctl because the process shouldn't be restarted automatically after being halted.
panacead start --halt-height=<H> 
```

3. After the chain is halted, make a backup of your `~/.panacead` directory.

Small files can be backed up by the following:
```bash
mkdir -p ~/panacea-2-backup/.panacead
cp -R ~/.panacead/config ~/panacea-2-backup/.panacead/

mkdir -p ~/panacea-2-backup/.panacead/data
cp -R ~/.panacead/data/priv_validator_state.json ~/panacea-2-backup/.panacead/data/

cp -R ~/.panaceacli ~/panacea-2-backup/
```

For big data files in the `~/.panacead/data/`, take an AWS EBS snapshot if your node is on AWS.

Alternatively, you can backup ~/.panacead/data/` to AWS S3:
```bash
cd ~/.panacead
tar cvzf - data | aws s3 cp - s3://panacea-snapshot/panacea-2-2021xxxx-v1.3.3.tar.gz
```

4. Export state.
```bash
panacead export --for-zero-height --height=<H> > ~/panacea-2-export.json
```

5. Prepare the new Panacea v2.0.0 binary.
```bash
git clone https://github.com/medibloc/panacea-core
cd panacea-core
git checkout v2.0.0
make install

panacead version --long
```

6. Migrate the exported state to the genesis file which is compatible with the new Panacea version.
```bash
# TODO: to be described
panacead migrate ~/panacea-2-export.json --chain-id panacea-3 > ~/panacea-3-genesis.json
```

7. Verify that the SHA-256 hash of the migrated file is the same as the hash of the genesis file
   on the [panacea-launch](https://github.com/medibloc/panacea-launch/panacea-3/genesis.json) GitHub.
```bash
jq -S -c -M '' ~/panacea-3-genesis.json | shasum -a 256
```

8. Rename the config directory and reset state.
```bash
mv ~/.panacead ~/.panacea

panacead unsafe-reset-all
```

9. Move the new genesis file to the config directory.
```bash
cp ~/panacea-3-genesis.json ~/.panacea/config/genesis.json
```

10. Start the daemon
```bash
panacead start
# or, sudo systemctl start panacead
```