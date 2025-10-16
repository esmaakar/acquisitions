# Docker Setup Guide

This guide explains how to run the Acquisitions API using Docker with different configurations for development and production environments.

## Overview

The Docker setup supports two main environments:

- **Development**: Uses Neon Local proxy with ephemeral database branches
- **Production**: Connects directly to Neon Cloud database

## Prerequisites

1. **Docker & Docker Compose**: Install [Docker Desktop](https://www.docker.com/products/docker-desktop/)
2. **Neon Account**: Create account at [console.neon.tech](https://console.neon.tech)
3. **Neon API Key**: Generate from Neon Console → Account Settings → API Keys
4. **Neon Project**: Create or use existing project

## Quick Start

### Development Environment

1. **Setup Environment Variables**:
   ```bash
   # Copy the development template
   cp .env.development .env
   
   # Edit .env with your Neon credentials
   NEON_API_KEY=your_api_key_here
   NEON_PROJECT_ID=your_project_id_here
   PARENT_BRANCH_ID=main
   JWT_SECRET=your-development-jwt-secret
   ```

2. **Start Development Environment**:
   ```bash
   npm run docker:dev
   ```

3. **Access Your Application**:
   - API: http://localhost:3000
   - Database: localhost:5432 (via Neon Local proxy)

4. **Stop Development Environment**:
   ```bash
   npm run docker:dev:down
   ```

### Production Environment

1. **Setup Environment Variables**:
   ```bash
   # Copy the production template
   cp .env.production .env
   
   # Edit .env with your production settings
   DATABASE_URL=postgresql://username:password@your-neon-endpoint.region.aws.neon.tech/dbname?sslmode=require
   JWT_SECRET=your-super-secure-production-jwt-secret
   ```

2. **Start Production Environment**:
   ```bash
   npm run docker:prod
   ```

3. **Stop Production Environment**:
   ```bash
   npm run docker:prod:down
   ```

## Environment Configuration

### Development (.env.development)

```env
# Server Configuration
PORT=3000
NODE_ENV=development
LOG_LEVEL=debug

# Neon Local Configuration
NEON_API_KEY=your_neon_api_key_here
NEON_PROJECT_ID=your_neon_project_id_here
PARENT_BRANCH_ID=main

# Database Configuration (Neon Local)
DATABASE_URL=postgres://neon:npg@neon-local:5432/neondb?sslmode=require

# JWT Configuration
JWT_SECRET=your-development-jwt-secret
```

### Production (.env.production)

```env
# Server Configuration
PORT=3000
NODE_ENV=production
LOG_LEVEL=info

# Database Configuration (Neon Cloud)
DATABASE_URL=postgresql://username:password@your-neon-endpoint.region.aws.neon.tech/dbname?sslmode=require

# JWT Configuration
JWT_SECRET=your-super-secure-production-jwt-secret

# Additional Production Settings
CORS_ORIGIN=https://your-frontend-domain.com
```

## Available Docker Commands

| Command | Description |
|---------|-------------|
| `npm run docker:dev` | Start development environment with Neon Local |
| `npm run docker:dev:down` | Stop development environment |
| `npm run docker:dev:logs` | View development logs |
| `npm run docker:prod` | Start production environment |
| `npm run docker:prod:down` | Stop production environment |
| `npm run docker:prod:logs` | View production logs |
| `npm run docker:build` | Build application Docker image |
| `npm run docker:clean` | Clean up Docker system and volumes |

## How It Works

### Development with Neon Local

1. **Neon Local Container**: Runs `neondatabase/neon_local:latest`
   - Creates ephemeral database branches automatically
   - Proxies connections to your Neon project
   - Branches are deleted when container stops

2. **Application Container**: Your Node.js app
   - Connects to `neon-local:5432` instead of external database
   - Hot-reloads source code changes
   - Uses development configuration

### Production with Neon Cloud

1. **Application Container**: Your Node.js app only
   - Connects directly to Neon Cloud database
   - Uses production optimizations
   - Resource limits and health checks

## Database Operations

### Running Migrations

**Development**:
```bash
# Start the development environment first
npm run docker:dev

# In another terminal, run migrations inside the container
docker exec acquisitions-app-dev npm run db:migrate
```

**Production**:
```bash
# Start the production environment first
npm run docker:prod

# Run migrations inside the container
docker exec acquisitions-app-prod npm run db:migrate
```

### Database Studio

```bash
# Start development environment
npm run docker:dev

# Access Drizzle Studio inside container
docker exec -it acquisitions-app-dev npm run db:studio
```

## Troubleshooting

### Common Issues

1. **Port Already in Use**:
   ```bash
   # Check what's using port 3000
   netstat -ano | findstr :3000
   
   # Stop existing containers
   docker stop $(docker ps -q)
   ```

2. **Neon Local Connection Issues**:
   ```bash
   # Check Neon Local container logs
   docker logs acquisitions-neon-local
   
   # Verify environment variables
   docker exec acquisitions-neon-local env | grep NEON
   ```

3. **Database Connection Errors**:
   ```bash
   # Test database connectivity
   docker exec acquisitions-app-dev node -e "
   import { db } from '#config/database.js';
   console.log('Testing database connection...');
   "
   ```

4. **SSL Certificate Issues** (Development):
   The app is configured to handle Neon Local's self-signed certificates automatically.

### Logs and Debugging

```bash
# View all logs
npm run docker:dev:logs

# View specific service logs
docker logs acquisitions-app-dev
docker logs acquisitions-neon-local

# Interactive shell access
docker exec -it acquisitions-app-dev sh
```

## File Structure

```
acquisitions/
├── Dockerfile                  # Main application container
├── docker-compose.dev.yml      # Development with Neon Local
├── docker-compose.prod.yml     # Production with Neon Cloud
├── .dockerignore               # Files to exclude from build
├── .env.development            # Development environment template
├── .env.production             # Production environment template
├── .neon_local/                # Neon Local metadata (git ignored)
└── logs/                       # Application logs
```

## Security Notes

1. **Never commit `.env` files** to version control
2. **Use strong JWT secrets** in production
3. **Rotate API keys** regularly
4. **Use HTTPS** in production
5. **Limit container resources** in production

## Performance Optimization

### Development
- Source code is mounted as volume for hot reload
- Debug logging enabled
- No resource limits

### Production
- Multi-stage builds for smaller images
- Non-root user for security
- Resource limits configured
- Health checks enabled
- Production logging level

## Next Steps

1. Set up CI/CD pipeline with these Docker configurations
2. Configure monitoring and alerting
3. Set up backup strategies for production
4. Implement secrets management (AWS Secrets Manager, etc.)
5. Configure load balancing for multiple instances

For more information about Neon Local, visit: https://neon.com/docs/local/neon-local