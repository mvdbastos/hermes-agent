# WhatsApp container deployment

This branch adds `docker-compose.whatsapp.yml`, a dedicated container definition for
Hermes' built-in Baileys WhatsApp bridge.

The WhatsApp bridge is not a separate Node service. Hermes' gateway starts and
supervises the bridge from inside `whatsapp-service`; the container therefore uses
the normal Hermes image build and runs `gateway run`.

## Build and start

From the repository root:

```bash
docker compose -f docker-compose.whatsapp.yml build --pull
docker compose -f docker-compose.whatsapp.yml up -d
```

On Windows PowerShell, the same commands work from the repository directory.

## Pair the WhatsApp account

Create a persistent data directory first if it does not exist, then run the
interactive setup command so the QR code is visible in the terminal:

```bash
docker compose -f docker-compose.whatsapp.yml run --rm --no-deps whatsapp-service whatsapp
```

Scan the displayed QR code from WhatsApp → Settings → Linked Devices. The
credentials are stored below the configured `HERMES_DATA_DIR` and must not be
committed or shared.

## Configuration

Copy the following settings into a local `.env` file next to the Compose file,
or set them in the host environment:

```dotenv
HERMES_UID=10000
HERMES_GID=10000
HERMES_DATA_DIR=./.hermes-whatsapp
WHATSAPP_ENABLED=true
WHATSAPP_MODE=bot
WHATSAPP_DM_POLICY=pairing
WHATSAPP_ALLOWED_USERS=
WHATSAPP_GROUP_POLICY=pairing
WHATSAPP_GROUP_ALLOWED_USERS=
WHATSAPP_REQUIRE_MENTION=false
```

For a controlled deployment, set `WHATSAPP_ALLOWED_USERS` to a comma-separated
list of phone numbers with country codes and no plus sign. Do not use an
allow-all value unless this is an intentional test deployment.

## Update the feature branch

The current development workflow builds on the Docker host:

```bash
git fetch origin whatsapp-service
git switch whatsapp-service
git reset --hard origin/whatsapp-service
docker compose -f docker-compose.whatsapp.yml build --pull
docker compose -f docker-compose.whatsapp.yml up -d --force-recreate
```

The eventual CI/CD workflow should replace the local build with a published
image and change the Compose file from `build:` to `image:`. The persistent
`HERMES_DATA_DIR` volume remains unchanged.
