# Astrid Intelligence Validator Installer

Automated setup and management for the Astrid Intelligence Validator stack.

This installer sets up the validator, Redis, and Watchtower services, and provides a CLI tool (`astrid`) for easy management: start, stop, update, configure, and monitor your validator from the command line.

## Quick Install

Run this command in your terminal:

```sh
bash -c "$(curl -sSfL https://raw.githubusercontent.com/astridintelligence/sn-127-install/master/install)"
```

## Components

- **Validator** — Runs `astridintelligence/astrid-validator:latest`
- **Redis** — Local instance for task coordination
- **Watchtower** — Automatically updates containers every 30 seconds

## Usage

The installer places a lightweight CLI tool called `astrid` in `/usr/local/bin`.

All commands are run as:

```sh
astrid <command> [options]
```

### Main Commands

- `astrid start` — Start all services (validator, redis, watchtower)
- `astrid stop` — Stop all services
- `astrid set --KEY "value" [--ANOTHER "value2"]` — Set environment variables in `.env` (e.g. mnemonic, API URL)
- `astrid status` — Show status and health of all services
- `astrid health` — Show detailed health and recent logs
- `astrid monitor [validator|redis|watchtower]` — Tail logs for a service
- `astrid update [--compose] [--install] [--pull] [--restart]` — Update CLI, docker-compose.yml, install script, pull images, and/or restart services

### Example Workflow

```sh
# 1. Install (if not already done)
bash -c "$(curl -sSfL https://raw.githubusercontent.com/astridintelligence/sn-127-install/master/install)"

# 2. Set your validator mnemonic (required)
astrid set --VALIDATOR_MNEMONIC "xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx"

# 3. Set your validator display name (required)
astrid set --VALIDATOR_DISPLAY_NAME "My Bittensor name"

# 4. Start the stack
astrid start

# 5. Check status
astrid status

# 6. View logs
astrid monitor validator
```

### Updating

To update the CLI and/or stack components:

```sh
# Update just the CLI
astrid update

# Update docker-compose.yml
astrid update --compose

# Update install script
astrid update --install

# Pull latest images
astrid update --pull

# Restart services after update
astrid update --restart
```

## Project Structure

After install, your project directory (default: `$HOME/astrid`) will contain:

```
.
├── astrid                # CLI helper (copied to /usr/local/bin)
├── install               # installer script (main entrypoint)
├── docker-compose.yml    # service definitions
├── .env                  # environment variables
└── README.md
```

## Environment Variables

The `.env` file (created in your project directory) supports:

- `VALIDATOR_MNEMONIC` (required)
- `VALIDATOR_DISPLAY_NAME` (required)
- `NODE_ENV` (default: production)
- `API_URL` (default: https://api.astrid.global/v1)
- `REDIS_URL` (default: redis://redis:6379)
- `HEARTBEAT_INTERVAL_MS` (default: 15000)
- `MAX_CONCURRENT_TASKS` (default: 2)

You can set these with `astrid set --KEY "value"`.

Example `.env`:

```env
VALIDATOR_MNEMONIC="your mnemonic here"
VALIDATOR_DISPLAY_NAME="validator name"
NODE_ENV=production
API_URL=https://api.astrid.global/v1
REDIS_URL=redis://redis:6379
HEARTBEAT_INTERVAL_MS=15000
MAX_CONCURRENT_TASKS=2
```

## Watchtower

Watchtower checks for new Docker image versions every 30 seconds and automatically redeploys containers labeled with:

```
com.centurylinklabs.watchtower.enable=true
```

## Requirements

- Docker with Compose plugin
- curl
- bash
- (optional) sudo for installing the astrid binary

## Cleanup

To remove all services and volumes:

```sh
astrid stop
docker volume rm sn-127-install_redis-data
```

## License

MIT — use freely at your own risk.
