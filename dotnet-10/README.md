# .NET 10 Development Environment (DevContainer + Docker Compose)

Reproducible development environment for **.NET 10** applications powered by **VS Code DevContainers** and **Docker Compose**, featuring ready-to-use local infrastructure and a multi-stage `Dockerfile` optimized for production.

## Prerequisites

* [Docker Engine](https://docs.docker.com/engine/install/) / Docker Desktop with Docker Compose v2
* [Visual Studio Code](https://code.visualstudio.com/)
* [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension

## Project Structure & Convention

The `Dockerfile` follows a single entry-point convention so you can build and publish without modifying it:

```text
.
├── .devcontainer/
│   └── devcontainer.json
├── src/
│   └── App/
│       └── App.csproj        # Main entry-point project
├── .env                      # Local environment variables (do not commit)
├── .env.example              # Environment variables template
├── compose.yaml              # Development service (app)
├── infrastructure.yaml       # Infrastructure services
└── Dockerfile                # Multi-stage (development, build, final)
```

> **Important:** For the production build to work out of the box, the executable project must be located at `src/App/App.csproj`.

## Quick Start

1. **Configure environment variables:**
   ```bash
   cp .env.example .env
   ```
   *(Adjust `APP_NAME`, ports, or passwords if needed).*

2. **Open the project in the DevContainer:**
   * Open the folder in VS Code.
   * Press `Ctrl + Shift + P` and run **`Dev Containers: Reopen in Container`**.

3. **Verify the environment** inside VS Code's integrated terminal:
   ```bash
   dotnet --version
   node -v
   ```

## Included Services

All services share the `${APP_NAME}-network` Docker network. From your .NET application inside the container, connect using the **Internal Host** (service name) and its internal port.

| Service | Image | Internal Host | Local Port (`.env`) | Default Access / Credentials |
| :--- | :--- | :--- | :--- | :--- |
| **App (.NET 10)** | `devcontainers/dotnet:10.0` | `app` | `5247` | `http://localhost:5247` |
| **SQL Server 2025** | `mssql/server:2025-latest` | `mssql` | `1433` | User: `sa` / Pass: `D3faultP@ssw0rd` |
| **PostgreSQL 18** | `postgres:18.4-alpine` | `postgres` | `5432` | DB: `defaultdb` / User: `postgres` / Pass: `D3faultP@ssw0rd` |
| **Redis 8** | `redis:8.8.0-alpine` | `redis` | `6379` | No password by default |
| **RabbitMQ 4** | `rabbitmq:4.3.2-management-alpine` | `rabbitmq` | `5672` / `15672` | UI: `http://localhost:15672` (`guest` / `guest`) |
| **Adminer** | `adminer:5.4.2` | `adminer` | `8080` | UI: `http://localhost:8080` |
| **pgAdmin 4** | `dpage/pgadmin4:9.16` | `pgadmin` | `5050` | UI: `http://localhost:5050` (`pgadmin@example.com` / `D3faultP@ssw0rd`) |
| **Redis Insight** | `redis/redisinsight:3.6` | `redis-insight` | `5540` | UI: `http://localhost:5540` |

> **Data Persistence:** Database storage, message queues, and the NuGet package cache (`/home/vscode/.nuget/packages`) are persisted using Docker named volumes.

## Production Build

By default, the `Dockerfile` builds the `final` stage using `mcr.microsoft.com/dotnet/aspnet:10.0-noble-chiseled` (*distroless* image, no shell, non-root `$APP_UID` user, listening on port `8080`):

```bash
docker build -t my-app:latest .
docker run -p 8080:8080 my-app:latest
```

## Optional: Terminal AI Agents (OAuth)

Node.js is not included by default to keep the .NET DevContainer lightweight. If you want to use terminal-based AI agents (such as Gemini CLI, Claude Code, Codex CLI, or OpenCode), enable the Node.js feature first and install only the CLIs you need.

### 1. Enable Node.js in `.devcontainer/devcontainer.json`

Add the Node.js feature to your `features` block and rebuild the container (`Dev Containers: Rebuild Container`):

```json
"features": {
  "ghcr.io/devcontainers/features/node:2": {}
}
```

### 2. Install your preferred AI CLI

Once inside the container terminal, install only the agent(s) you use:

```bash
# Install only the ones you use
npm install -g @google/gemini-cli
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g opencode-ai
```

### 3. Persist OAuth Sessions Across Container Rebuilds

To keep your OAuth logins and global instruction files (`CLAUDE.md`, etc.) when rebuilding the container, add only the relevant folders for your agent(s) to `.devcontainer/devcontainer.json`:

```json
"initializeCommand": "mkdir -p ${localEnv:HOME}/.gemini ${localEnv:HOME}/.claude ${localEnv:HOME}/.codex ${localEnv:HOME}/.config/opencode ${localEnv:HOME}/.local/share/opencode",
"mounts": [
    "source=${localEnv:HOME}/.claude,target=/home/vscode/.claude,type=bind,consistency=cached",
  "source=${localEnv:HOME}/.gemini,target=/home/vscode/.gemini,type=bind,consistency=cached",
  "source=${localEnv:HOME}/.codex,target=/home/vscode/.codex,type=bind,consistency=cached",
  "source=${localEnv:HOME}/.config/opencode,target=/home/vscode/.config/opencode,type=bind,consistency=cached",
  "source=${localEnv:HOME}/.local/share/opencode,target=/home/vscode/.local/share/opencode,type=bind,consistency=cached"
]
```