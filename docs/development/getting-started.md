# Getting Started with FastGPT Development

## Introduction

This guide will help you set up a development environment for FastGPT and understand the development workflow. FastGPT is a monorepo project that uses pnpm workspaces to manage multiple packages and applications.

## Prerequisites

### System Requirements

- **Node.js**: Version 18.16.0 or higher
- **pnpm**: Version 9.0.0 or higher
- **Python**: 3.11+ (for sandbox functionality)
- **Docker**: Latest version for container services
- **Git**: Latest version

### Database Requirements (choose one or more)
- **MongoDB**: 5.0+ (primary database)
- **PostgreSQL**: 13+ with pgvector extension (optional vector storage)
- **Milvus**: 2.0+ (optional high-performance vector storage)

### Development Tools (Recommended)
- **VS Code**: With TypeScript, ESLint, and Prettier extensions
- **Postman**: For API testing
- **MongoDB Compass**: For database visualization
- **Docker Desktop**: For container management

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/labring/FastGPT.git
cd FastGPT
```

### 2. Install Dependencies

```bash
# Install pnpm if not already installed
npm install -g pnpm

# Install all dependencies for the monorepo
pnpm install
```

### 3. Environment Setup

Create a `.env.local` file in the `projects/app` directory:

```bash
cp projects/app/.env.example projects/app/.env.local
```

Edit the `.env.local` file with your configuration:

```env
# Database Configuration
MONGODB_URI=mongodb://localhost:27017/fastgpt
PG_URI=postgresql://user:password@localhost:5432/fastgpt

# AI Service Configuration
OPENAI_API_KEY=your_openai_api_key
OPENAI_BASE_URL=https://api.openai.com/v1
ANTHROPIC_API_KEY=your_anthropic_api_key

# Other Configuration
REDIS_URL=redis://localhost:6379
SECRET_KEY=your_secret_key_here
```

### 4. Start Development Services

Using Docker Compose for external services:

```bash
# Start MongoDB, Redis, and other services
docker-compose -f docker-compose.dev.yml up -d
```

### 5. Start Development Environment

```bash
# Start all projects in development mode
pnpm dev
```

This will start:
- **Main App** (NextJS): http://localhost:3000
- **Sandbox Service** (NestJS): http://localhost:3001
- **MCP Server** (Bun): http://localhost:3002
- **Documentation Site**: http://localhost:3003

## Project Structure

### Monorepo Overview

```
fastgpt/
├── packages/               # Shared libraries
│   ├── global/            # Types, constants, utilities
│   ├── service/           # Backend services and logic
│   ├── web/               # Frontend components and styles
│   └── templates/         # Application templates
├── projects/              # Applications
│   ├── app/               # Main NextJS application
│   ├── sandbox/           # Code execution sandbox
│   ├── mcp_server/        # MCP protocol server
│   └── marketplace/       # Template marketplace
├── plugins/               # External plugins
├── test/                  # Test files and utilities
└── document/              # Documentation
```

### Key Directories

#### Main Application (`projects/app/`)
```
src/
├── pages/
│   ├── api/              # API routes
│   ├── app/              # App Router pages
│   └── chat/             # Chat interface
├── components/           # React components
├── service/              # Backend services
├── utils/                # Utility functions
└── types/                # TypeScript types
```

#### Backend Services (`packages/service/`)
```
src/
├── core/
│   ├── ai/               # AI service integration
│   ├── workflow/         # Workflow engine
│   └── dataset/          # Knowledge base management
├── common/
│   ├── mongo/            # MongoDB models
│   └── vector/           # Vector database abstraction
└── api/                  # API utilities
```

## Development Workflow

### 1. Code Organization

#### Creating New Components

```typescript
// packages/web/components/my-component/index.tsx
import { Box, Text } from '@chakra-ui/react';

interface MyComponentProps {
  title: string;
  children: React.ReactNode;
}

export const MyComponent: React.FC<MyComponentProps> = ({ title, children }) => {
  return (
    <Box p={4} borderWidth={1} borderRadius="md">
      <Text fontWeight="bold" mb={2}>{title}</Text>
      {children}
    </Box>
  );
};
```

#### Adding API Endpoints

```typescript
// projects/app/src/pages/api/v1/my-endpoint.ts
import { NextApiRequest, NextApiResponse } from 'next';
import { connectToDatabase } from '@fastgpt/service/common/mongo';

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  if (req.method !== 'GET') {
    return res.status(405).json({ error: 'Method not allowed' });
  }

  try {
    await connectToDatabase();
    // Your logic here
    res.json({ data: 'success' });
  } catch (error) {
    console.error(error);
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

### 2. Working with Workflows

#### Creating Custom Nodes

```typescript
// packages/service/core/workflow/nodes/my-node.ts
import { NodeExecutor } from '@fastgpt/service/core/workflow/engine';

export class MyNodeExecutor implements NodeExecutor {
  async execute(node, context, runtime) {
    const { input } = node.data;

    // Process input
    const result = await this.process(input);

    return {
      status: 'success',
      outputs: { result }
    };
  }

  private async process(input: any): Promise<any> {
    // Your processing logic
  }
}
```

### 3. Database Operations

#### Using MongoDB Models

```typescript
import { UserModel } from '@fastgpt/service/common/mongo/models';

// Create user
const user = new UserModel({
  username: 'john_doe',
  email: 'john@example.com',
  passwordHash: await hashPassword('password123')
});
await user.save();

// Query users
const users = await UserModel.find({ status: 'active' })
  .limit(10)
  .sort({ createdAt: -1 });
```

#### Working with Vector Databases

```typescript
import { VectorStore } from '@fastgpt/service/common/vector';

// Insert embeddings
const vectorStore = new VectorStore();
await vectorStore.insert('my_collection', [
  {
    id: 'doc1',
    content: 'This is the document content',
    embedding: [0.1, 0.2, 0.3, ...],
    metadata: { source: 'pdf' }
  }
]);

// Search similar documents
const results = await vectorStore.search('my_collection', {
  vector: queryEmbedding,
  topK: 5,
  filters: { source: 'pdf' }
});
```

## Testing

### Running Tests

```bash
# Run all tests
pnpm test

# Run tests for specific package
pnpm --filter @fastgpt/service test

# Run tests with coverage
pnpm test --coverage

# Run E2E tests
pnpm test:e2e
```

### Writing Tests

#### Unit Tests

```typescript
// test/unit/my-component.test.ts
import { render, screen } from '@testing-library/react';
import { MyComponent } from '@fastgpt/web/components/my-component';

describe('MyComponent', () => {
  it('renders title correctly', () => {
    render(<MyComponent title="Test Title">Content</MyComponent>);
    expect(screen.getByText('Test Title')).toBeInTheDocument();
  });
});
```

#### Integration Tests

```typescript
// test/integration/api.test.ts
import { createMocks } from 'node-mocks-http';
import handler from '../../projects/app/src/pages/api/v1/my-endpoint';

describe('/api/v1/my-endpoint', () => {
  it('returns data successfully', async () => {
    const { req, res } = createMocks({ method: 'GET' });
    await handler(req, res);

    expect(res._getStatusCode()).toBe(200);
    expect(JSON.parse(res._getData())).toEqual({ data: 'success' });
  });
});
```

## Code Quality

### Linting and Formatting

```bash
# Run ESLint
pnpm lint

# Auto-fix linting issues
pnpm lint:fix

# Format code with Prettier
pnpm format-code

# Check for type errors
pnpm type-check
```

### Pre-commit Hooks

FastGPT uses Husky for pre-commit hooks that:
- Run linting and formatting
- Execute tests
- Prevent broken code from being committed

## Common Development Tasks

### 1. Adding New AI Provider

```typescript
// packages/service/core/ai/providers/my-provider.ts
import { BaseAIProvider } from '@fastgpt/service/core/ai/provider';

export class MyAIProvider extends BaseAIProvider {
  async chatCompletion(request) {
    // Implementation
  }

  async embeddings(input) {
    // Implementation
  }
}
```

### 2. Creating a Plugin

1. Create plugin directory: `plugins/my-plugin/`
2. Initialize package: `cd plugins/my-plugin && npm init`
3. Implement plugin interface
4. Add Dockerfile for deployment
5. Register plugin in system

### 3. Updating Configuration

```typescript
// projects/app/data/config.json
{
  "aiProviders": [
    {
      "id": "my-provider",
      "name": "My AI Provider",
      "endpoint": "https://api.my-ai.com",
      "models": ["my-model-1", "my-model-2"]
    }
  ]
}
```

## Debugging

### Debugging Backend (Node.js)

```bash
# Start with debug flag
node --inspect-brk dist/server.js

# Or use VS Code debugger
# Add launch configuration to .vscode/launch.json
```

### Debugging Frontend

1. Open Chrome DevTools
2. Go to Sources tab
3. Navigate to webpack:///
4. Set breakpoints in your code

### Common Issues

1. **Port Conflicts**: Check if ports 3000-3002 are available
2. **Database Connection**: Verify MongoDB is running
3. **Environment Variables**: Ensure all required env vars are set
4. **Dependencies**: Try `pnpm clean && pnpm install`

## Contributing Guidelines

### Before Submitting

1. Create a feature branch from main
2. Write tests for new functionality
3. Ensure all tests pass
4. Update documentation if needed
5. Submit a pull request

### Code Review Process

1. Automated checks run on PR
2. Manual review by maintainers
3. Feedback and requested changes
4. Approval and merge

### Commit Messages

Follow conventional commits format:
- `feat: Add new feature`
- `fix: Fix bug in component`
- `docs: Update documentation`
- `style: Format code`
- `refactor: Refactor code`
- `test: Add tests`
- `chore: Update dependencies`

## Performance Optimization

### Frontend Optimization

1. Use React.memo for expensive components
2. Implement lazy loading for heavy components
3. Optimize bundle size with dynamic imports
4. Use Chakra UI's shouldForwardProp

### Backend Optimization

1. Implement caching with Redis
2. Use database indexes for queries
3. Batch database operations
4. Optimize vector search queries

## Resources

### Documentation
- [Official Documentation](https://doc.fastgpt.io/)
- [API Reference](./api/overview.md)
- [Architecture Guide](../architecture/overview.md)

### Community
- [GitHub Issues](https://github.com/labring/FastGPT/issues)
- [Discord Server](https://discord.gg/fastgpt)
- [Discussions](https://github.com/labring/FastGPT/discussions)

### Tools
- [Chakra UI Documentation](https://chakra-ui.com/docs)
- [NextJS Documentation](https://nextjs.org/docs)
- [MongoDB Documentation](https://docs.mongodb.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

## Getting Help

If you encounter issues:

1. Check the troubleshooting section
2. Search existing GitHub issues
3. Ask in the Discord community
4. Create a new issue with detailed information

Remember to include:
- Error messages
- Steps to reproduce
- Your environment details
- Expected vs actual behavior

Happy coding with FastGPT! 🚀