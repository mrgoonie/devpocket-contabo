# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DevPocket Server is a production-ready FastAPI backend for a mobile-first cloud IDE. It provides secure, scalable development environments accessible from mobile devices using Contabo VPS instances. The system manages user authentication, development environments (VPS with SSH access), and real-time terminal access via WebSocket-to-SSH bridge.

## Architecture

The application follows a layered architecture with clear separation of concerns:

```
app/
├── main.py              # FastAPI application entry point with middleware
├── core/                # Core infrastructure components
│   ├── config.py        # Pydantic settings with environment variables
│   ├── database.py      # PostgreSQL async connection and SQLAlchemy setup
│   ├── security.py      # JWT, password hashing, security headers
│   └── logging.py       # Structured logging with structlog
├── models/              # SQLAlchemy ORM models and Pydantic schemas
│   ├── user.py          # User, authentication, and subscription models
│   └── environment.py   # Development environment and resource models
├── services/            # Business logic layer
│   ├── auth_service.py  # User authentication, Google OAuth, account lockout
│   └── environment_service.py  # Environment lifecycle, WebSocket sessions
├── api/                 # HTTP/WebSocket route handlers
│   ├── auth.py          # Authentication endpoints (register, login, OAuth)
│   ├── environments.py  # Environment CRUD operations
│   └── websocket.py     # Real-time terminal and log streaming
└── middleware/          # Request/response middleware
    ├── auth.py          # JWT token validation and user context
    └── rate_limiting.py # In-memory rate limiting
```

## Key Architectural Patterns

**Dependency Injection**: FastAPI's `Depends()` system is used throughout for database connections, authentication, and service injection. Services require `set_database()` calls to initialize database connections.

**Async/Await**: All I/O operations are asynchronous using asyncpg with SQLAlchemy (async PostgreSQL driver), httpx for external API calls, and asyncssh for SSH connections to VPS instances.

**Service Layer Pattern**: Business logic is separated into service classes (`auth_service`, `environment_service`, `contabo_service`) that are injected into API routes.

**Settings Management**: Configuration uses Pydantic Settings with environment variable support. Settings are centralized in `app.core.config.Settings`, including Contabo API credentials and configurations.

**Security Middleware Chain**: Multiple middleware layers handle security (rate limiting, CORS, security headers) before requests reach route handlers.

**SSH Bridge Pattern**: WebSocket connections are bridged to SSH connections for direct VPS access, providing native terminal experience.

## Environment Management with Contabo

The application uses Contabo VPS instances for development environments:

**VPS Architecture**: Each environment is a dedicated Contabo VPS instance with full SSH access, persistent block storage, and cloud-init configuration.

**Environment Lifecycle**:
- VPS instances are created via Contabo API with cloud-init scripts
- SSH keys are injected for secure access
- Tmux sessions persist across connections in `/home/devpocket/.tmux/`
- Block storage volumes provide data persistence

**Cloud-Init Initialization**: Startup scripts are embedded in cloud-init user-data, including package installation, tmux setup, and environment configuration.

**SSH Bridge**: WebSocket connections are bridged to SSH connections, providing direct PTY access to the VPS terminal.

**Template Management**: Templates are stored as YAML files in `./scripts/templates/` with cloud-init configurations and can be loaded into the database using `scripts/load_templates.py`.

## Contabo Integration Details

**API Credentials**: Stored in `CONTABO_CLIENT_ID`, `CONTABO_CLIENT_SECRET`, `CONTABO_USERNAME`, `CONTABO_PASSWORD` environment variables
**SSH Keys**: Managed per-user and injected into VPS instances during creation
**Regions**: Configurable default region with support for all Contabo data centers
**Sizes**: Mapped to subscription tiers (starter: VPS-S-1vcpu-2gb, pro: VPS-M-2vcpu-4gb)
**Volumes**: Attached block storage for persistent data

## Development rules

- always update the related docs in `./docs` folder if the code changes affect the docs
- use `source venv/bin/activate` to activate the virtual environment before running any command
