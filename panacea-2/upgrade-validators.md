- [Intro](#intro)
- [Phase 1: `v1.2.5-internal` -> `v1.2.7-internal`](#phase-1-v125-internal---v127-internal)
  * [Purpose](#purpose)
  * [Steps](#steps)
  * [Rollback Plan](#rollback-plan)
- [Phase 2: `v1.2.7-internal` -> `v1.3.3-internal`](#phase-2-v127-internal---v132-internal)
  * [Purpose](#purpose-1)
  * [Steps](#steps-1)
  * [Rollback Plan](#rollback-plan-1)
- [Notes for Service Providers](#notes-for-service-providers)
- [Reference](#reference)


# Intro

- The upgrade height: **H** (To be decided soon).
- The upgrade procedure should be performed on `Jan. 5th, 2021 at or around 14:00 UTC`.
    - Note that there are some API breaking changes which should be followed up by service providers, such as trading companies. All service providers should prepare related changes on their side before the Mainnet upgrade. For details, please see the [Notes for Service Providers](#notes-for-service-providers).

# Phase 1: `v1.2.5-internal` -> `v1.2.7-internal`

## Purpose

For using the new `--halt-height` flag which was released as `v1.2.6-internal`. Also, `v1.2.7-internal` has a bug fix regards to the genesis export.

Because `v1.3.2` has the new cosmos module `did`, all nodes should be stopped at the same height, so that the genesis can be exported safely. To do that, the `--halt-height` is needed.

The upgrade `v1.2.5-internal` -> `v1.2.7-internal` can be done without exporting the genesis.

## Steps

0. If you are running validator nodes, please backup `~/.panacead/data/` to AWS S3 from one of nodes
```bash
sudo systemctl stop panacealcd
sudo systemctl stop panacead

cd ~/.panacead

# This example uses our internal AWS S3. Please change this to your environment.
tar czvf - data | aws s3 cp - s3://panacea-snapshot/mainnet-data-2020xxxx-v1.2.5.tar.gz

sudo systemctl start panacead
sudo systemctl start panacealcd
```

1. Backup the old panacea binaries
```bash
mkdir ~/v1.2.5_bak
cp /usr/local/bin/panacea* ~/v1.2.5_bak/

cp -R ~/.panaceacli ~/v1.2.5_bak/

mkdir -p ~/v1.2.5_bak/.panacead/
cp -R ~/.panacead/config ~/v1.2.5_bak/.panacead/
```

2. Stop `panacealcd` and `panacead` service
```bash
# If they were already terminated at the step 0, don't need to do anything.
sudo systemctl stop panacealcd
sudo systemctl stop panacead
```

3. Prepare the new panacea binaries
```bash
git checkout v1.2.7-internal && make install
sudo cp ~/go/bin/panacea* /usr/local/bin/
panaceacli version --long
panacead version --long
```

4. Start `panacead` with `--halt-height`
Don't use `systemctl` because we need to use the `--halt-height` option and the process shouldn't be restarted automatically.
Note that the halt height should be `H`.
```bash
panacead start --home=/home/centos/.panacead --halt-height=H
```

5. Start the `panacealcd` service (REST API server).
```bash
sudo systemctl start panacealcd
```

## Rollback Plan

Because `v1.2.7-internal` doesn't change any configs and data format, we can just rollback the binaries.

```bash
sudo cp ~/v1.2.5_bak/panacea* /usr/local/bin/

sudo systemctl start panacead
sudo systemctl start panacealcd
```


# Phase 2: `v1.2.7-internal` -> `v1.3.3-internal`

## Purpose

For DID operations + New `medibloc/cosmos-sdk` v0.37.15

## Steps

0. We don't need to backup data/configs. Let's use the backup made at the Phase 1 to reduce the downtime.

1. Wait until the `panacead` process is stopped at the halt-height.
```bash
watch 'ps -ef | grep panacead'
```

2. Stop the `panacealcd` service (REST API server).
```bash
sudo systemctl stop panacealcd
```

3. Export existing state
Note that the halt height is `H`.
```bash
panacead export --for-zero-height --height=H > ~/v1.2.7_genesis_export.json
```

4. Prepare the new panacea binaries.
```bash
git checkout v1.3.3-internal && make install
sudo cp ~/go/bin/panacea* /usr/local/bin/
panaceacli version --long
panacead version --long
```

5. Migrate exported state from the current cosmos v0.35.x to the new v0.36+ version.
Note that the chain ID must be updated to the `panacea-2` to prevent the double-signing.
```bash
panacead migrate v0.36 ~/v1.2.7_genesis_export.json --chain-id=panacea-2 --genesis-time=<genesis-time> > ~/v1.2.7_genesis_export_migrated.json
```

The `<genesis-time>` should be computed relative to the blocktime of `<halt-height>`. It shall be the blocktime of `<halt-height>` + `60 mins` with the sub-seconds truncated.

6. Reset state.
```bash
panacead unsafe-reset-all
```

7. Move the new genesis file to the config directory.
```bash
cp ~/v1.2.7_genesis_export_migrated.json ~/.panacead/config/genesis.json
```

8. Modify the `db_backend` from `leveldb` to `goleveldb`.
```bash
vi ~/.panacead/config/config.toml

# Database backend: goleveldb | memdb | cleveldb
db_backend = "goleveldb"
```

9. Start the `panacead`
Here, let's use the `systemctl` again, because we don't need the `--halt-height` anymore.
```bash
sudo systemctl start panacead
```

10. Modify the chain-id for `panacealcd`.
```bash
vi ~/.panaceacli/config/config.toml

chain-id = "panacea-2"

# or,

sudo vim /etc/systemd/system/panacealcd.service

ExecStart=/usr/local/bin/panaceacli .... --chain-id panacea-2
```

11. Start the `panacealcd` (REST API server).

```bash
sudo systemctl start panacealcd
```

## Rollback Plan

Because `v1.3.3-internal` will change some configs and data format, we should rollback with the data backed up at the Phase 1.
```bash
cd ~/.panacead
s3 cp s3://panacea-snapshot/mainnet-data-2020xxxx-v1.2.5.tar.gz - | tar -xzv

rm -rf ~/.panaceacli
cp -R ~/v1.2.5_bak/.panaceacli ~

rm -rf ~/.panacead/config
cp -R ~/v1.2.5_bak/.panacead/config ~/.panacead/

sudo cp ~/v1.2.5_bak/panacea* /usr/local/bin/
```

- [Notes for Service Providers](#notes-for-service-providers)
https://github.com/medibloc/panacea-launch/blob/master/panacea-2/upgrade.md#notes-for-service-providers

# Reference
https://github.com/cosmos/gaia/blob/master/docs/migration/cosmoshub-2.md
