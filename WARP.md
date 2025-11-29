# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview
Node.js/Express REST API for an acquisitions management system using Drizzle ORM with Neon (PostgreSQL) database. The project uses ES modules (type: "module") and implements path aliases for cleaner imports.

## Development Commands

### Running the Application
```bash
npm run dev
```
Starts development server with Node's `--watch` flag for auto-reload on file changes.

### Database Management
```bash
npm run db:generate    # Generate migration files from schema changes
npm run db:migrate     # Apply pending migrations to database
npm run db:studio      # Open Drizzle Studio UI for database inspection
```

**Important**: After modifying model files in `src/models/`, always run `db:generate` to create migrations, then `db:migrate` to apply them.

### Code Quality
```bash
npm run lint           # Check for linting issues
npm run lint:fix       # Auto-fix linting issues
npm run format         # Format code with Prettier
npm run format:check   # Check if code is properly formatted
```

## Architecture

### Path Aliases
The project uses Node.js subpath imports (defined in package.json) for cleaner imports:
- `#config/*` → `./src/config/*`
- `#controllers/*` → `./src/controllers/*`
- `#models/*` → `./src/models/*`
- `#routes/*` → `./src/routes/*`
- `#services/*` → `./src/services/*`
- `#utils/*` → `./src/utils/*`
- `#validations/*` → `./src/validations/*`

Always use these aliases instead of relative paths.

### Application Structure
The codebase follows a layered architecture:

**Entry Point Flow**:
`index.js` → loads environment variables → imports `server.js` → `server.js` starts Express app from `app.js`

**Layer Responsibilities**:
- **Routes** (`src/routes/`): Define API endpoints and attach controllers
- **Controllers** (`src/controllers/`): Handle HTTP requests/responses, validate input using Zod schemas, orchestrate services
- **Services** (`src/services/`): Business logic, database operations via Drizzle ORM
- **Models** (`src/models/`): Drizzle schema definitions using pgTable
- **Validations** (`src/validations/`): Zod schemas for request validation
- **Utils** (`src/utils/`): Shared utilities (JWT, cookies, formatting)
- **Config** (`src/config/`): Database connection and Winston logger setup
- **Middleware** (`src/middleware/`): Currently empty, intended for custom middleware

### Database Pattern
- Uses Drizzle ORM with Neon serverless PostgreSQL driver
- Models define schema using `pgTable` from `drizzle-orm/pg-core`
- Database instance exported from `src/config/database.js` as `{db, sql}`
- Schema files in `src/models/` are auto-discovered by drizzle.config.js
- Migrations stored in `./drizzle` directory

### Authentication Pattern
- JWT tokens generated with `jwttoken.sign()` from `#utils/jwt.js`
- Tokens stored in HTTP-only cookies using `cookies.set()` from `#utils/cookies.js`
- Cookie settings: httpOnly, secure in production, sameSite: 'strict', 15min maxAge
- Password hashing uses bcrypt with salt rounds of 10

### Logging
- Winston logger configured in `src/config/logger.js`
- Logs to files: `logs/error.log` (errors only) and `logs/combined.log` (all levels)
- Console logging enabled in non-production environments
- Morgan middleware integrated to log HTTP requests via Winston
- Service metadata: `{ service: 'adquisitions-api' }`

### Validation Pattern
Controllers use Zod schemas with `.safeParse()` and return formatted validation errors via `formatValidationErrors()` from `#utils/format.js` on failure.

## Code Style Conventions

### ESLint Rules
- 2-space indentation
- Single quotes for strings
- Semicolons required
- Unix line endings
- No var declarations (use const/let)
- Unused vars allowed if prefixed with underscore: `_unusedVar`
- Prefer const over let
- Use object shorthand and arrow functions

### File Naming
All files use kebab-case: `auth.controller.js`, `user.model.js`, `auth.validations.js`

## Environment Setup
Copy `.env.example` to `.env` and configure:
```
PORT=3000
NODE_ENV=development
LOG_LEVEL=info
DATABASE_URL=          # Neon PostgreSQL connection string
JWT_SECRET=            # Cryptographically secure random string
JWT_EXPIRES_IN=1d      # JWT expiration duration
```

## API Endpoints
Base path: `/api`

**Health checks**:
- `GET /` - Basic hello response
- `GET /health` - Health check with uptime
- `GET /api` - API status check

**Auth** (`/api/auth`):
- `POST /api/auth/sign-up` - User registration (implemented)
- `POST /api/auth/sign-in` - User login (placeholder)
- `POST /api/auth/sign-out` - User logout (placeholder)

## Current State
The project is in early development with basic authentication infrastructure. Only user sign-up is fully implemented. Sign-in and sign-out endpoints exist as stubs. The middleware directory is empty, suggesting planned additions for authentication checks and error handling.
