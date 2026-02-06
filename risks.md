# Security Risks Assessment for Nanobot

This document outlines security risks identified during the technical assessment of the Nanobot framework.

## 1. High Severity Risks

### 1.1 Unauthenticated WhatsApp Bridge
-   **Description**: The WhatsApp bridge (Node.js service) exposes a WebSocket server that accepts commands to send messages and broadcasts all incoming messages.
-   **Vulnerability**: The WebSocket server (`bridge/src/server.ts`) does not implement any authentication mechanism.
-   **Impact**: Any local process (or remote user if the port is exposed) can connect to the bridge to read all private WhatsApp messages and send messages on behalf of the user.
-   **Mitigation**: Implement an authentication token mechanism between the Python client and the Node.js bridge. Bind the WebSocket server strictly to `127.0.0.1` by default.

### 1.2 Unrestricted Filesystem Access by Default
-   **Description**: The `Filesystem` tools (`read_file`, `write_file`, etc.) have a `restrict_to_workspace` parameter, but it defaults to `False` in the configuration.
-   **Vulnerability**: By default, the AI agent has read/write access to the entire filesystem of the host user.
-   **Impact**: Malicious prompts or hallucinated commands could read sensitive files (SSH keys, env vars) or delete/overwrite system files.
-   **Mitigation**: Change the default of `restrict_to_workspace` to `True` in `nanobot/config/schema.py`.

### 1.3 Arbitrary Code Execution (Design Feature)
-   **Description**: The agent is equipped with a `Shell` tool and `Cron` capabilities that allow executing arbitrary shell commands.
-   **Vulnerability**: While this is a feature, it represents a massive attack surface. The agent can execute any command the user can.
-   **Impact**: Complete system compromise if the agent is tricked (Prompt Injection) or malfunctions.
-   **Mitigation**:
    -   Run the agent in a sandboxed environment (Docker) by default.
    -   Require explicit user confirmation for high-risk commands.
    -   Implement a whitelist of allowed commands.

## 2. Medium Severity Risks

### 2.1 Plain Text API Keys
-   **Description**: API keys are stored in `~/.nanobot/config.json` in plain text.
-   **Vulnerability**: If the config file is read by an attacker (or the agent itself via `read_file`), keys are compromised.
-   **Impact**: Loss of funds (LLM credits), unauthorized usage of external services.
-   **Mitigation**: Use system keyring/credential manager or environment variables injected at runtime. Ensure config file permissions are strict (`0600`).

### 2.2 Unbounded Media Downloads
-   **Description**: The Telegram channel downloads all incoming photos, voice notes, and documents to `~/.nanobot/media`.
-   **Vulnerability**: There appear to be no limits on the file size or the total number/size of files downloaded.
-   **Impact**: Denial of Service (DoS) via disk space exhaustion. An attacker could flood the bot with large files.
-   **Mitigation**: Implement file size limits and retention policies (auto-delete after N days).

### 2.3 Lack of Rate Limiting
-   **Description**: The bot processes every message it receives immediately.
-   **Vulnerability**: No rate limiting on incoming messages.
-   **Impact**: DoS via message flooding, leading to high API costs (LLM usage) and potential service crash.
-   **Mitigation**: Implement a token-bucket rate limiter for incoming messages per user/chat.

## 3. Low Severity Risks

### 3.1 Potential ReDoS in Markdown Parsing
-   **Description**: The `_markdown_to_telegram_html` function uses several Regex substitutions to parse Markdown.
-   **Vulnerability**: Complex or malicious Markdown inputs could potentially cause Catastrophic Backtracking (ReDoS).
-   **Impact**: Temporary CPU spike / hanging of the Telegram polling loop.
-   **Mitigation**: Use a robust, non-regex-based Markdown parser (e.g., `markdown-it-py` with a custom renderer).

### 3.2 WebSocket Binding
-   **Description**: The `websockets` library and Node.js `ws` server may bind to `0.0.0.0` (all interfaces) by default if not explicitly set to `localhost`.
-   **Impact**: Exposes services to the local network or public internet if no firewall is present.
-   **Mitigation**: Explicitly bind to `127.0.0.1` in `nanobot/cli/commands.py` (for gateway) and `bridge/src/server.ts`.
