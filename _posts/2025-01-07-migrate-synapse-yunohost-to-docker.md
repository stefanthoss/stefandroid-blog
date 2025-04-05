---
layout: post
title: "Migrate Synapse Matrix Server from YunoHost to Docker"
description: "Migrate a Synapse server for the Matrix protocol which was initially started as a YunoHost application to the official Docker image."
tags: server linux docker
---

I started hosting a Matrix server aka Synapse server through YunoHost and while that has worked pretty well, YunoHost plugins are hard to maintain and issues are hard to debug. So I decided to migrate the Synapse installation to a Docker Compose setup that I have full control over. This guide explains how to migrate the server from the YunoHost plugin to the official Docker image. It was tested with YunoHost 11.2.31 and the Matrix/Synapse plugin 1.109.0~ynh1.

## Backup and Stop Old Server

First, create and download a backup of the Synapse server in YunoHost. Stop the server, but be aware that there is some downtime for your Matrix service. Be sure to stay logged in on at least one Matrix client device - you will need that access later to migrate from YunoHost SSO to password-based authentication.

```shell
sudo systemctl stop synapse.service
sudo yunohost backup create --apps synapse
```

The backup file might be very large, so make sure you have enough space! You can use `--dry-run` to get a size estimate in Bytes. The largest part of my backup were the log files. You might want to trim those first.

Next, unpack the backup file on the server you're setting up the new Docker-based server. The location of the backup files will be referred to as `${YUNOHOST_BACKUP_FILE}` in the instructions below.

## Setup New Server

First, create your `docker-compose.yml` based on the official guide: <https://github.com/element-hq/synapse/blob/develop/contrib/docker/README.md>. I assume that you mount the Synapse data directory to `./files` as shown in the sample file. Otherwise, adjust that path in the instructions below.

Now generate a fresh `homeserver.yaml` using the following command:

```shell
docker compose run --rm synapse generate
```

After that's done, add the following YAML code in the `database` section of your `homeserver.yaml` (adjust the database config to whatever you have defined in your `docker-compose.yml` for the database container):

```yaml
database:
  name: psycopg2
  args:
    user: synapse
    password: changeme
    dbname: synapse
    host: db
    cp_min: 5
    cp_max: 10
```

In your `homeserver.yaml`, also copy the following parameters from the old `${YUNOHOST_BACKUP_FILE}/apps/synapse/backup/etc/matrix-synapse/homeserver.yaml`:

* server_name
* enable_registration
* registration_shared_secret
* allow_guest_access
* macaroon_secret_key
* form_secret

Copy the file `${YUNOHOST_BACKUP_FILE}/apps/synapse/backup/etc/matrix-synapse/toastbrot.net.signing.key` to `./files/` and adapt the `signing_key_path` in your new `homeserver.yaml` accordingly.

Copy the contents of `${YUNOHOST_BACKUP_FILE}/apps/synapse/backup/home/yunohost.app/synapse/media/` to `./files/media_storage/`.

Do a `sudo chown -R 991:991 ./files/` to fix the file permissions.

In YunoHost, you're relying on the built-in mail server. When hosting a Synapse server on its own, you have to bring your own mail server. I recommend [docker-mailserver](https://github.com/docker-mailserver/docker-mailserver) if you're self-hosting, but any mail server will do. Adapt the `email` section to use a new mail server, check <https://element-hq.github.io/synapse/latest/usage/configuration/config_documentation.html#email> for details.

Now start up the Docker Compose database and restore the backup file from the YunoHost database, which is included in the Synapse backup:

```shell
docker compose up db -d
cat ${YUNOHOST_BACKUP_FILE}/apps/synapse/backup/dump.sql | docker compose exec -T db psql -U synapse
```

## Start and Test

Now you're ready to start the Synapse server:

```shell
docker compose up -d
```

Your new Synapse server should be up and running! Check the logs to be sure.

The last step is to change the DNS entries for your Matrix service to point to the new server so that clients and other servers can connect. Change your DNS entries to point to the new server. There is also an SRV entry that points to port 8448 by YunoHost convention - that one needs to be changed to 443 which is the Synapse default.

## Federation

Depending on your setup, you might also have to follow the instructions in <https://github.com/element-hq/synapse/blob/develop/docs/delegate.md> to make federation work. If your setup allows it, you can point the server name to the homeserver URL and add `serve_server_wellknown: true` to your `homeserver.yaml`.

Then use <https://federationtester.matrix.org> to test the federation setup.

## Password Reset

Since your Synapse account doesn't have an email or a password (it was using SSO on YunoHost), you have to set those. I recommend using an authenticated session in one of your Matrix clients.

Through your Matrix client, you can set an email address for your account. Then you have to elevate your account to admin status. Login to the database with `docker compose exec -it db psql -U synapse` and elevate your user to admin:

```sql
UPDATE users SET admin = 1 WHERE name = '@foo:bar.com';
```

Then you can use the [Admin API](https://element-hq.github.io/synapse/latest/usage/administration/admin_api/index.html) to set a new password and erase the old `external_ids` sections that contain the YunoHost SSO references.

Now you should be able to log in using username and password on a new client.

## Helpful Links

* <https://gist.github.com/matusnovak/37109e60abe79f4b59fc9fbda10896da>
* <https://www.gibiris.org/eo-blog/posts/2022/01/21_containterise-synapse-postgres.html>
* <https://github.com/YunoHost-Apps/synapse_ynh/blob/master/doc/ADMIN.md>
