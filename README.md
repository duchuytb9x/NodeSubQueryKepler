# Welcome to Kepler!
<img width="700" alt="Screen Shot 2022-11-01 at 20 54 41" src="https://blog.subquery.network/content/images/size/w1200/2023/04/kepler-on-poly.png">

## Recommended hardware requirements
- CPU: 2 CPU
- Memory: 8 GB RAM
- Disk: 400 GB SSD Storage
>Storage requirements will vary based on various factors (age of the chain, transaction rate, etc)

## Official documentation:

- How to Become an Indexer [here](https://academy.subquery.network/subquery_network/kepler/indexers/become-an-indexer.html)

- Install Indexing Service on Linux [here](https://academy.subquery.network/subquery_network/kepler/indexers/install-indexer-linux.html)


## 1. Install Docker and Docker-Compose:
- Get update:
```
sudo apt-get update
```

- Install the docker:
```
sudo apt install docker.io
```

- Install the docker-compose:
```
# get the latest version for docker-compose
sudo curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m) -o /usr/bin/docker-compose

# fix permissions after download:
sudo chmod +x /usr/bin/docker-compose

# verify the installation
sudo docker-compose version

```
## 2. Download the Docker Compose File for Indexer Services:
Run the following command:
```
mkdir subquery-indexer && cd subquery-indexer
curl https://raw.githubusercontent.com/subquery/indexer-services/kepler/docker-compose.yml -o docker-compose.yml
```
This will overwrite the existing docker-compose.yml file.

Make sure the indexer service versions are correct:


| Service                        | Version   |
| :----------------------------- | :-------- |
| onfinality/subql-coordinator   | latest    |
| onfinality/subql-indexer-proxy | latest    |

>Important
>
>Please change the **POSTGRES_PASSWORD** in postgres and **postgres-password** in coordinator-service to your own one

## 3. Start Indexer Services:

Run the service using the following command::
```
sudo docker-compose up -d
```

It will start the following services:

- coordinator_db
- coordinator_service
- coordinator_proxy
- proxy-redis

## 4. Set Up Auto Start:

Create `/etc/systemd/system/subquery.service.` 

Run the service using the following command::
```
sudo nano /etc/systemd/system/subquery.service
```

Coppy and Paste:

```
[Unit]
Description="Subquery systemd service"
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=root
SyslogIdentifier=subquery
SyslogFacility=local7
KillSignal=SIGHUP
WorkingDirectory=/root/subquery-indexer
ExecStart=/usr/bin/docker-compose up -d

[Install]
WantedBy=multi-user.target
```

Register and start the service by running:
```
systemctl enable subquery.service
systemctl start subquery.service
```

After that, verify that the service is running:
```
systemctl status subquery.service
```

## 5. Restoring Dictionary Projects:

- Download Polkadot Snapshot

```
curl -o polkadot.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/polkadot/polkadot.tar
```

- Download Moonbeam Snapshot

```
curl -o moonbeam.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/moonbean/moonbean.tar
```

- Extract Polkadot Snapshot

```
tar -xvf polkadot.tar
```

- Extract Moonbeam Snapshot

```
tar -xvf moonbeam.tar
```

- Extract Moonbeam Snapshot

```
cp /root/polkadot/schema_qmzgazq7e1ozgfu.dump /root/subquery-indexer/.data/postgres/schema_qmzgazq7e1ozgfu.dump
```

- Extract Moonbeam Snapshot

```
cp /root/moonbeam/schema_qmrwisx41srrr8f.dump /root/subquery-indexer/.data/postgres/schema_qmrwisx41srrr8f.dump
```

- Install Tmux

```
apt install tmux
```

- Open Tmux

```
tmux
```

- Restore PostgresQL Polkadot database

```
docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmzgazq7e1ozgfu.dump > /root/restore.log 2>&1
```

- Restore PostgresQL Moonbeam database

```
docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmrwisx41srrr8f.dump > /root/restore.log 2>&1
```

>Note
>
>We use the `-j` parameter to update the number of jobs running concurrently. Depending on your machine size, you may want to increase this number to speed up the restore process. [Read more](https://www.postgresql.org/docs/current/app-pgrestore.html)

## ALL DONE!!!
