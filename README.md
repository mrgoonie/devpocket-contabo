# DevPocket Server

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://python.org)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104+-green.svg)](https://fastapi.tiangolo.com)
[![Contabo](https://img.shields.io/badge/Contabo-VPS-0080FF.svg)](https://contabo.com)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **The mobile-first cloud IDE backend powered by Contabo VPS**

DevPocket Server is a production-ready FastAPI backend that provides secure, scalable development environments accessible from mobile devices. Each environment runs on a dedicated Contabo VPS with full SSH access, providing native performance and complete control.

## 🚀 Features

- **🔧 Dedicated VPS**: Each environment runs on its own Contabo VPS instance
- **👥 User Management**: Automated `dev` user creation with sudo privileges
- **🖥️ Full SSH Access**: Direct terminal access as `dev` user via WebSocket-to-SSH bridge
- **📱 Mobile-First**: Optimized for mobile development workflows
- **🔒 Secure**: JWT authentication, user password protection, SSH access
- **⚙️ Complete Dev Stack**: Pre-installed Docker, Git, Python, Node.js, AI coding tools
- **⚡ Fast Setup**: Cloud-init automated installation with progress tracking
- **📊 Scalable**: Auto-scaling VPS instances based on subscription tiers
- **🌍 Multi-Region**: Deploy VPS instances in Contabo data centers worldwide
- **💾 Persistent Storage**: Attached block storage volumes for data persistence

## 🏗️ Architecture

```mermaid
flowchart TD
   A[Mobile App] <--> B[FastAPI Server] <--> C[Contabo API]
   B --> D[WebSocket-SSH Bridge]
   D <--> E[VPS (SSH as 'dev' user)]
   E --> F[Development Environment]
   F --> F1[Ubuntu 22.04 LTS]
   F --> F2['dev' user with sudo access]
   F --> F3[Complete development stack]
   F --> F4[Persistent storage]
   F --> F5[AI coding tools]
```

## 🚀 Environment Creation Workflow

When you create a new environment, here's what happens automatically:

### 1. **VPS Provisioning** (30-60s)
- Contabo creates VPS instance in specified region/size
- Attaches persistent block storage volume
- Injects cloud-init script with user configuration

### 2. **User Setup** (30s)
- Creates `dev` user with your provided password
- Grants full sudo privileges (passwordless)
- Sets up SSH access and home directory
- Configures default shell and environment

### 3. **Development Stack Installation** (5-8 minutes)
```bash
# Core system tools
apt update && apt install -y curl wget git vim nano htop build-essential

# Docker installation
curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
usermod -aG docker dev

# Python 3.11+ with pip
apt install -y python3.11 python3-pip python3-venv

# Node.js via NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install --lts && nvm use --lts

# Package managers
npm install -g pnpm yarn pm2

# AI Coding Tools
npm install -g @anthropic/claude-cli
pip install google-generativeai
curl -sSL https://github.com/microsoft/vscode-server/releases/latest/download/vscode-server-linux-x64.tar.gz | tar -xz
```

### 4. **Environment Configuration** (1-2 minutes)
- tmux configuration for persistent sessions
- Git global configuration setup
- Environment variables and PATH updates
- SSH key management and permissions
- Final system optimizations

### 5. **Ready State**
Your environment is now ready with:
- SSH access as `dev` user
- Full sudo privileges
- All development tools installed and configured
- Persistent tmux sessions
- AI coding assistants ready to use

### Key Components

- **FastAPI Server**: REST API and WebSocket endpoints
- **Contabo Integration**: VPS and storage management
- **SSH Bridge**: WebSocket-to-SSH proxy for terminal access
- **PostgreSQL**: User data, environment metadata, templates
- **Authentication**: JWT with refresh tokens, Google OAuth
- **Rate Limiting**: In-memory rate limiting per user

## 🛠️ Tech Stack

- **Backend**: FastAPI, Python 3.11+
- **Database**: PostgreSQL (asyncpg with SQLAlchemy async)
- **Infrastructure**: Contabo VPS, Block Storage
- **Authentication**: JWT, OAuth2, bcrypt
- **Networking**: asyncssh, WebSockets
- **Monitoring**: structlog, health checks
- **Deployment**: Docker, docker-compose

## 📋 Prerequisites

- Python 3.11+
- PostgreSQL (local or cloud)
- Contabo account with API credentials
- SSH key pair for VPS access

## ⚡ Quick Start

### 1. Clone and Setup

```bash
git clone https://github.com/yourusername/devpocket-server-do.git
cd devpocket-server-do

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 2. Environment Configuration

Create `.env` file:

```env
# Database
DATABASE_URL=postgresql+asyncpg://username:password@localhost:5432/devpocket_server

# Security
SECRET_KEY=your-secret-key-here
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7

# Contabo
CONTABO_CLIENT_ID=your-contabo-client-id
CONTABO_CLIENT_SECRET=your-contabo-client-secret
CONTABO_USERNAME=your-contabo-username
CONTABO_PASSWORD=your-contabo-password
CONTABO_DEFAULT_REGION=EU
CONTABO_DEFAULT_SIZE=VPS-S-1vcpu-2gb
CONTABO_SSH_KEY_ID=your-ssh-key-id

# Email (optional)
RESEND_API_KEY=your-resend-api-key
FROM_EMAIL=noreply@yourdomain.com

# CORS
ALLOWED_ORIGINS=http://localhost:3000,https://yourdomain.com
```

### 3. Database Setup

```bash
# Start PostgreSQL (if running locally)
sudo systemctl start postgresql

# Initialize database (create tables and indexes)
python -c "from app.core.database import init_database; import asyncio; asyncio.run(init_database())"
```

### 4. Load Templates

```bash
# Load default environment templates
python scripts/load_templates.py
```

### 5. Run Development Server

```bash
# Start the server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

The API will be available at `http://localhost:8000`

- **API Documentation**: `http://localhost:8000/docs`
- **Health Check**: `http://localhost:8000/health`

## 📁 Project Structure

```
app/
├── main.py                 # FastAPI application entry point
├── core/                   # Core infrastructure
│   ├── config.py           # Settings and configuration
│   ├── database.py         # PostgreSQL connection and SQLAlchemy setup
│   ├── security.py         # JWT, password hashing, security
│   └── logging.py          # Structured logging
├── models/                 # SQLAlchemy ORM models and Pydantic schemas
│   ├── user.py             # User and authentication models
│   └── environment.py      # Environment and VPS models
├── services/               # Business logic layer
│   ├── auth_service.py     # Authentication and user management
│   ├── environment_service.py  # Environment lifecycle management
│   └── contabo_service.py # Contabo API integration
├── api/                    # HTTP and WebSocket endpoints
│   ├── auth.py             # Authentication endpoints
│   ├── environments.py     # Environment CRUD operations
│   ├── websocket.py        # Terminal and logs WebSocket
│   └── users.py            # User management endpoints
└── middleware/             # Request/response middleware
    ├── auth.py             # JWT validation and user context
    └── rate_limiting.py    # Rate limiting middleware

docs/                       # Documentation
scripts/                    # Utility scripts and templates
tests/                      # Test suites
```

## 📡 API Endpoints

### Authentication
- `POST /api/v1/auth/register` - User registration
- `POST /api/v1/auth/login` - User login
- `POST /api/v1/auth/refresh` - Refresh access token
- `GET /api/v1/auth/me` - Get current user

### Environments
- `POST /api/v1/environments` - Create new environment (VPS)
- `GET /api/v1/environments` - List user environments
- `GET /api/v1/environments/{id}` - Get environment details
- `POST /api/v1/environments/{id}/start` - Power on VPS
- `POST /api/v1/environments/{id}/stop` - Power off VPS
- `POST /api/v1/environments/{id}/snapshot` - Create VPS snapshot
- `POST /api/v1/environments/{id}/resize` - Resize VPS

### Contabo Resources
- `GET /api/v1/regions` - List available regions
- `GET /api/v1/sizes` - List VPS sizes and pricing

### WebSocket
- `WS /api/v1/ws/terminal/{id}` - SSH terminal access
- `WS /api/v1/ws/logs/{id}` - Cloud-init installation logs

## 🌐 WebSocket Integration

### Terminal Access

Connect to environment terminal via WebSocket:

```javascript
const ws = new WebSocket('ws://localhost:8000/api/v1/ws/terminal/ENV_ID?token=JWT_TOKEN');

// Connection established - logged in as 'dev' user with full access
ws.onopen = () => {
  console.log('Connected as dev user with sudo privileges');
};

// Send command (runs as dev user)
ws.send(JSON.stringify({
  type: 'input',
  data: 'docker ps\\n'  // Can run docker commands
}));

// Receive output
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  if (message.type === 'welcome') {
    console.log('Available tools:', message.environment.installed_tools);
  } else if (message.type === 'output') {
    console.log(message.data);
  }
};

// Resize terminal
ws.send(JSON.stringify({
  type: 'resize',
  cols: 80,
  rows: 24
}));
```

### Installation Logs

Monitor environment setup progress:

```javascript
const ws = new WebSocket('ws://localhost:8000/api/v1/ws/logs/ENV_ID?token=JWT_TOKEN');

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  switch (message.type) {
    case 'installation_log':
      console.log('Log:', message.data);
      break;
    case 'installation_complete':
      console.log('Installation completed!');
      break;
    case 'installation_error':
      console.error('Installation failed:', message.error);
      break;
  }
};
```

## 🐳 Docker Deployment

### Development

```bash
# Build and run with docker-compose
docker-compose up --build

# Run in background
docker-compose up -d
```

### Production

```bash
# Production build
docker-compose -f docker-compose.prod.yml up --build -d

# With SSL (using Traefik/nginx)
docker-compose -f docker-compose.prod.yml -f docker-compose.ssl.yml up -d
```

## 🔧 Configuration

### Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql+asyncpg://user:pass@localhost:5432/devpocket_server` |
| `SECRET_KEY` | JWT secret key | - |
| `CONTABO_CLIENT_ID` | Contabo API client ID | - |
| `CONTABO_CLIENT_SECRET` | Contabo API client secret | - |
| `CONTABO_USERNAME` | Contabo username | - |
| `CONTABO_PASSWORD` | Contabo password | - |
| `CONTABO_DEFAULT_REGION` | Default VPS region | `EU` |
| `CONTABO_DEFAULT_SIZE` | Default VPS size | `VPS-S-1vcpu-2gb` |
| `CONTABO_SSH_KEY_ID` | SSH key ID | - |
| `DEV_USER_DEFAULT_SHELL` | Default shell for dev user | `/bin/bash` |
| `INSTALL_TIMEOUT_MINUTES` | Cloud-init timeout | `15` |
| `ALLOWED_ORIGINS` | CORS allowed origins | `*` |

### Subscription Tiers

| Tier | Price | VPS Instances | Specs | Storage | Pre-installed Stack |
|------|-------|----------|-------|---------|--------------------|
| Starter | $19/mo | 3 | 1 vCPU, 1GB RAM | 25GB SSD | Full dev stack + AI tools |
| Pro | $99/mo | Unlimited | 2 vCPU, 2GB RAM | 60GB SSD | Full dev stack + AI tools |

**Development Stack (All Tiers)**:
- Ubuntu 22.04 LTS with `dev` user (sudo access)
- Docker, Git, Curl, Python 3.11+, Node.js LTS (NVM)
- Package managers: pip, npm, pnpm, yarn
- AI Tools: Claude Code CLI, Google Gemini CLI, Open Code
- Terminal: tmux with development-optimized configuration

## 🧪 Testing

```bash
# Run all tests
pytest

# Run specific test file
pytest tests/test_auth.py

# Run with coverage
pytest --cov=app

# Run integration tests
pytest tests/integration/
```

## 📊 Monitoring

### Health Checks

- `GET /health` - Service health with detailed information
- `GET /health/live` - Simple liveness check

### Logging

Structured logging with contextual information:

```python
import structlog
logger = structlog.get_logger()

logger.info("Environment created", 
           user_id=user_id, 
           environment_id=env_id,
           vps_id=vps.id)
```

### Metrics

Key metrics to monitor:
- Environment creation success rate
- WebSocket connection stability
- VPS provisioning time
- API response times
- User authentication errors

## 🚀 Deployment

### Contabo Deployment

1. Fork this repository
2. Set up Contabo VPS instance
3. Set environment variables
4. Deploy using docker-compose or manual setup

### Manual Deployment

1. Set up PostgreSQL (managed or self-hosted)
2. Configure environment variables
3. Run database migrations
4. Deploy with docker-compose or Kubernetes
5. Set up SSL certificates (Let's Encrypt recommended)
6. Configure monitoring and alerting

## 🔒 Security

- **Authentication**: JWT with refresh tokens
- **Authorization**: Role-based access control
- **Rate Limiting**: Per-user request limiting
- **CORS**: Configurable cross-origin policies
- **SSH Keys**: Secure VPS access
- **Data Encryption**: Sensitive data encrypted at rest
- **Input Validation**: Comprehensive request validation

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

### Development Setup

```bash
# Install development dependencies
pip install -r requirements-dev.txt

# Install pre-commit hooks
pre-commit install

# Run linting
flake8 app/
black app/
isort app/

# Run type checking
mypy app/
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- 📖 **Documentation**: [docs/](docs/)
- 🐛 **Issues**: [GitHub Issues](https://github.com/yourusername/devpocket-server-do/issues)
- 💬 **Discussions**: [GitHub Discussions](https://github.com/yourusername/devpocket-server-do/discussions)
- 📧 **Email**: support@devpocket.app

## 🎯 Roadmap

### Phase 1 - Core Features ✅
- [x] User authentication and authorization
- [x] Contabo VPS management
- [x] SSH terminal access via WebSocket
- [x] Environment templates and cloud-init

### Phase 2 - Enhanced Experience
- [ ] Real-time collaboration
- [ ] File management API
- [ ] Database integration templates
- [ ] CI/CD pipeline templates
- [ ] Mobile app integration

### Phase 3 - Enterprise Features
- [ ] Team management
- [ ] Advanced monitoring and analytics
- [ ] Custom domain support
- [ ] SSO integration
- [ ] Audit logging

## 🌟 Acknowledgments

- **FastAPI** - Modern Python web framework
- **Contabo** - Cloud infrastructure provider
- **PostgreSQL** - Relational database with async support
- **Tmux** - Terminal multiplexer for session persistence

---

**Built with ❤️ for mobile developers who refuse to be limited by hardware.**