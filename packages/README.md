# Packages Directory

This directory contains FastGPT's shared dependency packages that are reused across multiple projects in the monorepo. These packages provide core functionality, types, and components that enable consistent behavior and reduce code duplication across different FastGPT applications.

## Package Structure

### `@fastgpt/global`

The global package contains shared types, constants, utilities, and business logic that are used across all FastGPT projects.

**Key Contents:**
- **Common Types**: Shared TypeScript interfaces and type definitions
- **Error Codes**: Standardized error handling and error code definitions
- **AI Models**: AI service interfaces, model configurations, and prompt templates
- **App Types**: Core application types for chats, datasets, workflows, and tools
- **Utilities**: Common utility functions for strings, files, dates, and system operations
- **Constants**: Global constants used throughout the application

**Dependencies:** Minimal external dependencies, focuses on pure TypeScript functionality

### `@fastgpt/service`

The service package contains backend business logic, database models, API controllers, and workflow engine implementations.

**Key Contents:**
- **Database Models**: MongoDB/Mongoose models and database schemas
- **API Controllers**: Server-side business logic and request handling
- **Workflow Engine**: Core workflow execution engine with node types and execution limits
- **AI Service Integration**: Unified interface for multiple LLM providers and vectorization
- **Template System**: Application template management and processing
- **File Processing**: Support for PDF, Word, Excel, images, and other file formats
- **Queue Management**: Background job processing and task queues

**Dependencies:** Database connectors (MongoDB, PostgreSQL), AI SDKs, file processing libraries

### `@fastgpt/web`

The web package contains shared frontend components, hooks, styles, and internationalization support for React applications.

**Key Contents:**
- **React Components**: Reusable UI components built with Chakra UI
- **Custom Hooks**: Shared React hooks for common functionality
- **Styling**: Chakra UI theme configuration and custom styles
- **Internationalization**: i18n support files and translation utilities
- **State Management**: Zustand stores and React Context providers
- **Form Components**: Form validation and input components
- **Charts and Visualizations**: Recharts-based data visualization components

**Dependencies:** React, Chakra UI, React Query, Lexical editor, various UI libraries

## Package Dependencies

```
@fastgpt/service → @fastgpt/global
@fastgpt/web → @fastgpt/global
```

The `global` package has no internal dependencies, while `service` and `web` depend on `global` for shared types and utilities.

## Usage in Projects

### Import Patterns

Use workspace references to import packages in projects:

```typescript
// Import from global package
import { type ChatItem, ErrorCode } from '@fastgpt/global';

// Import from service package
import { ChatService, WorkflowEngine } from '@fastgpt/service';

// Import from web package
import { Button, Input, MyModal } from '@fastgpt/web';
```

### Project Integration

Projects reference these packages through pnpm workspace configuration:

- `projects/app/` - Main NextJS application uses all three packages
- `projects/sandbox/` - NestJS sandbox uses `@fastgpt/global` and `@fastgpt/service`
- `projects/mcp_server/` - MCP server uses `@fastgpt/global` and `@fastgpt/service`

## Development Guidelines

### Modifying Packages

1. **Backward Compatibility**: Maintain backward compatibility when changing shared types
2. **Version Management**: Use workspace references (`workspace:*`) for internal dependencies
3. **Testing**: Write comprehensive tests for package functionality
4. **Documentation**: Update type definitions and add JSDoc comments
5. **Code Review**: All package changes require thorough review due to cross-project impact

### Best Practices

- Keep packages focused and avoid circular dependencies
- Use TypeScript strict mode for all packages
- Follow ESLint and Prettier configurations
- Add meaningful error messages and proper error handling
- Write unit tests for utility functions and core logic

### Building Packages

Packages are built automatically as part of the monorepo build process:

```bash
# Build all packages
pnpm build

# Test packages
pnpm test

# Lint packages
pnpm lint
```

## Workspace Configuration

This directory uses pnpm workspace configuration defined in the root `pnpm-workspace.yaml`. The packages are automatically linked across the monorepo, allowing seamless imports and development.

## Version Management

All packages use a shared version (`1.0.0`) and are versioned together as part of the FastGPT monorepo. Internal dependencies use workspace references to ensure compatibility during development.

## Architecture Benefits

- **Code Reuse**: Shared functionality across multiple projects
- **Type Safety**: Consistent TypeScript types across the entire platform
- **Maintenance**: Centralized updates and bug fixes
- **Development Efficiency**: Faster development with pre-built components
- **Consistency**: Unified behavior and styling across applications