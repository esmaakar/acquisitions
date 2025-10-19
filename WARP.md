# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Development Commands

### Core Development

```bash
# Start development server with hot reload
npm run dev

# Start production server
npm start

# Code quality and formatting
npm run lint              # Check for ESLint violations
npm run lint:fix          # Auto-fix ESLint violations
npm run format            # Format code with Prettier
npm run format:check      # Check Prettier formatting

# Database operations
npm run db:generate       # Generate Drizzle migrations
npm run db:migrate        # Run database migrations
npm run db:studio         # Open Drizzle Studio (database GUI)
```

### Docker Development

```bash
# Development with Neon Local (ephemeral database branches)
npm run dev:docker        # Start development environment
# Or use: sh ./scripts/dev.bash

# Production with Neon Cloud
npm run prod:docker       # Start production environment
# Or use: sh ./scripts/prod.sh
```

### Database Operations in Docker

```bash
# Run migrations inside development container
docker exec acquisitions-app-dev npm run db:migrate

# Access Drizzle Studio in container
docker exec -it acquisitions-app-dev npm run db:studio

# Run migrations inside production container
docker exec acquisitions-app-prod npm run db:migrate
```

## Architecture Overview

### Technology Stack

- **Runtime**: Node.js with ES modules
- **Framework**: Express.js 5.x
- **Database**: PostgreSQL via Neon (serverless PostgreSQL)
- **ORM**: Drizzle ORM with HTTP client
- **Security**: Arcjet (rate limiting, bot protection, security policies)
- **Authentication**: JWT with bcrypt password hashing
- **Logging**: Winston with Morgan for HTTP logging
- **Validation**: Zod schemas

### Project Structure

The codebase uses ES module imports with path mapping defined in `package.json`:

```
src/
├── config/           # Configuration (database, logger, Arcjet)
├── controllers/      # Request handlers
├── middleware/       # Express middleware (security, auth)
├── models/          # Drizzle schema definitions
├── routes/          # Route definitions
├── services/        # Business logic layer
├── utils/           # Utility functions (JWT, cookies, formatting)
└── validations/     # Zod validation schemas
```

### Path Mapping

The project uses import maps for cleaner imports:

- `#config/*` → `./src/config/*`
- `#controllers/*` → `./src/controllers/*`
- `#middleware/*` → `./src/middleware/*`
- `#models/*` → `./src/models/*`
- `#routes/*` → `./src/routes/*`
- `#services/*` → `./src/services/*`
- `#utils/*` → `./src/utils/*`
- `#validations/*` → `./src/validations/*`

### Database Configuration

- **Development**: Uses Neon Local proxy for ephemeral database branches
- **Production**: Connects directly to Neon Cloud
- **Migrations**: Managed by Drizzle Kit, schemas in `src/models/`
- **Configuration**: Database URL and environment-specific settings in `.env`

### Security Architecture

- **Arcjet Integration**: Role-based rate limiting (admin: 20/min, user: 10/min, guest: 5/min)
- **Bot Protection**: Automated request detection and blocking
- **Middleware Stack**: Helmet, CORS, security middleware applied globally
- **JWT Authentication**: Token-based auth with secure cookie handling

### API Structure

- **Base Endpoint**: `/api`
- **Health Check**: `/health` (returns status, timestamp, uptime)
- **Authentication**: `/api/auth` (sign-up, sign-in, sign-out)
- **Users**: `/api/users` (user management endpoints)

### Environment Configuration

Two main environments with different database strategies:

- **Development**: Neon Local with ephemeral branches (Docker: `docker-compose.dev.yml`)
- **Production**: Direct Neon Cloud connection (Docker: `docker-compose.prod.yml`)

### Development Workflow

1. **Local Development**: Use `npm run dev` for hot reload
2. **Database Changes**: Generate migrations with `npm run db:generate`, then run with `npm run db:migrate`
3. **Code Quality**: Run `npm run lint:fix` and `npm run format` before commits
4. **Docker Development**: Use `npm run dev:docker` for containerized development with Neon Local
5. **Database GUI**: Access Drizzle Studio with `npm run db:studio`

### Testing

The ESLint configuration includes test globals (describe, it, expect, etc.) but no test runner is currently configured in package.json scripts.

### Docker Notes

- **Development**: Uses Neon Local container for database branching
- **Production**: Connects directly to Neon Cloud
- **Containers**: Separate configurations for dev/prod with different resource limits
- **SSL**: Development environment handles Neon Local's self-signed certificates automatically
