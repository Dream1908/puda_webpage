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

Open <http://localhost:8787>.

## Configuration

- The current user's `~/.hermes` directory is mounted into both containers.
- The current user's `~/workspace` directory is exposed in the Web UI file browser.
- `HERMES_WEBUI_PORT` is the host port. It defaults to `8787`.
- `HERMES_WEBUI_PASSWORD` enables password authentication.

The port is bound to `127.0.0.1` by default. If you change `compose.yaml` to
expose it on a network interface, set a strong `HERMES_WEBUI_PASSWORD` first.

## Operations

```bash
docker compose logs -f hermes-webui
docker compose logs -f hermes-agent
docker compose restart hermes-webui
docker compose down
```
