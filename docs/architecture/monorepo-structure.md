# Monorepo Structure

## Introduction

FastGPT uses a sophisticated monorepo architecture managed by pnpm workspaces. This structure provides excellent code reuse, consistent tooling, and atomic commits across the entire codebase while maintaining clear separation of concerns between different components.

## Overview Structure

```
fastgpt/
├── packages/               # Shared libraries (9 directories)
│   ├── global/            # Global types, constants, utilities
│   ├── service/           # Backend services and business logic
│   ├── web/               # Shared frontend components
│   └── templates/         # Application templates
├── projects/              # Applications (4 directories)
│   ├── app/               # Main NextJS web application
│   ├── sandbox/           # Code execution sandbox (NestJS)
│   ├── mcp_server/        # Model Context Protocol server
│   └── marketplace/       # Template marketplace
├── plugins/               # External plugins
│   ├── model/             # AI model plugins
│   └── webcrawler/        # Web scraping plugins
├── deploy/                # Deployment configurations
│   ├── docker/            # Docker configurations
│   └── helm/              # Kubernetes Helm charts
├── test/                  # Centralized test utilities
└── document/              # Documentation site
```

## Packages (Shared Libraries)

### packages/global/
**Purpose**: Shared types, constants, and utilities used across all projects

```
packages/global/
├── openapi/               # API contracts and schemas
│   ├── v1/               # API version 1 definitions
│   └── schemas/          # JSON schema definitions
├── ai/                   # AI-related types and interfaces
│   ├── llm/              # LLM provider types
│   └── embedding/        # Embedding types
├── constants/            # Global constants
│   ├── error.ts         # Error codes and messages
│   └── workflow.ts      # Workflow constants
├── utils/               # Utility functions
│   ├── array.ts         # Array utilities
│   ├── object.ts        # Object utilities
│   └── string.ts        # String utilities
└── types/               # TypeScript type definitions
    ├── api.ts           # API request/response types
    ├── database.ts      # Database types
    └── user.ts          # User management types
```

**Key Files**:
- `packages/global/openapi/v1/api.yaml` - OpenAPI specification
- `packages/global/types/workflow.ts` - Workflow type definitions
- `packages/global/constants/config.ts` - Configuration constants

### packages/service/
**Purpose**: Backend services, database models, and business logic

```
packages/service/
├── core/                 # Core business logic
│   ├── ai/              # AI service integration
│   │   ├── llm/         # LLM provider implementations
│   │   ├── embedding/   # Embedding services
│   │   └── config/      # AI configuration
│   ├── dataset/         # Knowledge base management
│   │   ├── read.ts      # Document processing
│   │   ├── split.ts     # Text chunking
│   │   └── search.ts    # Search functionality
│   ├── workflow/        # Workflow engine
│   │   ├── engine/      # Execution engine
│   │   ├── nodes/       # Workflow node types
│   │   └── io/          # Input/output handling
│   └── plugin/          # Plugin management
├── common/              # Common utilities
│   ├── mongo/           # MongoDB utilities
│   │   ├── models/      # Database models
│   │   └── schemas/     # Mongoose schemas
│   ├── vector/          # Vector database abstraction
│   │   ├── pg.ts        # PostgreSQL implementation
│   │   └── milvus.ts    # Milvus implementation
│   └── auth/            # Authentication utilities
├── database/            # Database connection management
│   ├── connection.ts    # Connection factory
│   └── migrations/      # Database migrations
└── api/                 # Shared API logic
    ├── middleware/      # Express middleware
    ├── controllers/     # Base controllers
    └── validators/      # Request validation
```

**Key Files**:
- `packages/service/core/workflow/engine/index.ts` - Workflow execution engine
- `packages/service/core/ai/llm/index.ts` - LLM abstraction layer
- `packages/service/common/mongo/models/index.ts` - Database models

### packages/web/
**Purpose**: Shared frontend components, styles, and utilities

```
packages/web/
├── components/          # Reusable React components
│   ├── ui/             # Basic UI components
│   │   ├── Button/     # Button variants
│   │   ├── Modal/      # Modal components
│   │   └── Input/      # Input fields
│   ├── workflow/       # Workflow-specific components
│   │   ├── NodeEditor/ # Node editing interface
│   │   └── FlowCanvas/ # Canvas for workflows
│   └── layout/         # Layout components
├── hooks/              # Custom React hooks
│   ├── useWorkflow.ts  # Workflow state management
│   ├── useChat.ts      # Chat functionality
│   └── useApi.ts       # API communication
├── styles/             # Styling system
│   ├── theme.ts        # Chakra UI theme
│   └── global.css      # Global styles
├── i18n/               # Internationalization
│   ├── en/             # English translations
│   ├── zh/             # Chinese translations
│   └── ja/             # Japanese translations
├── stores/             # Zustand stores
│   ├── workflow.ts     # Workflow state
│   ├── app.ts          # Application state
│   └── user.ts         # User state
└── utils/              # Frontend utilities
    ├── format.ts       # Formatting utilities
    └── validation.ts   # Form validation
```

**Key Files**:
- `packages/web/components/workflow/FlowCanvas/index.tsx` - Workflow canvas
- `packages/web/styles/theme.ts` - UI theme configuration
- `packages/web/i18n/index.ts` - Internationalization setup

### packages/templates/
**Purpose**: Application templates for the marketplace

```
packages/templates/
├── chatbot/            # Chatbot templates
│   ├── customer-service/
│   └── qa-bot/
├── workflow/           # Workflow templates
│   ├── data-processing/
│   └── content-generation/
└── deployment/         # Deployment templates
    ├── docker/
    └── kubernetes/
```

## Projects (Applications)

### projects/app/
**Purpose**: Main FastGPT web application

```
projects/app/
├── src/
│   ├── pages/          # NextJS pages
│   │   ├── api/        # API routes
│   │   │   ├── v1/     # Versioned API endpoints
│   │   │   └── auth/   # Authentication endpoints
│   │   ├── app/        # App Router pages (if using)
│   │   └── chat/       # Chat interface pages
│   ├── components/     # App-specific components
│   │   ├── layout/     # Layout components
│   │   ├── forms/      # Form components
│   │   └── modals/     # Modal components
│   ├── service/        # Backend service layer
│   │   ├── api/        # API clients
│   │   ├── auth/       # Authentication logic
│   │   └── websocket/  # WebSocket handlers
│   └── utils/          # App utilities
├── data/               # Static data
│   ├── config.json     # App configuration
│   └── icons/          # Icon assets
├── public/             # Public assets
└── test/               # App-specific tests
```

**Key Files**:
- `projects/app/src/pages/api/v1/workflow/run.ts` - Workflow execution API
- `projects/app/src/service/auth/index.ts` - Authentication service
- `projects/app/data/config.json` - Application configuration

### projects/sandbox/
**Purpose**: Isolated code execution environment

```
projects/sandbox/
├── src/
│   ├── app/            # NestJS application
│   │   ├── modules/    # Feature modules
│   │   │   ├── code/   # Code execution module
│   │   │   └── python/ # Python runtime
│   │   └── guards/     # Security guards
│   ├── code/           # Code execution logic
│   │   ├── executor.ts # Code runner
│   │   └── sandbox.ts  # Isolation logic
│   └── utils/          # Sandbox utilities
├── test/               # Sandbox tests
└── Dockerfile          # Container definition
```

**Key Files**:
- `projects/sandbox/src/app/modules/code/executor.ts` - Code execution
- `projects/sandbox/src/app/guards/resource.guard.ts` - Resource limits

### projects/mcp_server/
**Purpose**: Model Context Protocol server implementation

```
projects/mcp_server/
├── src/
│   ├── server.ts       # MCP server entry
│   ├── handlers/       # MCP request handlers
│   ├── tools/          # MCP tool implementations
│   └── resources/      # MCP resource handlers
├── package.json        # Dependencies
└── bun.lockb          # Bun lockfile
```

**Key Files**:
- `projects/mcp_server/src/server.ts` - MCP server implementation
- `projects/mcp_server/src/tools/ai.ts` - AI tool implementations

### projects/marketplace/
**Purpose**: Template marketplace application

```
projects/marketplace/
├── src/
│   ├── pages/          # Marketplace pages
│   ├── components/     # Marketplace components
│   └── api/            # Marketplace API
└── data/               # Template data
```

## Workspace Configuration

### Root package.json
```json
{
  "name": "fastgpt-workspace",
  "private": true,
  "workspaces": [
    "packages/*",
    "projects/*"
  ],
  "scripts": {
    "dev": "pnpm --parallel -r dev",
    "build": "pnpm -r build",
    "test": "pnpm -r test",
    "lint": "eslint packages/*/src/**/*.ts projects/*/src/**/*.ts --fix"
  },
  "devDependencies": {
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.0.0",
    "prettier": "^3.0.0",
    "typescript": "^5.1.3"
  }
}
```

### pnpm-workspace.yaml
```yaml
packages:
  - 'packages/*'
  - 'projects/*'
  - 'plugins/*'
```

## Dependencies Management

### Internal Dependencies
Projects reference packages using workspace references:

```json
{
  "dependencies": {
    "@fastgpt/global": "workspace:*",
    "@fastgpt/service": "workspace:*",
    "@fastgpt/web": "workspace:*"
  }
}
```

### Version Management
- Use `workspace:*` for internal dependencies
- External dependencies defined in package.json
- Consistent versions across packages
- Automated updates with pnpm

## Build System

### Build Configuration

```json
{
  "scripts": {
    "build": "pnpm -r build",
    "build:app": "cd projects/app && pnpm build",
    "build:sandbox": "cd projects/sandbox && pnpm build",
    "build:mcp": "cd projects/mcp_server && bun build"
  }
}
```

### TypeScript Configuration
Base configuration in root `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "node",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "paths": {
      "@fastgpt/global": ["packages/global/src"],
      "@fastgpt/service": ["packages/service/src"],
      "@fastgpt/web": ["packages/web/src"]
    }
  },
  "references": [
    { "path": "./packages/global" },
    { "path": "./packages/service" },
    { "path": "./packages/web" },
    { "path": "./projects/app" },
    { "path": "./projects/sandbox" },
    { "path": "./projects/mcp_server" }
  ]
}
```

## Testing Strategy

### Test Organization
```
test/
├── unit/                # Unit tests
│   ├── packages/        # Package unit tests
│   └── projects/        # Project unit tests
├── integration/         # Integration tests
│   ├── api/            # API integration tests
│   └── workflow/       # Workflow integration tests
├── e2e/                # End-to-end tests
│   ├── scenarios/      # Test scenarios
│   └── fixtures/       # Test data
└── utils/              # Test utilities
    ├── database.ts     # Database test utils
    └── mocks.ts        # Mock implementations
```

### Test Configuration
```json
{
  "scripts": {
    "test": "vitest",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test"
  },
  "vitest": {
    "config": "test/vitest.config.ts"
  }
}
```

## Development Workflow

### 1. Setting Up Development
```bash
# Clone repository
git clone https://github.com/labring/FastGPT.git

# Install dependencies
pnpm install

# Start development
pnpm dev
```

### 2. Adding New Packages
```bash
# Create new package directory
mkdir packages/new-package

# Initialize package
cd packages/new-package
pnpm init

# Add to workspace
# Package name will be @fastgpt/new-package
```

### 3. Cross-Package Development
- Use workspace references for internal imports
- Run `pnpm dev` to start all services
- Changes in packages are hot-reloaded
- Atomic commits across packages

## Code Sharing Patterns

### 1. Shared Types
Define types in `packages/global/types`:
```typescript
// packages/global/types/workflow.ts
export interface WorkflowNode {
  id: string;
  type: string;
  data: Record<string, any>;
  position: { x: number; y: number };
}
```

### 2. Shared Utilities
Create utilities in `packages/global/utils`:
```typescript
// packages/global/utils/validation.ts
export function validateEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### 3. Shared Components
Build components in `packages/web/components`:
```typescript
// packages/web/components/ui/Button.tsx
export interface ButtonProps {
  variant: 'primary' | 'secondary';
  onClick: () => void;
  children: React.ReactNode;
}
```

## Best Practices

### 1. Package Boundaries
- Clear separation between packages
- Minimal dependencies between packages
- No circular dependencies
- Well-defined APIs between packages

### 2. Version Management
- Use workspace:* for internal deps
- Keep external deps consistent
- Update dependencies regularly
- Use semantic versioning

### 3. Code Organization
- Follow established patterns
- Use TypeScript paths for imports
- Keep related code together
- Document package APIs

### 4. Build Performance
- Only build what's needed
- Use incremental builds
- Cache build artifacts
- Optimize bundle sizes

## Migration Strategy

### Migrating from Monolith
1. **Identify Boundaries**: Find logical splits
2. **Extract Packages**: Move code to packages
3. **Update Imports**: Use workspace references
4. **Adjust Builds**: Update build configuration
5. **Test Thoroughly**: Ensure functionality

### Adding New Features
1. **Determine Location**: Package vs project
2. **Check Dependencies**: Minimize coupling
3. **Follow Patterns**: Use established structure
4. **Add Tests**: Maintain coverage
5. **Update Docs**: Document changes

## Troubleshooting

### Common Issues

1. **Workspace Resolution**
   ```bash
   # Clear workspace cache
   pnpm store prune
   ```

2. **TypeScript Path Issues**
   ```bash
   # Rebuild type references
   pnpm build --filter=@fastgpt/global
   ```

3. **Dependency Conflicts**
   ```bash
   # Check for conflicts
   pnpm ls --depth=0
   ```

4. **Build Failures**
   ```bash
   # Clean build
   pnpm clean && pnpm build
   ```

This monorepo structure provides a solid foundation for scalable development while maintaining code reuse and consistency across the entire FastGPT platform.