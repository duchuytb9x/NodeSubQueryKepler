# Restoring Dictionary Projects

The first SubQuery projects indexed on Kepler will be [Dictionary Projects](../../../academy/tutorials_examples/dictionary.md). These projects are commonly used, and are simple to index and create, but create large datasets.

In order to speed up the onboarding of Indexers, we are providing database snapshots for most of these dictionaries

## Downloading Database Snapshots

| Network   | Deployment ID                                    | Database Size | S3 Bucket URL                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | BT Magnet Link                                                                                                                                                                                                                                                                                             | SHA256                                                             |
| --------- | ------------------------------------------------ | ------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Pollkadot | `QmZGAZQ7e1oZgfuK4V29Fa5gveYK3G2zEwvUzTZKNvSBsm` | ~220GB | [S3 URL](https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/polkadot/polkadot.tar) | TBC | `5725b13d70d73956289f7e0e448bc9a8b5c2146a864057572c4fa97843aa7fab` |
| Kusama    | `QmXwfCF8858YY924VHgNLsxRQfBLosVbB31mygRLhgJbWn` | ~260GB | [S3 URL](https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/kusama/kusama.tar) | TBC | 722882ecdefada068ac5538552d71cdf01fb87856449fcc242e2552002cbb55d | 
|Moonbeam  | `QmRwisx41SRrR8fhNTRpKetNdpP7SaNkRAVrRgcdwoEtCH` | ~200GB | [S3 URL](https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/moonbean/moonbean.tar) | TBC |00d1185d37f6b96423aec6e08ff7e07dede07210af71e4f988ac1f0c949716de |                                                                                                 

You can download the snapshot either from the s3 bucket URL or the BitTorrent magnet link:

### Download Snapshot

Downloading via Bittorrent will be faster, you can use your favourite bittorent client or install Aria2

```bash
sudo apt update
sudo apt install aria2

aria2c <Magnet_Link>
```

- To download Polkadot from an S3 bucket

```bash
curl -o polkadot.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/polkadot/polkadot.tar
```
- To download Kusama from an S3 bucket

```bash
curl -o kusama.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/kusama/kusama.tar
```
- To download Moonbeam from an S3 bucket

```bash
curl -o moonbean.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/moonbean/moonbean.tar
```

## Restoring the Database Snapshot

This assumes that you have an indexer [running locally](../../../run_publish/run.md) with admin access to a PostgresQL database (you will be using the `pg_restore` command).

First extract the dowloaded snapshot and then extract it using the following command. You will get 3 files: `.mmr` and `schema_xxxxxxx.dump` and a shell script: `restore.sh`

```bash
tar -xvf polkadot.tar
```

```bash
tar -xvf kusama.tar
```

```bash
tar -xvf moonbean.tar
```

You can choose to use the `restore.sh` script to restore data automatically.

There are 2 parameters to run this script, the first one is your `mmrPath` which config in your coordinator container, the second one is you data folder path, normally it will be `.data/postgres` folder at the same path with the `docker-compose.yml` file. For example:
```
sh restore.sh /home /root/indexer-services/.data/postgres > restore.log 2>&1 &
```

::: note
Make sure your `indexer_db` and `indexer_coordinator` containers are running with healthy status
:::


Alternatively you can choose the following steps to do the data restore manually.

1. Move `.mmr` to `<MMR_PATH>/poi/<Deployment_CID>`. Note that `<MMR_PATH>` is the path in your `docker-compose.yml` for the indexer-coordinator container under `--mmrPath`.




2. Copy the `schema_xxxxxxx.dump` to `.data/postgres/` and then use this command:

- Copy schema_qmzgazq7e1ozgfu.dump of Polkadot to `/root/subquery-indexer/.data/postgres`

```bash
cp /root/polkadot/schema_qmzgazq7e1ozgfu.dump /root/subquery-indexer/.data/postgres/schema_qmzgazq7e1ozgfu.dump
```

- Copy schema_qmxwfcf8858yy92.dump of Kusama to `/root/subquery-indexer/.data/postgres`

```bash
cp /root/kusama/schema_qmxwfcf8858yy92.dump /root/subquery-indexer/.data/postgres/schema_qmxwfcf8858yy92.dump
```

- Copy schema_qmrwisx41srrr8f.dump of Moonbeam to `/root/subquery-indexer/.data/postgres`

```bash
cp /root/moonbean/schema_qmrwisx41srrr8f.dump /root/subquery-indexer/.data/postgres/schema_qmrwisx41srrr8f.dump
```

- Run this command in `tmux`:

```bash
docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmzgazq7e1ozgfu.dump > /root/restore.log 2>&1
```

- Run this command in `tmux`:

```bash
docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmxwfcf8858yy92.dump > /root/restore.log 2>&1
```

- Run this command in `tmux`:

```bash
docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmrwisx41srrr8f.dump > /root/restore.log 2>&1
```

>Note
>
>We use the `-j` parameter to update the number of jobs running concurrently. Depending on your machine size, you may want to increase this number to speed up the restore process. [Read more](https://www.postgresql.org/docs/current/app-pgrestore.html)

The restore process will start and take quite a long time (like 2 days), please make sure you run this cmd in the background (use tools like tmux/screen/nohup). Here is an example of the output log.

```
pg_restore: creating SCHEMA "schema_qmzj9whrhrmvn2h"
pg_restore: creating FUNCTION "schema_qmzj9whrhrmvn2h.schema_notification()"
pg_restore: creating TABLE "schema_qmzj9whrhrmvn2h._metadata"
pg_restore: creating TABLE "schema_qmzj9whrhrmvn2h._poi"
pg_restore: creating TABLE "schema_qmzj9whrhrmvn2h.events"
pg_restore: creating TABLE "schema_qmzj9whrhrmvn2h.extrinsics"
pg_restore: creating TABLE "schema_qmzj9whrhrmvn2h.spec_versions"
pg_restore: processing data for table "schema_qmzj9whrhrmvn2h._metadata"
pg_restore: processing data for table "schema_qmzj9whrhrmvn2h._poi"
pg_restore: processing data for table "schema_qmzj9whrhrmvn2h.events"
pg_restore: processing data for table "schema_qmzj9whrhrmvn2h.extrinsics"
pg_restore: processing data for table "schema_qmzj9whrhrmvn2h.spec_versions"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._metadata _metadata_pkey"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._poi _poi_chainBlockHash_key"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._poi _poi_hash_key"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._poi _poi_mmrRoot_key"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._poi _poi_parentHash_key"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h._poi _poi_pkey"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h.events events_pkey"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h.extrinsics extrinsics_pkey"
pg_restore: creating CONSTRAINT "schema_qmzj9whrhrmvn2h.spec_versions spec_versions_pkey"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.events_block_height__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.events_event__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.events_module__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.extrinsics_block_height__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.extrinsics_call__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.extrinsics_module__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.extrinsics_tx_hash__block_range"
pg_restore: creating INDEX "schema_qmzj9whrhrmvn2h.poi_hash"
pg_restore: creating TRIGGER "schema_qmzj9whrhrmvn2h._metadata 0xc1aaf8b4176d0f02"
```

After the data restored, you can start adding the specific project to you service inside admin app, and start indexing the project, the indexing will start basing on the restored data and continue indexing the project.
