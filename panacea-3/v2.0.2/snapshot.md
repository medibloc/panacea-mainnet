# Panacea v2.0.2 Snapshot Instructions

Basically, follow the guide [Join the Network](https://medibloc.gitbook.io/panacea-core/guide/join-the-network) to complete the initial setup.

The currently stored data is replaced with snapshot data.
```shell
# Remove before data.
rm -rf ~/.panacea/data ~/.panacea/wasm

curl https://panacea-snapshots.s3.ap-northeast-2.amazonaws.com/mainnet-panacea-3-20211210-v2.0.2.tar.gz | tar xvz -C ~/.panacea/

panacead version | grep 2.0.2 || echo 'Bad version! Please update to v2.0.2'
```

You can synchronize blocks from height 2536957.
