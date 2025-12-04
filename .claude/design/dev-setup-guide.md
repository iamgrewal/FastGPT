# FastGPT Development Environment Setup Guide

## Project Overview

FastGPT is an AI Agent building platform based on Next.js, featuring a monorepo architecture managed with pnpm workspace.

## Environment Requirements

- **Node.js**: >= 20.0.0 (current: 20.18.0) ✅
- **pnpm**: 9.15.9+ (current: 10.13.1) ✅
- **Database**: MongoDB/PostgreSQL/Milvus (optional, can use remote database for development)

## Project Structure

```
FastGPT/
├── packages/           # Shared library code
│   ├── global/         # Global types, constants, utilities
│   ├── service/        # Backend services, database models, API controllers
│   └── web/           # Frontend components, hooks, styles, i18n
├── projects/           # Applications
│   ├── app/           # Main NextJS Web application
│   ├── sandbox/       # NestJS code execution sandbox
│   └── mcp_server/    # MCP server implementation
├── document/          # Documentation site
├── plugins/           # External plugins
└── deploy/           # Deployment configurations
```

## Initial Setup Steps

### 1. Environment Configuration

Copy and modify environment configuration files:
```bash
# Main application configuration
cp projects/app/.env.template projects/app/.env.local

# MCP server configuration (optional)
cp projects/mcp_server/.env.template projects/mcp_server/.env
```

### 2. Database Configuration

Configure database in `projects/app/.env.local`:

**MongoDB** (required):
```
MONGODB_URI="mongodb://username:password@localhost:27017/fastgpt?authSource=admin&directConnection=true"
MONGODB_LOG_URI="mongodb://username:password@localhost:27017/fastgpt?authSource=admin&directConnection=true"
```

**Vector Database** (choose one):
```
# PostgreSQL with vector extension
PG_URL=postgresql://username:password@localhost:5432/postgres

# OR Milvus vector database
MILVUS_ADDRESS=http://localhost:19530
MILVUS_TOKEN=your_token
```

### 3. Development Commands

**Start all services**:
```bash
pnpm dev
```

**Start individual services**:
```bash
# Main application (Web + API)
cd projects/app && pnpm dev

# Sandbox service
cd projects/sandbox && pnpm dev

# MCP server (requires Bun)
cd projects/mcp_server && bun dev
```

**Other common commands**:
```bash
# Build all projects
pnpm build

# Run tests
pnpm test

# Code linting and formatting
pnpm lint
pnpm format-code

# Generate internationalization files
pnpm create:i18n
```

## Development Tools

- **Main application**: http://localhost:3000
- **API endpoints**: http://localhost:3000/api/
- **Sandbox service**: http://localhost:3002
- **Plugin service**: http://localhost:3003

## Important Notes

1. **First startup**: Ensure database services are running, or use remote database configuration
2. **Dependencies**: Project has automatically installed 2332+ dependency packages
3. **Code standards**: Project is configured with ESLint and Prettier, auto-formatting on commit
4. **Git Hooks**: Husky is enabled and will run checks on code commits

## Troubleshooting

- If you encounter dependency issues, run `pnpm install` to reinstall
- If you encounter permission issues, check database connection configuration
- MCP server requires Bun runtime to be installed

## Next Steps

After configuration is complete, visit http://localhost:3000 to start using FastGPT!