# Welcome to Kepler in Linux!
<img width="700" alt="Screen Shot 2022-11-01 at 20 54 41" src="https://blog.subquery.network/content/images/size/w1200/2023/04/kepler-on-poly.png">

## Recommended hardware requirements
- CPU: 2 CPU
- Memory: 8 GB RAM
- Disk: 1 TB SSD Storage
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
| onfinality/subql-coordinator   | v1.0.4    |
| onfinality/subql-indexer-proxy | v1.0.2    |

>Important
>
>Please go through the docker-compose file carefully, and change the following parameters to your own values:
>- POSTGRES_PASSWORD
>- postgres-password
>- secret-key
>- jwt-secret

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

- Download Polkadot Snapshot:

```
curl -o polkadot.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/polkadot/polkadot.tar
```

- Download Kusama Snapshot:

```
curl -o kusama.tar https://kepler-dictionary-projects.s3.ap-southeast-2.amazonaws.com/kusama/kusama.tar
```

- Extract Polkadot Snapshot:

```
tar -xvf polkadot.tar
```
- Extract Kusama Snapshot:

```
tar -xvf kusama.tar
```

- Copy schema_qmzgazq7e1ozgfu.dump of Polkadot to `/root/subquery-indexer/.data/postgres`:

```
cp /root/polkadot/schema_qmzgazq7e1ozgfu.dump /root/subquery-indexer/.data/postgres/schema_qmzgazq7e1ozgfu.dump
```

- Copy .mmr of Polkadot to `/home/poi/QmZGAZQ7e1oZgfuK4V29Fa5gveYK3G2zEwvUzTZKNvSBsm`:

```
mkdir -p /home/poi/QmZGAZQ7e1oZgfuK4V29Fa5gveYK3G2zEwvUzTZKNvSBsm
cp /root/polkadot/.mmr /home/poi/QmZGAZQ7e1oZgfuK4V29Fa5gveYK3G2zEwvUzTZKNvSBsm/.mmr
```


- Copy schema_qmxwfcf8858yy92.dump of Kusama to `/root/subquery-indexer/.data/postgres`:

```
cp /root/kusama/schema_qmxwfcf8858yy92.dump /root/subquery-indexer/.data/postgres/schema_qmxwfcf8858yy92.dump
```

- Copy .mmr of Kusama to `/home/poi/QmXwfCF8858YY924VHgNLsxRQfBLosVbB31mygRLhgJbWn`:

```
mkdir -p /home/poi/QmXwfCF8858YY924VHgNLsxRQfBLosVbB31mygRLhgJbWn
cp /root/kusama/.mmr /home/poi/QmXwfCF8858YY924VHgNLsxRQfBLosVbB31mygRLhgJbWn/.mmr
```

- Install Tmux:

```
apt install tmux
```

- Open Tmux:

```
tmux
```

- Restore PostgresQL Polkadot database:

```
tmux new-session -d -s restorePolkadot 'docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmzgazq7e1ozgfu.dump > /root/restore.log 2>&1'
```

- Restore PostgresQL Kusama database:

```
tmux new-session -d -s restoreKusama 'docker exec -it indexer_db pg_restore -v -j 5 -h localhost -p 5432 -U postgres -d postgres /var/lib/postgresql/data/schema_qmxwfcf8858yy92.dump > /root/restore1.log 2>&1'
```

- To list the running tmux sessions, you can use the command:

```
tmux ls
```

>Note
>
>We use the `-j` parameter to update the number of jobs running concurrently. Depending on your machine size, you may want to increase this number to speed up the restore process. [Read more](https://www.postgresql.org/docs/current/app-pgrestore.html)

The restore process will start and take quite a long time (like 2 days), please make sure you run this cmd in the background (use tools like tmux/screen/nohup). Here is an example of the output log.

```
pg_restore: creating SCHEMA "schema_qmzgazq7e1ozgfu"
pg_restore: processing item 421 FUNCTION schema_notification()
pg_restore: creating FUNCTION "schema_qmzgazq7e1ozgfu.schema_notification()"
pg_restore: processing item 215 TABLE _metadata
pg_restore: creating TABLE "schema_qmzgazq7e1ozgfu._metadata"
pg_restore: processing item 216 TABLE _poi
pg_restore: creating TABLE "schema_qmzgazq7e1ozgfu._poi"
pg_restore: processing item 217 TABLE events
pg_restore: creating TABLE "schema_qmzgazq7e1ozgfu.events"
pg_restore: processing item 218 TABLE extrinsics
pg_restore: creating TABLE "schema_qmzgazq7e1ozgfu.extrinsics"
pg_restore: processing item 219 TABLE spec_versions
pg_restore: creating TABLE "schema_qmzgazq7e1ozgfu.spec_versions"
pg_restore: entering main parallel loop
pg_restore: launching item 3311 TABLE DATA _metadata
pg_restore: launching item 3312 TABLE DATA _poi
pg_restore: launching item 3313 TABLE DATA events
pg_restore: launching item 3314 TABLE DATA extrinsics
pg_restore: launching item 3315 TABLE DATA spec_versions
pg_restore: processing data for table "schema_qmzgazq7e1ozgfu.spec_versions"
pg_restore: processing data for table "schema_qmzgazq7e1ozgfu.events"
pg_restore: processing data for table "schema_qmzgazq7e1ozgfu._metadata"
pg_restore: processing data for table "schema_qmzgazq7e1ozgfu._poi"
pg_restore: processing data for table "schema_qmzgazq7e1ozgfu.extrinsics"
pg_restore: finished item 3311 TABLE DATA _metadata
pg_restore: launching item 3149 CONSTRAINT _metadata _metadata_pkey
pg_restore: creating CONSTRAINT "schema_qmzgazq7e1ozgfu._metadata _metadata_pkey"
pg_restore: finished item 3149 CONSTRAINT _metadata _metadata_pkey
pg_restore: launching item 3183 TRIGGER _metadata 0x3f87069a64ae6471
pg_restore: creating TRIGGER "schema_qmzgazq7e1ozgfu._metadata 0x3f87069a64ae6471"
pg_restore: finished item 3183 TRIGGER _metadata 0x3f87069a64ae6471
pg_restore: launching item 3184 TRIGGER _metadata 0xadf0dc4658acf912
pg_restore: creating TRIGGER "schema_qmzgazq7e1ozgfu._metadata 0xadf0dc4658acf912"
pg_restore: finished item 3184 TRIGGER _metadata 0xadf0dc4658acf912
pg_restore: finished item 3315 TABLE DATA spec_versions
pg_restore: launching item 3178 INDEX 0x510c96a4f8868d8a
pg_restore: creating INDEX "schema_qmzgazq7e1ozgfu.0x510c96a4f8868d8a"
```
- To check the restore is finished, the log will end as follows::

```
pg_restore: finished main parallel loop
```

After the data restored, you can start adding the specific project to you service inside admin app, and start indexing the project, the indexing will start basing on the restored data and continue indexing the project.


## 6. SSL Certificate Configuration

### Why should you configure and enable a SSL certificate?

SSL certificate is an important way to secure your indexer service and encrypt traffic between your query service and the Consumers requesting data from it. It's a good idea to obtain an SSL certificate for any professional service and SubQuery is no different. Additionally, a SSL certificate is required for when a Consumer access a website via SSL and that website is requesting data from your SubQuery project.

**Since it's such a standard best practice, although it's not required, wee rank Indexer higher in the Indexer leaderboard if they have SSL enabled.**

### How do I know that an Indexer has SSL enabled

The `Proxy Server Endpoint` in indexer's metadata will start with `https` instead of `http`.

### Enabling SSL

### 6.1. Purchase a domain name

You can purchase a domain from a lot of domain providers, we won't be providing a guide on how to do this. The rest of this guide assumes you have purchased `mysqindexer.com`

### 6.2. Config DNS

Add a A record to your domain that points to your indexer's server IP address. For example, we add an A record for `proxy.mysqindexer.com` to our server's IP `210.10.5.61`.

### 6.3. Get a SSL Cert

You can get a free SSL Cert from [Let's Encrypt](https://letsencrypt.org/), they offer a lot of ways to get a SSL Cert, you can choose the [one that fits your needs](https://letsencrypt.org/docs/client-options/).

This tutorial will use Certbot + NGINX + Ubuntu as an example.

#### 6.3.1 Install Certbot

Check [https://certbot.eff.org/instructions](https://certbot.eff.org/instructions) on how to do this step

#### 6.3.2 Install & Config NGINX
##### 6.3.2.1 Install Nginx
We will use nginx for two purpose. 
1. Listen on port 1080 to allow Let's encrypt to verify your domain name.
2. As a reverse proxy to forward traffic from port 443 (https) to indexer-proxy.
```shell
sudo apt install -y nginx
```

##### 6.3.2.2 Reconfig indexer-proxy
In the default settings, indexer-proxy listen on port 80, now we need to change it to 1080. 
```shell
# docker-compose.yml
proxy:
    image: onfinality/subql-indexer-proxy:v1.0.1
    container_name: indexer_proxy
    restart: always
    ports:
      - 1080:1080
    command:
      - --host=0.0.0.0
      - --port=1080
      - --auth
      - --jwt-secret=<...>
      - --secret-key=<...>
      - --service-url=http://indexer_coordinator:8000
      - --network=kepler                                  # network type, need to be same with coordinator
      - --network-endpoint=https://polygon-rpc.com
      - --token-duration=24                                 # query auth token validity [hours]
      - --redis-endpoint=redis://indexer_cache
```
then restart the indexer-proxy container
```shell
docker-compose up -d --no-deps proxy
```
##### 6.3.2.3 Config NGINX
Edit your NGINX configuration to add the following (e.g. it would usually be at `/etc/nginx/sites-available/proxy.mysqindexer.com`)

```shell
# /etc/nginx/sites-available/proxy.mysqindexer.com
server {
    listen 443 ssl; // Update the ports to listen on
    listen [::]:443 ssl;

    server_name proxy.mysqindexer.com; // update the server name to match your DNS address

    location / {
      proxy_pass      http://127.0.0.1:1080;
    }

}

# link the new configution with a symlink to your edited file
sudo ln -s /etc/nginx/sites-available/proxy.mysqindexer.com /etc/nginx/sites-enabled/proxy.mysqindexer.com
```

Then finish configuration.
##### 6.3.2.3 Run certbot
```bash
# run certbot
sudo certbot --nginx -d proxy.mysqindexer.com

# output
# Saving debug log to /var/log/letsencrypt/letsencrypt.log
# Requesting a certificate for proxy.mysqindexer.com

# Successfully received certificate.
# Certificate is saved at: /etc/letsencrypt/live/proxy.mysqindexer.com/fullchain.pem
# Key is saved at:         /etc/letsencrypt/live/proxy.mysqindexer.com/privkey.pem
# This certificate expires on 2023-08-06.
# These files will be updated when the certificate renews.
# Certbot has set up a scheduled task to automatically renew this certificate in the background.
#
# Deploying certificate
# Successfully deployed certificate for proxy.mysqindexer.com to /etc/nginx/sites-enabled/proxy.mysqindexer.com
# Congratulations! You have successfully enabled HTTPS on https://proxy.mysqindexer.com
#
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# If you like Certbot, please consider supporting our work by:
#  * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
#  * Donating to EFF:                    https://eff.org/donate-le
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```

### 6.4. Update your indexer metadata

Update your indexer metadata, set the `Proxy Server Endpoint` to `https://proxy.mysqindexer.com`


## ALL DONE!!!
