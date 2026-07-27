# Hermes Web UI

Docker Compose configuration for the community Hermes Web UI. Hermes Agent's
gateway runs on the host as a systemd user service; Docker runs only the Web UI.

## Prerequisites

- Docker with the Compose plugin
- An existing Hermes Agent configuration in `~/.hermes`

Install and configure Hermes Agent before starting the Web UI:

```bash
curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash
hermes model
```

## Start

Copy the example configuration if `.env` does not exist:

```bash
cp .env.example .env
```

On Linux or macOS, set `WANTED_UID` and `WANTED_GID` in `.env` to the
values returned by:

```bash
id -u
id -g
```

Install and start the host gateway once:

```bash
hermes gateway install
hermes gateway start
hermes gateway status
```

Then start the Dockerized frontend:

```bash
docker compose up -d --build --remove-orphans
```

The Web UI is built locally from `Dockerfile.webui` because it contains a
compatibility patch; it is not pulled as a `puda-hermes-webui` image from Docker
Hub. The `--build` option rebuilds that local image when the Dockerfile or its
upstream base image changes. `--remove-orphans` also removes the legacy Docker
gateway container if this repository was used before the host-gateway change.

Open <http://localhost:8787>.

### Tailscale access

To expose the Web UI only on this machine's Tailscale interface, find its
Tailscale IPv4 address and set it as the bind address in `.env`:

```bash
tailscale ip -4
# .env
HERMES_WEBUI_BIND_ADDRESS=100.x.y.z
HERMES_WEBUI_PASSWORD=replace-with-a-strong-password
```

Recreate the Web UI container, then open `http://100.x.y.z:8787` from another
device on the same tailnet:

```bash
docker compose up -d --force-recreate --remove-orphans hermes-webui
```

## Configuration

- The current user's `~/.hermes` directory is mounted into the Web UI container,
  so the host gateway and Web UI share configuration, credentials, sessions,
  skills, and memory.
- The host Hermes Agent source at `~/.hermes/hermes-agent` is mounted read-only
  so the Web UI can run its in-process chat agent without starting a second
  gateway.
- The current user's `~/workspace` directory is exposed in the Web UI file browser.
- `HERMES_WEBUI_BIND_ADDRESS` is the host interface address. It defaults to
  `127.0.0.1`; use the host's Tailscale IP for tailnet-only remote access.
- `HERMES_WEBUI_PORT` is the host port. It defaults to `8787`.
- `HERMES_WEBUI_PASSWORD` enables password authentication.

The port is bound to `127.0.0.1` by default. If you set
`HERMES_WEBUI_BIND_ADDRESS` to a network interface, set a strong
`HERMES_WEBUI_PASSWORD` first.

## Operations

```bash
docker compose logs -f hermes-webui
docker compose restart hermes-webui
docker compose down
hermes gateway status
hermes gateway restart
```

The Web UI runs browser chat in-process. The host systemd gateway independently
handles Telegram and other messaging platforms. Do not run `hermes gateway run`
inside another container with the same platform credentials, because duplicate
instances compete for the same polling connection.
