# Docker Deployment Guide

This guide provides technical information for deploying **nanobot** using Docker. The application is containerized to include both the Python core and the Node.js bridge (required for WhatsApp).

## Prerequisites

- [Docker Engine](https://docs.docker.com/engine/install/) installed on your host machine.

## Building the Image

To build the Docker image from the source code, run the following command in the root of the repository:

```bash
docker build -t nanobot .
```

This process will:
1.  Use `ghcr.io/astral-sh/uv:python3.12-bookworm-slim` as the base image.
2.  Install Node.js 20 (for the WhatsApp bridge).
3.  Install Python dependencies using `uv`.
4.  Build the Node.js bridge in `bridge/`.
5.  Expose port `18790`.

## Configuration & Persistence

The application stores its configuration and local data in `~/.nanobot` (inside the container: `/root/.nanobot`). To ensure your configuration (including API keys and session data) persists across container restarts, you **must** mount a volume.

We recommend mounting your host's `~/.nanobot` directory to the container's `/root/.nanobot`.

## Running the Application

### 1. Initialization (First Run)

If you haven't configured the application yet, run the onboarding wizard:

```bash
docker run -it --rm \
  -v ~/.nanobot:/root/.nanobot \
  nanobot onboard
```

This will guide you through creating the `config.json` file. Alternatively, you can manually create `~/.nanobot/config.json` on your host machine.

### 2. Gateway Mode (Daemon)

To run the main gateway (which handles Telegram, WhatsApp, and other channels):

```bash
docker run -d \
  --name nanobot \
  --restart unless-stopped \
  -v ~/.nanobot:/root/.nanobot \
  -p 18790:18790 \
  nanobot gateway
```

- **-d**: Run in detached mode.
- **--restart unless-stopped**: Automatically restart the container if it crashes or the host reboots.
- **-p 18790:18790**: Maps the gateway port.

### 3. CLI / Agent Mode

You can run one-off commands or chat with the agent directly via the CLI:

**Check Status:**
```bash
docker run --rm \
  -v ~/.nanobot:/root/.nanobot \
  nanobot status
```

**Chat with Agent:**
```bash
docker run --rm \
  -v ~/.nanobot:/root/.nanobot \
  nanobot agent -m "Hello, how are you?"
```

### 4. WhatsApp Linking

To link a WhatsApp account, you need to access the QR code. Run the following command and scan the QR code that appears in your terminal:

```bash
docker run -it --rm \
  -v ~/.nanobot:/root/.nanobot \
  nanobot channels login
```

## Technical Details

- **Base Image**: Python 3.12 Slim (Bookworm)
- **Runtime Dependencies**: Node.js 20, Python 3.12
- **Default Port**: 18790 (Gateway)
- **Workdir**: `/app`
- **Config Path**: `/root/.nanobot`

## Troubleshooting

- **Permission Issues**: Ensure the user running the Docker command has read/write access to the host's `~/.nanobot` directory.
- **Port Conflicts**: If port 18790 is in use, map it to a different host port (e.g., `-p 18791:18790`).
- **Bridge Errors**: Check logs with `docker logs nanobot`. If the bridge fails to start, ensure Node.js was installed correctly during the build (it is handled by the Dockerfile).
