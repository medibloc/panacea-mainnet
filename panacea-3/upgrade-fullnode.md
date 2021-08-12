# Guidance for Full Node Operators

- The commit hash of Panacea [v2.0.1](https://github.com/medibloc/panacea-core/releases/tag/v2.0.1): `b36d1dac432c75a6d865e75767fe227a4ca125ca`
- The previous `panacea-2` chain will be halted at the block height: `12574375`.
- The new `panacea-3` chain will be started with the block height: `0`. But, all data will be preserved.

1. Verify that you are currently running the correct version (v1.3.3) of `panacead`.
```bash
panacead version --long
```

2. Stop your `panacead` and `panaceacli` processes.
```bash
# Stop the REST API server
pkill panaceacli

# Stop the Panacea daemon
pkill panacead
```

3. Back up your `~/.panacead` and `~/.panaceacli` directories.

Note that the new Panacea v2.0 will use only one home directory: `~/.panacea`.
So, the `~/.panacead` and `~/.panaceacli` are not used anymore.
```bash
mkdir -p ~/panacea-2-backup
mv ~/.panacead ~/panacea-2-backup/
mv ~/.panaceacli ~/panacea-2-backup/
```

For safety, it is also recommended to back up those directories to your cloud or external devices.

4. Install the Panacea v2.0.1.
```bash
git clone https://github.com/medibloc/panacea-core
cd panacea-core
git checkout v2.0.1
make install

panacead version --long
```

5. Create a new `~/.panacea` directory and copy parameters from previous config files.
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

6. Download the new `genesis.json`.
```bash
curl -o - https://raw.githubusercontent.com/medibloc/panacea-launch/master/panacea-3/genesis.json.gz | gunzip > ~/.panacea/config/genesis.json
```

7. Enable REST API enabled if you want

In case you have been running REST server with the command `panaceacli rest-server` previously,
running this command will not be necessary anymore.
REST API server is now in-process with the `panacead` and can be enabled by `~/.panacea/config/app.toml`.
```toml
[api]
# Enable defines if the API server should be enabled.
enable = true
# Swagger defines if swagger documentation should automatically be registered.
swagger = true
```
The `swagger` setting refers to enabling/disabling Swagger docs via the `/swagger/` API endpoint.

Alternatively, you can also enable a gRPC server which is more recommended than the REST server.
```toml
[grpc]
# Enable defines if the gRPC server should be enabled.
enable = true
# Address defines the gRPC server address to bind to.
address = "0.0.0.0:9090"
```

8. Start the daemon
```bash
panacead start
```

