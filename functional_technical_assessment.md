# Functional and Technical Assessment of Nanobot

## 1. Project Overview

**Nanobot** is a lightweight, personal AI assistant framework designed to be research-ready and extensible. It aims to provide a core agent loop with modular skills, memory management, and multi-channel communication capabilities.

-   **Version**: 0.1.3.post4
-   **License**: MIT
-   **Core Philosophy**: "Small code, big capabilities" (~4000 lines of code).

## 2. Functional Assessment

### 2.1 Core Capabilities
-   **Agent Loop**: The central brain (`AgentLoop`) processes incoming messages, maintains context, and decides on actions (tool use or response). It supports multi-step reasoning (`max_iterations`).
-   **Context Management**:
    -   Builds a dynamic system prompt including Identity (`SOUL.md`), Tools (`TOOLS.md`), Memory (`MEMORY.md`), and active Skills.
    -   Maintains conversation history per session.
-   **Memory**:
    -   **Short-term**: In-memory conversation history.
    -   **Long-term**: File-based persistent memory (`workspace/memory/MEMORY.md` and daily notes).
    -   **Tools**: Agent can read/write to its memory files.
-   **Identity**: Customizable via Markdown files (`SOUL.md`, `AGENTS.md`) in the workspace.

### 2.2 Skills System
-   **Modular Design**: Skills are self-contained directories in `nanobot/skills` or `workspace/skills`.
-   **Definition**: Defined via `SKILL.md` with YAML frontmatter containing metadata (name, description, requirements).
-   **Dynamic Loading**: Skills can be loaded on-demand or set to "always loaded".
-   **Requirements Checking**: The system checks for required system binaries or environment variables before enabling a skill.

### 2.3 Communication Channels
Nanobot supports multiple interfaces for user interaction:
-   **CLI**: Direct terminal interaction.
-   **Telegram**: Full-featured bot using long-polling. Supports text, photos, voice (with transcription), and documents.
-   **WhatsApp**: Implemented via a separate bridge (see Technical Assessment).
-   **Other**: Architecture supports adding more channels (Discord, Feishu, etc.).

### 2.4 Extensibility
-   **Tool Registry**: Central registry for registering and executing tools.
-   **Built-in Tools**:
    -   `FileSystem` (Read, Write, Edit, List)
    -   `Shell` (Execute commands - dangerous but powerful)
    -   `Web` (Search, Visit pages)
    -   `Cron` (Scheduling)
    -   `Spawn` (Sub-agents)

## 3. Technical Assessment

### 3.1 Architecture
The project follows a modular, event-driven architecture:
-   **Message Bus (`nanobot.bus`)**: Decouples channels from the agent. Channels publish `InboundMessage`, Agent processes and publishes `OutboundMessage`.
-   **Service Layer**: Independent services for Cron, Heartbeat, and Channels.
-   **Dependency Injection**: Heavy use of dependency injection for passing configuration and services (e.g., `Provider`, `MessageBus`).

### 3.2 Tech Stack
-   **Language**: Python 3.11+ (leveraging modern `asyncio` features).
-   **LLM Interface**: `litellm` library provides a unified interface for OpenAI, Anthropic, OpenRouter, etc.
-   **CLI**: `typer` and `rich` for a polished command-line experience.
-   **Data Validation**: `pydantic` for configuration and data modeling.
-   **Networking**: `httpx` and `websockets` for async I/O.

### 3.3 WhatsApp Bridge
-   **Architecture**: Microservice pattern.
-   **Technology**: Node.js / TypeScript.
-   **Library**: `@whiskeysockets/baileys` (Reverse engineered WhatsApp Web API).
-   **Communication**: WebSocket connection between Python `WhatsAppChannel` and Node.js `BridgeServer`.
-   **Protocol**: JSON-based message passing over WebSocket.

### 3.4 Data Storage
-   **Configuration**: `~/.nanobot/config.json` (Pydantic settings).
-   **State**: Local filesystem.
    -   **Memory**: Markdown files.
    -   **Media**: `~/.nanobot/media`.
    -   **Cron Jobs**: JSON file (`jobs.json`).
-   **Database**: No SQL/NoSQL database used; relies on the filesystem for simplicity.

### 3.5 Deployment
-   **Docker**: `Dockerfile` provided for containerized deployment.
-   **Local**: Can run directly on the host machine (Mac/Linux preferred).

## 4. Key Observations
-   **Code Quality**: Clean, typed Python code with good separation of concerns.
-   **Simplicity**: Avoids complex vector databases or orchestration frameworks in favor of simple file-based context and direct LLM calls.
-   **Power vs. Safety**: Prioritizes capability (shell access, unrestricted filesystem by default) over safety, making it suitable for personal use by developers but risky for shared/production environments without hardening.
