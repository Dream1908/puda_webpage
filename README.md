# Hermes Web UI

Docker Compose configuration for Hermes Agent and the community Hermes Web UI.

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

Then start the frontend:

```bash
docker compose pull
docker compose up -d --build
```

`docker compose pull` updates the published Hermes Agent image. The Web UI is
built locally from `Dockerfile.webui` because it contains a compatibility patch;
it is not pulled as a `puda-hermes-webui` image from Docker Hub. The `--build`
option rebuilds that local image when the Dockerfile or its upstream base image
changes.

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
docker compose up -d --force-recreate hermes-webui
```

## Configuration

- The current user's `~/.hermes` directory is mounted into both containers.
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
docker compose logs -f hermes-agent
docker compose restart hermes-webui
docker compose down
```
