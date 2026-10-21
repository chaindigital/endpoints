<p align="center">
  <a href="https://snapshots.chaindigital.tech"><img src="https://raw.githubusercontent.com/chaindigital/endpoints/main/media/banner.svg?v=5" alt="Chain Digital — Celestia RPC, API and snapshots"/></a>
</p>

<p align="center">
  <img width="100%" src="https://capsule-render.vercel.app/api?type=rect&color=0:0E4C75,50:56CCF2,100:2D9CDB&height=4&section=header"/>
</p>

<p align="center">
  <a href="https://snapshots.chaindigital.tech"><b>snapshots.chaindigital.tech</b></a> &nbsp;·&nbsp;
  <a href="https://chaindigital.tech/tools">chaindigital.tech/tools</a> &nbsp;·&nbsp;
  <a href="https://github.com/chaindigital">github.com/chaindigital</a>
</p>

<br/>

Public RPC, REST API and pruned snapshots run by [Chain Digital](https://chaindigital.tech) from our own Celestia nodes.
The live page at **[snapshots.chaindigital.tech](https://snapshots.chaindigital.tech)** always shows the current snapshot, so check there first.

<br/>

## Celestia

| | Mainnet `celestia` | Testnet `mocha-5` |
|---|---|---|
| **RPC** | [`celestia-rpc.chaindigital.tech`](https://celestia-rpc.chaindigital.tech) | [`celestia-testnet-rpc.chaindigital.tech`](https://celestia-testnet-rpc.chaindigital.tech) |
| **API** | [`celestia-api.chaindigital.tech`](https://celestia-api.chaindigital.tech) | [`celestia-testnet-api.chaindigital.tech`](https://celestia-testnet-api.chaindigital.tech) |
| **Snapshot** | [`latest.tar.zst`](https://snapshots.chaindigital.tech/celestia/mainnet/latest.tar.zst) | [`latest.tar.zst`](https://snapshots.chaindigital.tech/celestia/mocha-5/latest.tar.zst) |
| **Manifest** | [`latest.json`](https://snapshots.chaindigital.tech/celestia/mainnet/latest.json) | [`latest.json`](https://snapshots.chaindigital.tech/celestia/mocha-5/latest.json) |
| **All files** | [`/celestia/mainnet/`](https://snapshots.chaindigital.tech/celestia/mainnet/) | [`/celestia/mocha-5/`](https://snapshots.chaindigital.tech/celestia/mocha-5/) |

### Quick check

```bash
# latest block height
curl -s https://celestia-rpc.chaindigital.tech/status | jq -r .result.sync_info.latest_block_height

# node version over REST
curl -s https://celestia-api.chaindigital.tech/cosmos/base/tendermint/v1beta1/node_info | jq -r .application_version.version
```

<br/>

## Snapshots

Pruned snapshots, rebuilt every 6 hours from live nodes. Each one has a `.sha256` file next to it, and `latest.json` describes the current one:

```json
{
  "network":    "mainnet",
  "filename":   "celestia_mainnet_<date>_<time>.tar.zst",
  "height":     "…",
  "version":    "…",
  "db_backend": "pebbledb",
  "pruning":    "default",
  "size":       "…",
  "sha256":     "…",
  "timestamp":  "…"
}
```

> [!IMPORTANT]
> Snapshots use **`pebbledb`**. Set `app-db-backend = "pebbledb"` in `app.toml` before restoring. The node will not open the database on any other backend.

### Restore, streaming

Fastest: download and unpack in one pass. For the testnet, replace `mainnet` with `mocha-5`.

```bash
sudo systemctl stop celestia-appd

cp $HOME/.celestia-app/data/priv_validator_state.json \
   $HOME/.celestia-app/priv_validator_state.json.backup

rm -rf $HOME/.celestia-app/data

curl -L https://snapshots.chaindigital.tech/celestia/mainnet/latest.tar.zst \
    | tar --zstd -xf - -C $HOME/.celestia-app/

mv $HOME/.celestia-app/priv_validator_state.json.backup \
   $HOME/.celestia-app/data/priv_validator_state.json

sed -i 's/^app-db-backend = .*/app-db-backend = "pebbledb"/' \
    $HOME/.celestia-app/config/app.toml

sudo systemctl restart celestia-appd && sudo journalctl -u celestia-appd -f
```

### Restore, verified

Downloads the file named in the manifest together with its checksum, and unpacks only if the checksum matches. You need free disk space for the archive.

```bash
NET=mainnet   # or mocha-5
BASE=https://snapshots.chaindigital.tech/celestia/$NET
FILE=$(curl -s $BASE/latest.json | jq -r .filename)

cd /tmp
curl -LO $BASE/$FILE
curl -LO $BASE/$FILE.sha256
sha256sum -c $FILE.sha256 || exit 1

sudo systemctl stop celestia-appd
cp $HOME/.celestia-app/data/priv_validator_state.json $HOME/.celestia-app/priv_validator_state.json.backup
rm -rf $HOME/.celestia-app/data
tar --zstd -xf /tmp/$FILE -C $HOME/.celestia-app/
mv $HOME/.celestia-app/priv_validator_state.json.backup $HOME/.celestia-app/data/priv_validator_state.json
sed -i 's/^app-db-backend = .*/app-db-backend = "pebbledb"/' $HOME/.celestia-app/config/app.toml
sudo systemctl restart celestia-appd && rm /tmp/$FILE /tmp/$FILE.sha256
```

> [!NOTE]
> Restoring `priv_validator_state.json` keeps your validator from signing heights it has already signed. Never skip that step on a validator.

<br/>

---

<p align="center">
  <sub>Something down or out of date? Open an issue here or write to <a href="mailto:chaindigitalsystem@gmail.com">chaindigitalsystem@gmail.com</a></sub>
</p>
