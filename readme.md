# Storj satellite documentation

[Storj labs](https://storj.io) does not provide documentation for running it's satellite software. I would like to run my own satellite, here I am writing down my thoughts as a try to figure out how it works. Please contribute if you know something!!

## Software and dependencies

Copy the `docker-compose.yaml` file from the repo and optionally adjust some parameters.

For the satellite version tag, go to [docker hub](https://hub.docker.com/r/storjlabs/satellite/tags) and use the search box to find the tag for the latest stable release (these correspond which storage node version numbers, see the github releases page)

Start the satellite to generate folders and configuration files. It will crash because it's not configured yet.

Use `docker-compose up -d` to start everything

Use `docker-compose stop` to stop the containers and `docker-compose logs -f` to view the logs.

## Identity

TODO

## Satellite database settings

In the satellite data folder, edit `satellite/config.yaml`.

* `database` should be set to `postgres://postgres:YOURPASSWORD@postgres/satellite?sslmode=disable`
* `live-accounting.storage-backend` should be set to `redis://redis:6379?db=0`
* `server.revocation-dburl` can be left default (sqlite database)

Now, enable the environment variable `SATELITE_MIGRATE` and let the satellite create the databases. This env var should also be enabled when upgrading to a newer version.

The logs should show something like this:

```log
INFO    Configuration loaded    {"Location": "/root/.local/share/storj/satellite/config.yaml"}
INFO    migration.migrate.105   [...]
[...]
INFO    migration.migrate.112   [...]
INFO    migration.migrate       Database Version        {"version": 112}
```

## Running the satellite

Now, restart the satellite without `SATELLITE_MIGRATE`

```log
INFO    Configuration loaded    {"Location": "/root/.local/share/storj/satellite/config.yaml"}
INFO    Version info    {"Version": "1.6.4", "Commit Hash": "5e495f4b3bee91d6e6ac2fa43fc5034192146ab3", "Buil
INFO    Telemetry enabled
INFO    payments.stripe:clearing        running account balance update cycle
INFO    payments.stripe:clearing        running transactions update cycle
INFO    accounting:rollup       Rollup found no new tallies
INFO    accounting:rollup       Rollup found no new bw rollups
INFO    accounting:rollup       RollupStats is empty
INFO    payments.stripe:clearing        running account balance update cycle
INFO    payments.stripe:clearing        running transactions update cycle
INFO    payments.stripe:clearing        running account balance update cycle
```

The satellite can be started in different "modes". I guess that in production you will run multiple satellite instances at once, for example the main satellite, admin satellite and API satellite continuously and the GC and repair satellite instances on a cronjob.

### Admin

Set env var `SATELLITE_ADMIN: 'true'` and configure `admin.address` in `config.yaml`, for example to `0.0.0.0:7779`

Start the satellite, it will say

```log
INFO    Configuration loaded    {"Location": "/root/.local/share/storj/satellite/config.yaml"}
INFO    Telemetry enabled
```

Going to `address:7779` gives

> Authorization not enabled.

What is authorization and how to enable?

### Repair

`SATELLITE_REPAIR`

### API

`SATELLITE_API`

### Garbage collection

`SATELLITE_GC`

### Get nodes to use this satellite

???

<!-- ### How to "access" the satellite for users and admins?

I think there should be some web interface like europe-west-1.tardigrade.io.

Running `netstat -tulpn` in the satellite container shows

```text
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 127.0.0.11:34287        0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:37649         0.0.0.0:*               LISTEN      1/satellite
udp        0      0 127.0.0.11:37895        0.0.0.0:*                           -
```

even though config.yml has

```yaml
admin.address: 0.0.0.0:7779
server.address: 0.0.0.0:7777
server.private-address: 0.0.0.0:7778
console.address: 0.0.0.0:10100
marketing.address: 0.0.0.0:8090
``` -->
