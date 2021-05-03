# Intro

- The upgrade height: **H** (TO BE ANNOUNCED)
- The upgrade procedure should be performed on `June 8th, 2021 at or around 02:00 UTC`.
- Version: `v1.3.3` -> `v1.3.4` (TO BE RELEASE SOON)
- Chain ID: `panacea-2` -> `panacea-3`


## Purpose

- New features that support more DID Document properties
- Genesis parameter updates for the validator joining


## Upgrade Procedure

1. Stop `panacead` service
```bash
sudo systemctl stop panacead
```

2. Restart `panacead` with `--halt-height`
Don't use `systemctl` because we need to use the `--halt-height` option and the process shouldn't be restarted automatically.
Note that the halt height should be `H` (TO BE ANNOUNCED).
```bash
panacead start --home=$HOME/.panacead --halt-height=H
```

3. Wait until the `panacead` process is stopped at the halt-height.
```bash
watch 'ps -ef | grep panacead'
```

4. Stop the `panacealcd` service (REST API server).
```bash
sudo systemctl stop panacealcd
```

5. Backup `config`s and `priv_validator_state.json`.

```bash
mkdir -p ~/backup/panacea-2/.panacead
cp -R ~/.panacead/config ~/backup/panacea-2/.panacead/

mkdir -p ~/backup/panacea-2/.panacead/data
cp ~/.panacead/data/priv_validator_state.json ~/backup/panacea-2/.panacead/data

cp -R ~/.panaceacli ~/backup/panacea-2/
```

Also, take an AWS EBS snapshot from one of validator nodes, after stopping the `panacead`.

Alternatively, you can backup `~/.panacead/data/` to AWS S3 from one of validator nodes:
```bash
sudo systemctl stop panacealcd
sudo systemctl stop panacead

cd ~/.panacead

# This example uses our internal AWS S3. Please change this to your environment.
tar czvf - data | aws s3 cp - s3://panacea-snapshot/mainnet-data-2020xxxx-v1.3.3.tar.gz
```

6. Export existing state
Note that the halt height is `H` (TO BE ANNOUNCED).
```bash
panacead export --for-zero-height --height=H > ~/v1.3.3_genesis_export.json
```

7. Prepare the new panacea binaries
```bash
git checkout v1.3.4 && make install
sudo cp ~/go/bin/panacea* /usr/local/bin/
panaceacli version --long
panacead version --long
```

8. Reset state.
```bash
panacead unsafe-reset-all
```

9. Modify parameters in the genesis file (for validator joining).
```bash
cp ~/v1.3.3_genesis_export.json ~/v1.3.4_genesis.json

vi ~/v1.3.4_genesis.json

cp ~/v1.3.4_genesis.json ~/.panacead/config/genesis.json
```
(TO BE DESCRIBED SOON)

And, upload the new genesis file to GitHub.
```bash
cp ~/.panacead/config/genesis.json $REPO/panacea-3/genesis.json
git add $REPO/panacea-3/genesis.json
git commit -m '...'
git push ...
```

10. Modify the chain-id for `panacealcd`.
```bash
vi ~/.panaceacli/config/config.toml

chain-id = "panacea-3"

# or,

sudo vim /etc/systemd/system/panacealcd.service

ExecStart=/usr/local/bin/panaceacli .... --chain-id panacea-3
```

11. Start services

```bash
sudo systemctl start panacead
sudo systemctl start panacealcd
```


## Rollback Plan

```bash
sudo systemctl stop panacead
sudo systemctl stop panacealcd

rm -rf ~/.panacead/data
cd ~/.panacead
s3 cp s3://panacea-snapshot/mainnet-data-2020xxxx-v1.3.3.tar.gz - | tar -xzv

rm -rf ~/.panaceacli
cp -R ~/backup/panacea-2/.panaceacli ~/

rm -rf ~/.panacead/config
cp -R ~/backup/panacea-2/.panacead/config ~/.panacead/

git checkout v1.3.3-internal && make install
sudo cp ~/go/bin/panacea* /usr/local/bin/

sudo systemctl start panacead
sudo systemctl start panacealcd
```


# Reference
https://github.com/cosmos/gaia/blob/master/docs/migration/cosmoshub-2.md
