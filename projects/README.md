# Projects Directory

This directory contains the main applications of the FastGPT platform. Each project is a standalone application that serves a specific purpose within the FastGPT ecosystem.

## Applications Overview

### `app/` - FastGPT Core Application
The main FastGPT web application built with NextJS. This is the primary user interface for the FastGPT platform, providing:
- AI Agent building interface
- Visual workflow orchestration through Flow
- Data processing and model invocation capabilities
- User management and authentication
- Dataset management and search functionality
- Real-time chat interface

**Technology Stack:** NextJS + TypeScript + ChakraUI + MongoDB/PostgreSQL

**Key Features:**
- Multi-language support (English, Chinese, Japanese)
- Plugin architecture integration
- REST API endpoints
- Real-time communication
- Responsive web design

### `sandbox/` - Code Execution Sandbox
A secure NestJS-based sandbox service for executing code within FastGPT workflows. This service provides isolated execution environments for user code.

**Technology Stack:** NestJS + TypeScript + isolated-vm

**Environment Requirements:**
- Python 3.11 environment
- Additional packages specified in `requirements.txt` will be installed automatically during runtime

**Package Management:**
- Add required Python packages to `requirements.txt` in the sandbox directory
- Current packages: `numpy`, `pandas`
- Note: Some packages may require additional system libraries (e.g., pandas requires libffi)

**Troubleshooting Permission Issues:**
If you encounter timeout or permission denied errors with newly added Python packages (and have confirmed it's not a syntax issue), execute the following commands within the Docker container:

```shell
# Enter the container
docker exec -it <container_name> /bin/bash

# Run system call tests
chmod -x testSystemCall.sh
bash ./testSystemCall.sh
```

After running these commands, update the `SYSTEM_CALLS` array in `src/sandbox/constants.py` by replacing or appending the new system calls from the test results.

### `mcp_server/` - Model Context Protocol Server
Implementation of the Model Context Protocol (MCP) server, enabling standardized communication between FastGPT and external AI models and tools.

**Technology Stack:** Bun + TypeScript + Express

**Key Features:**
- MCP protocol implementation
- External AI model integration
- Tool invocation capabilities
- Real-time model communication

**Development Commands:**
- `bun dev` - Start development server with hot reload
- `bun build` - Build the MCP server
- `bun start` - Start production server
- `npx @modelcontextprotocol/inspector` - Test MCP functionality

### `marketplace/` - Marketplace Application
A dedicated NextJS application for the FastGPT template and plugin marketplace, allowing users to discover, share, and deploy pre-built AI agents and workflows.

**Technology Stack:** NextJS + TypeScript + Tailwind CSS + MongoDB

**Key Features:**
- Template discovery and browsing
- Plugin marketplace integration
- User-generated content sharing
- Community features and ratings

## Relationship to Packages Directory

The `projects/` directory contains the runnable applications, while the `packages/` directory contains shared libraries and utilities:

- **`@fastgpt/global`** - Shared types, constants, and utilities
- **`@fastgpt/service`** - Backend services, database models, and API controllers
- **`@fastgpt/web`** - Shared frontend components, hooks, and styles
- **`@fastgpt/templates`** - Application templates

Projects import shared code from the packages directory using workspace references, promoting code reuse and maintainability across the FastGPT ecosystem.

## Development Workflow

1. **Setup Development Environment:**
   ```bash
   # From project root
   pnpm install
   pnpm dev  # Start all projects
   ```

2. **Individual Project Development:**
   ```bash
   # Main application
   cd projects/app && pnpm dev

   # Sandbox service
   cd projects/sandbox && pnpm dev

   # MCP server
   cd projects/mcp_server && bun dev

   # Marketplace
   cd projects/marketplace && pnpm dev
   ```

3. **Building Projects:**
   ```bash
   # Build all projects
   pnpm build

   # Build individual projects
   cd projects/<project_name> && pnpm build
   ```

## Deployment

Each project can be deployed independently:
- **app/**: Main web application deployment
- **sandbox/**: Containerized sandbox service for workflow code execution
- **mcp_server/**: Standalone MCP server for model integration
- **marketplace/**: Marketplace platform deployment

All projects support Docker deployment with configurations provided in their respective `Dockerfile` files.

## Security Considerations

- **Sandbox Isolation**: The sandbox service runs code in isolated environments using isolated-vm
- **Permission Management**: System calls are explicitly controlled and monitored
- **Container Security**: Each service runs in its own container with restricted permissions
- **API Authentication**: JWT-based authentication and authorization across all services

## Additional Resources

- [Main Documentation](../../document/)
- [Development Guidelines](../../CLAUDE.md)
- [API Reference](../../packages/global/openapi/)
- [Deployment Guide](../../deploy/)