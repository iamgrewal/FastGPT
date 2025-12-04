# CLAUDE.md

This file provides guidance for Claude Code (claude.ai/code) when working in this repository.

## Communication Language

**IMPORTANT**: All communication, documentation, and code comments must be in English only. This is strictly enforced to maintain consistency and clarity for all developers working on this project.

## Output Requirements

1. **Language**: All output must be in English only
2. **Design Documents**: Save to `.claude/design/` directory as Markdown files
3. **Plans**: All planning documents must be saved to `.claude/plan/` directory as Markdown files
4. **Code Comments**: Write all code comments in English
5. **Variable Names**: Use English for all variable, function, and class names

## Project Overview

FastGPT is an AI Agent building platform that provides out-of-the-box data processing, model invocation capabilities, and visual workflow orchestration through Flow. This is a full-stack TypeScript application built on NextJS with backend using MongoDB/PostgreSQL.

**Technology Stack**: NextJS + TypeScript + ChakraUI + MongoDB + PostgreSQL (PG Vector)/Milvus

## Architecture

This is a monorepo using pnpm workspaces with the following structure:

### Packages (Library Code)
- `packages/global/` - Shared types, constants, utilities across all projects
- `packages/service/` - Backend services, database models, API controllers, workflow engine
- `packages/web/` - Shared frontend components, hooks, styles, internationalization
- `packages/templates/` - Application templates for the template marketplace

### Projects (Applications)
- `projects/app/` - Main NextJS Web application (frontend + API routes)
- `projects/sandbox/` - NestJS code execution sandbox service
- `projects/mcp_server/` - Model Context Protocol server implementation
- `projects/marketplace/` - Marketplace application

### Key Directories
- `document/` - Documentation site (NextJS application and content)
- `plugins/` - External plugins (models, crawlers, etc.)
- `deploy/` - Docker and Helm deployment configurations
- `test/` - Centralized test files and utilities

## Development Commands

### Main Commands (run from project root)
- `pnpm dev` - Start development environment for all projects (using workspace scripts)
- `pnpm build` - Build all projects
- `pnpm test` - Run tests using Vitest
- `pnpm test:workflow` - Run workflow-specific tests
- `pnpm lint` - Run ESLint on all TypeScript files with auto-fix
- `pnpm format-code` - Format code using Prettier

### Project-Specific Commands

**Main Application (projects/app/)**:
- `cd projects/app && pnpm dev` - Start NextJS development server
- `cd projects/app && pnpm build` - Build NextJS application
- `cd projects/app && pnpm start` - Start production server

**Sandbox (projects/sandbox/)**:
- `cd projects/sandbox && pnpm dev` - Start NestJS development server in watch mode
- `cd projects/sandbox && pnpm build` - Build NestJS application
- `cd projects/sandbox && pnpm test` - Run Jest tests

**MCP Server (projects/mcp_server/)**:
- `cd projects/mcp_server && bun dev` - Start with Bun in watch mode
- `cd projects/mcp_server && bun build` - Build MCP server
- `cd projects/mcp_server && bun start` - Start MCP server

### Utility Commands
- `pnpm create:i18n` - Generate internationalization translation files
- `pnpm api:gen` - Generate OpenAPI documentation
- `pnpm initIcon` - Initialize icon resources
- `pnpm gen:theme-typings` - Generate Chakra UI theme type definitions

## Testing

The project uses Vitest for testing and generates coverage reports. Main test commands:
- `pnpm test` - Run all tests
- `pnpm test:workflow` - Run workflow-specific tests
- Test files are located in `test/` directory and `projects/app/test/`
- Coverage reports are generated in `coverage/` directory

## Code Organization Patterns

### Monorepo Structure
- Shared code is stored in `packages/` and imported through workspace references
- Each project in `projects/` is an independent application
- Use `@fastgpt/global`, `@fastgpt/service`, `@fastgpt/web` to import shared packages

### API Structure
- NextJS API routes are in `projects/app/src/pages/api/`
- API route contracts are defined in `packages/global/openapi/`
- Common server-side business logic is in `packages/service/` and `projects/app/src/service`
- Database models are in `packages/service/` using MongoDB/Mongoose

### Frontend Architecture
- React components are in `projects/app/src/components/` and `packages/web/components/`
- Uses Chakra UI for styling with custom theme in `packages/web/styles/theme.ts`
- Internationalization support files are in `packages/web/i18n/`
- Uses React Context and Zustand for state management

## Development Guidelines

### Environment Requirements
- **Package Manager**: Use pnpm with workspace configuration
- **Node Version**: Requires Node.js >= 18.16.0, pnpm >= 9.0.0
- **Database**: Supports MongoDB, PostgreSQL with pgvector, or Milvus vector storage
- **AI Integration**: Unified interface supporting multiple AI providers
- **Internationalization**: Full support for Chinese, English, and Japanese

### Key File Patterns
- All `.ts` and `.tsx` files use TypeScript exclusively
- Database models use Mongoose with TypeScript
- API routes follow NextJS conventions
- Component files use React functional components and hooks
- Shared type definitions are in `.d.ts` files in `packages/global/`

### Code Standards
- **Type Declarations**: Use `type` for type declarations instead of `interface` when possible
- **Code Style**: Follow ESLint and Prettier configurations (auto-applied on commit)
- **Git Hooks**: Husky is configured for pre-commit checks

## Agent Development Guidelines

### Workflow for Feature Implementation
1. **Design First**: Create detailed design documents before implementation
2. **User Confirmation**: Always get user confirmation before proceeding with implementation
3. **Test-Driven Approach**: Use "design-document → test-examples → code-writing → test-execution → code/documentation-correction" workflow
4. **Core Focus**: Use testing as the core mechanism to ensure design correctness

### Documentation Requirements
- All design documents must be in English
- Save designs to `.claude/design/` directory
- Include detailed implementation steps and test cases
- Document architecture decisions and trade-offs

## AI Service Architecture

### Model Integration
- **Unified Interface**: Abstract layer for multiple AI providers
- **Supported Models**: OpenAI, Anthropic Claude, local models, various open-source models
- **Vectorization**: Multiple embedding options (OpenAI, local models)
- **File Processing**: Support for PDF, Word, Excel, images

### Configuration
- AI Proxy endpoint: `AIPROXY_API_ENDPOINT`
- OpenAI base URL: `OPENAI_BASE_URL`
- API keys: `CHAT_API_KEY`
- Database connections: `MONGODB_URI`

## Workflow Engine

### Core Node Types
- `workflowStart`: Workflow initialization
- `answerNode`: Response generation
- `chatNode`: Chat interaction
- `datasetSearchNode`: Dataset search
- `httpRequest468`: HTTP request handling
- `pluginModule`: Plugin integration
- `code`: Code execution
- `ifElseNode`: Conditional logic
- `loop`: Loop control

### Execution Limits
- Maximum workflow runs: 500
- Maximum loop iterations: 50
- Timeout controls for safety

## Plugin Architecture

### Plugin Types
- **Model Plugins** (`plugins/model/`): Various LLM and specialized models
- **Crawler Plugins** (`plugins/webcrawler/`): Web scraping and search capabilities
- **Microservices**: Each plugin runs as an independent service
- **REST API**: Plugin communication via HTTP

### Deployment
- Docker containerization for each plugin
- Independent scaling and deployment
- Configuration-driven loading mechanism

## Security Architecture

### Authentication & Authorization
- JWT token-based authentication
- RBAC (Role-Based Access Control)
- API key management
- IP whitelisting support

### Security Measures
- Input validation and sanitization
- SQL injection prevention
- XSS protection
- CSRF protection
- File upload restrictions

## Performance Optimization

### Frontend Optimization
- Code splitting with dynamic imports
- Image optimization and compression
- HTTP caching strategies
- Lazy loading for components and images

### Backend Optimization
- Database indexing for query optimization
- Redis caching strategies
- Database connection pooling
- Asynchronous processing with message queues

## Environment Configuration

### Key Configuration Files
- Main config: `projects/app/data/config.json`
- Environment-specific configurations supported
- Model configurations: `packages/service/core/ai/config/`

### Feature Flags
- Log level: `LOG_LEVEL`
- Copyright hiding: `HIDE_CHAT_COPYRIGHT_SETTING`
- Rate limiting: `USE_IP_LIMIT`
- Workflow limits: `WORKFLOW_MAX_RUN_TIMES`

## Deployment Architecture

### Containerization
- Multi-stage Docker builds
- Optimized image sizes
- Security configurations
- Environment variable management

### Orchestration
- Kubernetes deployment via Helm charts
- Auto-scaling support
- Configuration management
- Service discovery

## Testing Architecture

### Test Configuration
- Vitest with 5 concurrent threads
- Coverage reporting enabled
- MongoDB Memory Server for isolated testing
- Environment isolation

### Test Types
- Unit tests
- Integration tests
- API tests
- Workflow tests

## Common Development Patterns

### Error Handling
- Unified error response format
- Graceful degradation
- Retry mechanisms for external services
- Comprehensive logging

### Database Patterns
- Connection pooling
- Transaction management
- Optimistic locking
- Audit trails for data changes

### API Design
- RESTful conventions
- Consistent response formats
- Version support
- OpenAPI documentation

## Debugging and Monitoring

### Logging
- Structured logging with different levels
- Centralized log collection
- Performance monitoring
- Error tracking

### Development Tools
- Hot reload for rapid development
- Source maps for debugging
- TypeScript strict mode
- Comprehensive IDE support

## Contributing to FastGPT

### Before Starting
- Read existing code and understand patterns
- Check for similar implementations
- Review architecture documentation
- Plan changes thoroughly

### Implementation Process
1. Create design document
2. Write tests first (TDD approach)
3. Implement minimal viable solution
4. Refactor and optimize
5. Update documentation
6. Get code review

### Quality Standards
- All new code must have tests
- Follow existing code patterns
- Maintain backward compatibility
- Document breaking changes

This comprehensive guide should help Claude Code assistants understand the FastGPT codebase architecture, development patterns, and best practices for contributing to this AI-powered platform.