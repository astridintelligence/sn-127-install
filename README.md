# Astrid Intelligence Validator Installer

Automated setup and management for the Astrid Intelligence Validator (Bittensor Subnet 127).

This installer sets up the validator and Watchtower services, and provides a CLI tool (`astrid`) for easy management: start, stop, update, configure, and monitor your validator from the command line.

The validator itself is a lightweight daemon: on an interval it fetches completed competitions from the Astrid Arena API, independently replays trade PnL, ranks the top 3 miners, and submits Bittensor weights on-chain. See the [source repository](https://github.com/astridintelligence/sn-127) for the full architecture.

## Quick Install

Run this command in your terminal:

```sh
bash -c "$(curl -sSfL https://raw.githubusercontent.com/astridintelligence/sn-127-install/master/install)"
```
## Source Code

- **Repository:** [https://github.com/astridintelligence/sn-127](https://github.com/astridintelligence/sn-127)

## Hardware Requirements

This stack runs as a **Node.js application packaged as a Docker image** (validator + Watchtower). You do not need to install Node.js locally — only Docker is required.

The validator is a single lightweight process: it polls the Arena API on an interval and occasionally signs/submits a Bittensor transaction. It does not run a task queue, a database, or sandboxed workloads, so requirements are modest.

### Minimum

- **Server type:** VPS or dedicated
- **CPU:** 1 vCPU
- **RAM:** 1–2 GB
- **Disk:** 10 GB SSD
- **Network:** Stable connection, public IPv4 recommended

### Recommended (headroom for smoother updates)

- **CPU:** 2 vCPU
- **RAM:** 4 GB
- **Disk:** 20 GB SSD

## Components

- **Validator** — Runs `astridintelligence/astrid-validator:latest`
- **Watchtower** — Automatically updates the validator container every 30 seconds

## Usage

The installer places a lightweight CLI tool called `astrid` in `/usr/local/bin`.

All commands are run as:

```sh
astrid <command> [options]
```

### Main Commands

- `astrid start` — Start all services (validator, watchtower)
- `astrid stop` — Stop all services
- `astrid set --KEY "value" [--ANOTHER "value2"]` — Set environment variables in `.env` (e.g. mnemonic, API URL)
- `astrid status` — Show status and health of all services
- `astrid health` — Show validator state and recent log lines
- `astrid monitor [validator|watchtower]` — Tail logs for a service
- `astrid update [--to <ref|sha>] [--pull] [--restart]` — Re-download the CLI, install script, and docker-compose.yml (pinned to `--to`, or the currently pinned ref), optionally pulling the latest image and restarting services

### Example Workflow

```sh
# 1. Install (if not already done)
bash -c "$(curl -sSfL https://raw.githubusercontent.com/astridintelligence/sn-127-install/master/install)"

# 2. Set your validator mnemonic or secret seed, and SS58 address (required)
astrid set --VALIDATOR_MNEMONIC "xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx-xxxx"
astrid set --VALIDATOR_SECRET_SEED "0x123..."
astrid set --VALIDATOR_SS58_ADDRESS "5YourSS58AddressHere"

# 3. Start the stack
astrid start

# 4. Check status
astrid status

# 5. View logs
astrid monitor validator
```

### Updating

`astrid update` always refreshes the `astrid` CLI, the `install` script, and `docker-compose.yml` to the target ref (the currently pinned ref by default). Use flags to also refresh the running containers:

```sh
# Refresh scripts only (no image pull, no restart)
astrid update

# Update to a specific ref/tag
astrid update --to v1.2.0

# Pull the latest validator image
astrid update --pull

# Pull and restart services
astrid update --pull --restart
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

- `VALIDATOR_MNEMONIC` or `VALIDATOR_SECRET_SEED` (required)
- `VALIDATOR_SS58_ADDRESS` (required)
- `VALIDATOR_SS58_FORMAT` (default: 42)
- `NODE_ENV` (default: production)
- `ARENA_API_URL` (default: https://arena-api.astrid.global)
- `BITTENSOR_ENABLED` (default: true)
- `BITTENSOR_WS_ENDPOINT` (default: Finney mainnet)
- `BITTENSOR_WEIGHT_INTERVAL_MS` (default: 3600000)
- `LOG_LEVEL` (default: info)
- `SLACK_API_TOKEN`, `SLACK_CHANNEL`, `SLACK_ERROR_CHANNEL`, `SLACK_INFO_CHANNEL` (optional, for Slack alerting)

You can set these with `astrid set --KEY "value"`.

Example `.env`:

```env
VALIDATOR_MNEMONIC="your mnemonic here"
VALIDATOR_SECRET_SEED="0x123..."
VALIDATOR_SS58_ADDRESS="5YourSS58AddressHere"
VALIDATOR_SS58_FORMAT=42

NODE_ENV=production
ARENA_API_URL=https://arena-api.astrid.global

BITTENSOR_ENABLED=true
BITTENSOR_WS_ENDPOINT=wss://entrypoint-finney.opentensor.ai:443
BITTENSOR_WEIGHT_INTERVAL_MS=3600000

LOG_LEVEL=info
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

To remove all services:

```sh
astrid stop
```

## License

MIT — use freely at your own risk.
