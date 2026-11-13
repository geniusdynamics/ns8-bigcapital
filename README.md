# ns8-bigcapital

Online accounting software, built to automate business financial processes.

Module id: `bigcapital`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/bigcapital:latest 1
```

The command returns the instance id, for example `bigcapital1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/bigcapital1 --data - <<'EOF'
{
  "host": "bigcapital.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'bigcapital.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/redis` | `8.10.3` |
| `docker.io/mariadb` | `13.0.2` |
| `docker.io/mongo` | `9.0.8` |
| `docker.io/bigcapitalhq/webapp` | `v0.22.0` |
| `docker.io/bigcapitalhq/server` | `v0.22.0` |
| `docker.io/geniusdynamics/bigcapital-migrations` | `latest` |
| `docker.io/geniusdynamics/bigcapital-webapp` | `v1.0.0` |

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/bigcapital.sql`
- `volumes/bigcapital-app`
- `volumes/mongo-app`
- `volumes/mongo-configdb`
- `volumes/mysql-data`
- `volumes/envoy`

## Services

- `bigcapital-migrations.service`
- `bigcapital-server.service`
- `bigcapital-webapp.service`
- `bigcapital.service`
- `mariadb-app.service`
- `mongo-app.service`
- `redis-app.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/bigcapital1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/bigcapital:latest","instances":["bigcapital1"],"force":true}'
remove-module --no-preserve bigcapital1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- mariadb-app.service: uses %S/state, kickstart uses %E/state
- bigcapital-server.service: uses %S/state, kickstart uses %E/state
- README still has the template boilerplate
- mariadb-app.service: %S/state changed to %E/state (review the bind mounts that remain)
- bigcapital-server.service: %S/state changed to %E/state (review the bind mounts that remain)
- imageroot/systemd/user/bigcapital-server.service: smarthost lines removed, check the unit still starts
- BIGCAPITAL_MIGRATIONS_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- BIGCAPITAL_WEBAPP_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- MONGO_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- REDIS_IMAGE is used by the module but no image in the label provides it: leftover from the template?
- SERVER_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://docs.bigcapital.app/

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
