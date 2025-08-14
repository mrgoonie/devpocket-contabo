# DevPocket API Endpoints

## Overview
This document lists all available API endpoints in the DevPocket server.

## Base URL
- Development: `http://localhost:8000`
- Production: Your deployed server URL

## Authentication
Most endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

## Health Check Endpoints

### GET /health
Health check endpoint
- **Authentication:** Not required
- **Returns:** Service health status with version and environment info


### GET /health/live
Liveness check for service
- **Authentication:** Not required
- **Returns:** Basic service availability status

### GET /api/v1/info
API information endpoint
- **Authentication:** Not required
- **Returns:** API metadata and version information

## Authentication API

### POST /api/v1/auth/register
Register a new user account
- **Authentication:** Not required
- **Body:** User registration data (username, email, password, fullName)
- **Returns:** Created user object with verification message
- **Status:** 201 Created

### POST /api/v1/auth/login
User login with email or username
- **Authentication:** Not required
- **Body:** Login credentials (usernameOrEmail, password)
- **Returns:** JWT tokens (access_token, refresh_token) and user info
- **Status:** 200 OK

### POST /api/v1/auth/google (Coming Soon)
Google OAuth login
- **Authentication:** Not required
- **Body:** Google OAuth token
- **Returns:** JWT tokens and user info
- **Status:** 200 OK
- **Note:** This endpoint is not yet implemented

### GET /api/v1/auth/me
Get current user information
- **Authentication:** Required
- **Returns:** Current user profile data
- **Status:** 200 OK

### POST /api/v1/auth/logout
Logout user (invalidate tokens)
- **Authentication:** Required
- **Returns:** Success message
- **Status:** 200 OK

### POST /api/v1/auth/refresh
Refresh JWT access token
- **Authentication:** Refresh token required
- **Body:** Refresh token
- **Returns:** New access token
- **Status:** 200 OK

### POST /api/v1/auth/verify-email
Verify user email address
- **Authentication:** Not required
- **Body:** Email verification token
- **Returns:** Success message
- **Status:** 200 OK

### POST /api/v1/auth/resend-verification
Resend email verification
- **Authentication:** Required
- **Returns:** Success message
- **Status:** 200 OK

## Users API

### GET /api/v1/users/me
Get current user profile
- **Authentication:** Required
- **Returns:** User profile with environment count
- **Status:** 200 OK

### PUT /api/v1/users/me
Update current user profile
- **Authentication:** Required
- **Body:** Update data (fullName, username, avatarUrl)
- **Returns:** Updated user object
- **Status:** 200 OK
- **Note:** Email changes require separate secure flow

### POST /api/v1/users/change-password
Change user password
- **Authentication:** Required
- **Body:** Password change data (currentPassword, newPassword)
- **Returns:** Success message
- **Status:** 200 OK
- **Note:** All refresh tokens are revoked after password change

### GET /api/v1/users/me/stats
Get user statistics
- **Authentication:** Required
- **Returns:** User statistics including environment counts and activity
- **Status:** 200 OK

### DELETE /api/v1/users/me
Delete current user account
- **Authentication:** Required with email verification
- **Body:** Account deletion confirmation (password)
- **Returns:** Success message
- **Status:** 200 OK
- **Note:** Soft delete - user data is anonymized

## Environments API

### POST /api/v1/environments
Create a new development environment using DigitalOcean Droplets
- **Authentication:** Required
- **Body:** Environment creation data (name, template_id, region, size, etc.)
- **Returns:** Created environment object with "creating" status
- **Status:** 201 Created
- **Architecture:** Uses DigitalOcean Droplets with full SSH access
- **Provisioning:** Cloud-init scripts for automated setup and configuration

### GET /api/v1/environments
List user's environments
- **Authentication:** Required
- **Query Parameters:**
  - `status`: Filter by status (creating, installing, running, stopped, terminated, error)
  - `template_id`: Filter by template
- **Returns:** Array of environment objects

### GET /api/v1/environments/{environment_id}
Get specific environment details
- **Authentication:** Required
- **Returns:** Environment object with full details

### PUT /api/v1/environments/{environment_id}
Update an existing environment
- **Authentication:** Required
- **Body:** Environment update data
- **Returns:** Updated environment object

### DELETE /api/v1/environments/{environment_id}
Delete an environment
- **Authentication:** Required
- **Returns:** Success message
- **Status:** 204 No Content

### POST /api/v1/environments/{environment_id}/start
Start an environment by powering on the droplet
- **Authentication:** Required
- **Returns:** Success message
- **Note:** Environment must be in stopped state
- **Mechanism:** Uses DigitalOcean API to power on the droplet
- **Benefits:** Preserves all data and configuration, maintains IP address

### POST /api/v1/environments/{environment_id}/stop
Stop an environment by powering off the droplet
- **Authentication:** Required
- **Returns:** Success message
- **Note:** Environment must be in running state
- **Mechanism:** Uses DigitalOcean API to power off the droplet
- **Benefits:** Preserves all data, no billing for powered-off droplets (only storage)

### POST /api/v1/environments/{environment_id}/restart
Restart an environment
- **Authentication:** Required
- **Returns:** Success message
- **Note:** Environment must be in running, stopped, or error state

### GET /api/v1/environments/{environment_id}/metrics
Get environment resource usage metrics
- **Authentication:** Required
- **Returns:** Environment metrics data (CPU, memory, storage usage)

### GET /api/v1/environments/{environment_id}/logs
Get environment logs
- **Query Parameters:**
  - `lines`: Number of log lines to retrieve (1-1000, default: 100)
  - `since`: Get logs since timestamp (ISO format, e.g., 2024-01-01T12:00:00Z)
- **Authentication:** Required
- **Returns:** Log entries with metadata

## Templates API
Manage environment templates for different programming languages and frameworks.

### GET /api/v1/templates
List all available templates
- **Query Parameters:**
  - `category`: Filter by category (programming_language, framework, database, devops, operating_system)
  - `status`: Filter by status (active, deprecated, beta)
- **Authentication:** Required
- **Returns:** Array of template objects

### GET /api/v1/templates/{template_id}
Get specific template details
- **Authentication:** Required
- **Returns:** Template object with full details

### POST /api/v1/templates
Create a new template (Admin only)
- **Authentication:** Admin required
- **Body:** Template creation data
- **Returns:** Created template object

### PUT /api/v1/templates/{template_id}
Update an existing template (Admin only)
- **Authentication:** Admin required
- **Body:** Template update data
- **Returns:** Updated template object

### DELETE /api/v1/templates/{template_id}
Delete a template (Admin only) - Sets status to deprecated
- **Authentication:** Admin required
- **Returns:** Success message

### POST /api/v1/templates/initialize
Initialize default templates (Admin only)
- **Authentication:** Admin required
- **Returns:** Success message

## DigitalOcean Resources API
Manage DigitalOcean resources for environment deployment.

### GET /api/v1/regions
Get available DigitalOcean regions
- **Authentication:** Required
- **Returns:** Array of available regions with features

### GET /api/v1/sizes
Get available droplet sizes
- **Authentication:** Required
- **Returns:** Array of droplet sizes with pricing and specifications

### POST /api/v1/environments/{environment_id}/snapshot
Create a snapshot of the environment's droplet
- **Authentication:** Required
- **Body:** Snapshot name
- **Returns:** Snapshot object with ID and status
- **Status:** 201 Created

### POST /api/v1/environments/{environment_id}/resize
Resize the droplet to a different size
- **Authentication:** Required
- **Body:** New size specification
- **Returns:** Resize status
- **Note:** Droplet will be powered off during resize

## WebSocket API
Real-time communication endpoints for terminal and log streaming.

### WebSocket /api/v1/ws/terminal/{environment_id}
Terminal access to environment via SSH bridge
- **Authentication:** JWT token via query parameter `?token=jwt_token`
- **Connection:** WebSocket-to-SSH bridge to droplet
- **Messages:**
  - `{"type": "input", "data": "command\n"}` - Send terminal input
  - `{"type": "resize", "cols": 80, "rows": 24}` - Resize terminal
  - `{"type": "ping"}` - Keepalive (responds with pong)
- **Returns:** Real-time terminal output from SSH session

### WebSocket /api/v1/ws/logs/{environment_id}
Real-time log streaming from environment, including cloud-init logs
- **Authentication:** JWT token via query parameter `?token=jwt_token`
- **Connection:** SSH connection to stream cloud-init logs
- **Messages:**
  - `{"type": "ping"}` - Keepalive (responds with pong)
- **Returns:** Real-time log output from cloud-init process
- **Message Types:**
  - `installation_log` - Log lines from cloud-init process
  - `installation_complete` - Cloud-init completed successfully
  - `installation_status` - Cloud-init status updates
  - `installation_error` - Cloud-init error messages

## Default Templates

All templates include a comprehensive development stack pre-installed:

**Base Development Stack (All Templates)**:
- **System:** Ubuntu 22.04 LTS with `dev` user (sudo privileges)
- **Core Tools:** docker, curl, git, wget, vim, nano, htop, tmux
- **Languages:** Python 3.11+, Node.js LTS (via NVM)
- **Package Managers:** pip, npm, pnpm, yarn
- **AI Tools:** Claude Code CLI, Google Gemini CLI, Open Code
- **Terminal:** Pre-configured tmux with development optimizations

### Python Development
- **Base:** Ubuntu 22.04 + Development Stack
- **Additional:** virtualenv, pipenv, poetry, Flask, Django support
- **Python Version:** 3.11+ (configurable)
- **Default Port:** 8080

### Node.js Development
- **Base:** Ubuntu 22.04 + Development Stack 
- **Additional:** Express, React, Vue tooling, PM2, nodemon
- **Node Version:** LTS (via NVM, configurable)
- **Default Port:** 3000

### Full-Stack Development
- **Base:** Ubuntu 22.04 + Development Stack
- **Additional:** PostgreSQL, Redis, nginx setup scripts
- **Databases:** Client tools for PostgreSQL, MySQL, MongoDB
- **Default Port:** 8080

### Coding Agents (Premium)
- **Base:** Ubuntu 22.04 + Development Stack
- **Additional:** Claude Code, Gemini CLI, Open Code pre-configured
- **Features:** AI-powered coding assistance, code generation
- **Optimized:** For AI-assisted development workflows
- **Default Port:** 8080

## Response Formats

### User Response Object
```json
{
  "id": "string",
  "username": "string",
  "email": "string",
  "full_name": "string",
  "is_active": true,
  "is_verified": false,
  "avatar_url": "string",
  "subscription_plan": "free",
  "created_at": "2024-01-01T00:00:00Z",
  "last_login": "2024-01-01T00:00:00Z"
}
```

### Environment Object
```json
{
  "id": "string",
  "name": "string",
  "description": "string",
  "template_id": "string",
  "template_name": "string",
  "status": "running",
  "droplet_id": 123456789,
  "droplet_ip": "167.99.123.45",
  "ssh_port": 22,
  "region": "nyc3",
  "size": "s-1vcpu-1gb",
  "storage_size_gb": 10,
  "volume_id": "vol-123456",
  "environment_variables": {},
  "installation_completed": true,
  "cpu_usage": 0.25,
  "memory_usage": 0.45,
  "storage_usage": 0.10,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z",
  "last_activity_at": "2024-01-01T00:00:00Z"
}
```

### Template Object
```json
{
  "id": "string",
  "name": "string",
  "display_name": "string",
  "description": "string",
  "category": "programming_language",
  "tags": ["array", "of", "strings"],
  "docker_image": "string",
  "default_port": 8080,
  "default_resources": {
    "cpu": "500m",
    "memory": "1Gi",
    "storage": "10Gi"
  },
  "environment_variables": {},
  "startup_commands": ["array", "of", "commands"],
  "documentation_url": "string",
  "icon_url": "string",
  "status": "active",
  "version": "1.0.0",
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z",
  "usage_count": 0
}
```

### Droplet Object
```json
{
  "id": "string",
  "environment_id": "string",
  "droplet_id": 123456789,
  "name": "string",
  "status": "active",
  "ip_address": "167.99.123.45",
  "private_ip": "10.132.0.2",
  "region": "nyc3",
  "size": "s-1vcpu-1gb",
  "image": "ubuntu-22-04-x64",
  "volume_ids": ["vol-123456"],
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T00:00:00Z"
}
```

### Log Response
```json
{
  "environment_id": "string",
  "environment_name": "string",
  "logs": [
    {
      "timestamp": "2024-01-01T00:00:00Z",
      "level": "INFO",
      "message": "Application started successfully",
      "source": "container"
    }
  ],
  "total_lines": 100,
  "has_more": false
}
```

### Token Response
```json
{
  "access_token": "string",
  "refresh_token": "string",
  "token_type": "bearer",
  "expires_in": 3600,
  "user": {
    "id": "string",
    "username": "string",
    "email": "string"
  }
}
```

### Terminal Session Object
```json
{
  "id": "string",
  "environment_id": "string",
  "session_id": "string",
  "status": "active",
  "tmux_session_name": "devpocket_env123",
  "client_info": {
    "user_agent": "string",
    "ip_address": "string"
  },
  "started_at": "2024-01-01T00:00:00Z",
  "last_activity_at": "2024-01-01T00:00:00Z"
}
```

### Environment Log Object
```json
{
  "id": "string",
  "environment_id": "string",
  "level": "INFO",
  "message": "Container started successfully",
  "source": "container",
  "metadata": {},
  "timestamp": "2024-01-01T00:00:00Z"
}
```

### User Stats Response
```json
{
  "total_environments": 5,
  "active_tokens": 2,
  "environments_by_status": {
    "running": 3,
    "stopped": 1,
    "creating": 1
  },
  "recent_activity": {
    "environments_created_last_7_days": 2,
    "environments_created_last_30_days": 4,
    "environments_active_last_week": 3
  }
}
```

## Error Responses

All endpoints return structured error responses:
```json
{
  "detail": "Error message",
  "errors": [] // For validation errors
}
```

Common HTTP status codes:
- `200`: Success
- `201`: Created
- `204`: No Content
- `400`: Bad Request
- `401`: Unauthorized
- `403`: Forbidden (Admin required)
- `404`: Not Found
- `422`: Validation Error
- `429`: Too Many Requests (Rate Limited)
- `500`: Internal Server Error
- `503`: Service Unavailable

## Rate Limiting

API endpoints are rate limited to prevent abuse:
- Authentication endpoints: 5 requests per minute
- General API endpoints: 100 requests per minute
- WebSocket connections: 10 connections per user

Rate limit headers are included in responses:
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Remaining requests in current window
- `X-RateLimit-Reset`: Time when rate limit resets