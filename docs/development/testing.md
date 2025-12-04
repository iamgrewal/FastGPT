# Testing Architecture and Practices

## Introduction

FastGPT employs a comprehensive testing strategy to ensure code quality, reliability, and maintainability. This document outlines the testing architecture, tools, patterns, and best practices used throughout the project.

## Testing Pyramid

```
        ┌─────────────────┐
        │   E2E Tests     │  <- Few, slow, high value
        │                 │     (Playwright)
        └─────────────────┘
      ┌─────────────────────┐
      │  Integration Tests │  <- Medium number, medium speed
      │                     │     (Vitest, Supertest)
      └─────────────────────┘
    ┌─────────────────────────┐
    │      Unit Tests         │  <- Many, fast, focused
    │                         │     (Vitest)
    └─────────────────────────┘
```

## Testing Tools and Configuration

### 1. Vitest - Unit & Integration Testing

**Configuration** (`vitest.config.ts`):
```typescript
import { defineConfig } from 'vitest/config';
import { resolve } from 'path';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    setupFiles: ['./test/setup.ts'],
    coverage: {
      provider: 'c8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'test/',
        '**/*.d.ts',
        '**/*.config.*',
        'coverage/'
      ],
      thresholds: {
        global: {
          branches: 80,
          functions: 80,
          lines: 80,
          statements: 80
        }
      }
    },
    threads: true,
    maxConcurrency: 5,
    testTimeout: 30000,
    hookTimeout: 30000
  },
  resolve: {
    alias: {
      '@fastgpt/global': resolve(__dirname, 'packages/global/src'),
      '@fastgpt/service': resolve(__dirname, 'packages/service/src'),
      '@fastgpt/web': resolve(__dirname, 'packages/web/src')
    }
  },
  plugins: [tsconfigPaths()]
});
```

### 2. Test Setup

**Setup File** (`test/setup.ts`):
```typescript
import { beforeAll, afterAll, beforeEach, afterEach } from 'vitest';
import { MongoMemoryServer } from 'mongodb-memory-server';
import mongoose from 'mongoose';
import { Redis } from 'ioredis';

// Test environment variables
process.env.NODE_ENV = 'test';
process.env.LOG_LEVEL = 'error';

let mongoServer: MongoMemoryServer;
let redis: Redis;

beforeAll(async () => {
  // Start in-memory MongoDB
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);

  // Start in-memory Redis
  redis = new Redis({
    host: 'localhost',
    port: 6380, // Different port to avoid conflicts
    lazyConnect: true
  });
  await redis.connect();
});

afterAll(async () => {
  // Cleanup MongoDB
  await mongoose.disconnect();
  await mongoServer.stop();

  // Cleanup Redis
  await redis.disconnect();
});

beforeEach(async () => {
  // Clear all collections before each test
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }

  // Clear Redis
  await redis.flushall();
});

afterEach(() => {
  // Reset all mocks
  vitest.clearAllMocks();
});

// Global test utilities
global.testUtils = {
  generateId: () => Math.random().toString(36).substring(7),
  generateUser: (overrides = {}) => ({
    id: global.testUtils.generateId(),
    name: 'Test User',
    email: `test${Date.now()}@example.com`,
    ...overrides
  })
};
```

## Testing Patterns

### 1. Unit Testing

#### Testing Pure Functions
```typescript
// src/utils/validation.ts
export const isValidEmail = (email: string): boolean => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
};

// test/unit/utils/validation.test.ts
import { describe, it, expect } from 'vitest';
import { isValidEmail } from '@fastgpt/service/utils/validation';

describe('isValidEmail', () => {
  it('returns true for valid email addresses', () => {
    expect(isValidEmail('user@example.com')).toBe(true);
    expect(isValidEmail('test.email+tag@domain.co.uk')).toBe(true);
  });

  it('returns false for invalid email addresses', () => {
    expect(isValidEmail('')).toBe(false);
    expect(isValidEmail('invalid')).toBe(false);
    expect(isValidEmail('@domain.com')).toBe(false);
    expect(isValidEmail('user@')).toBe(false);
  });

  it('handles edge cases', () => {
    expect(isValidEmail('a@b.c')).toBe(true);
    expect(isValidEmail('user.name@domain.com')).toBe(true);
    expect(isValidEmail('user@domain')).toBe(false);
  });
});
```

#### Testing with Mocks
```typescript
// src/services/userService.ts
export class UserService {
  async createUser(userData: CreateUserData): Promise<User> {
    const user = new UserModel(userData);
    return user.save();
  }

  async getUserById(id: string): Promise<User | null> {
    return UserModel.findById(id);
  }
}

// test/unit/services/userService.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { UserService } from '@fastgpt/service/services/userService';
import { UserModel } from '@fastgpt/service/common/mongo/models';

// Mock the model
vi.mock('@fastgpt/service/common/mongo/models', () => ({
  UserModel: {
    prototype: {
      save: vi.fn()
    },
    findById: vi.fn(),
    constructor: vi.fn().mockImplementation(function (data) {
      this.data = data;
    })
  }
}));

describe('UserService', () => {
  let userService: UserService;

  beforeEach(() => {
    userService = new UserService();
    vi.clearAllMocks();
  });

  describe('createUser', () => {
    it('creates a user with valid data', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        passwordHash: 'hashedpassword'
      };

      const mockUser = { ...userData, _id: 'user123' };
      (UserModel.prototype.save as any).mockResolvedValue(mockUser);

      const result = await userService.createUser(userData);

      expect(UserModel).toHaveBeenCalledWith(userData);
      expect(UserModel.prototype.save).toHaveBeenCalled();
      expect(result).toEqual(mockUser);
    });

    it('handles save errors', async () => {
      const userData = {
        name: 'John Doe',
        email: 'john@example.com',
        passwordHash: 'hashedpassword'
      };

      const error = new Error('Database error');
      (UserModel.prototype.save as any).mockRejectedValue(error);

      await expect(userService.createUser(userData)).rejects.toThrow('Database error');
    });
  });

  describe('getUserById', () => {
    it('returns user when found', async () => {
      const userId = 'user123';
      const mockUser = { _id: userId, name: 'John Doe' };
      (UserModel.findById as any).mockResolvedValue(mockUser);

      const result = await userService.getUserById(userId);

      expect(UserModel.findById).toHaveBeenCalledWith(userId);
      expect(result).toEqual(mockUser);
    });

    it('returns null when user not found', async () => {
      const userId = 'nonexistent';
      (UserModel.findById as any).mockResolvedValue(null);

      const result = await userService.getUserById(userId);

      expect(UserModel.findById).toHaveBeenCalledWith(userId);
      expect(result).toBeNull();
    });
  });
});
```

### 2. Integration Testing

#### API Endpoint Testing
```typescript
// test/integration/api/users.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from 'vitest';
import request from 'supertest';
import { app } from '../../../projects/app/src/app';

describe('POST /api/v1/users', () => {
  let authToken: string;

  beforeEach(async () => {
    // Create and login user to get auth token
    const loginResponse = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'admin@example.com',
        password: 'password123'
      });

    authToken = loginResponse.body.data.token;
  });

  it('creates a user with valid data', async () => {
    const userData = {
      name: 'Jane Doe',
      email: 'jane@example.com',
      password: 'password123',
      role: 'user'
    };

    const response = await request(app)
      .post('/api/v1/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send(userData)
      .expect(201);

    expect(response.body).toMatchObject({
      success: true,
      data: {
        name: userData.name,
        email: userData.email,
        role: userData.role
      }
    });

    expect(response.body.data).not.toHaveProperty('password');
    expect(response.body.data).toHaveProperty('id');
  });

  it('validates required fields', async () => {
    const response = await request(app)
      .post('/api/v1/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send({})
      .expect(400);

    expect(response.body).toMatchObject({
      success: false,
      error: {
        code: 'VALIDATION_ERROR'
      }
    });

    expect(response.body.error.details).toContainEqual({
      field: 'name',
      message: 'Name is required'
    });

    expect(response.body.error.details).toContainEqual({
      field: 'email',
      message: 'Email is required'
    });
  });

  it('rejects invalid email format', async () => {
    const userData = {
      name: 'Jane Doe',
      email: 'invalid-email',
      password: 'password123'
    };

    const response = await request(app)
      .post('/api/v1/users')
      .set('Authorization', `Bearer ${authToken}`)
      .send(userData)
      .expect(400);

    expect(response.body.error.details).toContainEqual({
      field: 'email',
      message: 'Invalid email format'
    });
  });
});
```

#### Database Integration Testing
```typescript
// test/integration/database/user.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { UserModel } from '@fastgpt/service/common/mongo/models';

describe('UserModel Integration', () => {
  beforeEach(async () => {
    await UserModel.deleteMany({});
  });

  it('saves and retrieves user', async () => {
    const userData = {
      name: 'Test User',
      email: 'test@example.com',
      passwordHash: 'hashedpassword'
    };

    // Save user
    const user = new UserModel(userData);
    await user.save();

    // Retrieve user
    const retrieved = await UserModel.findById(user._id);

    expect(retrieved).toBeDefined();
    expect(retrieved?.name).toBe(userData.name);
    expect(retrieved?.email).toBe(userData.email);
    expect(retrieved?.passwordHash).toBe(userData.passwordHash);
    expect(retrieved?.createdAt).toBeInstanceOf(Date);
  });

  it('enforces unique email constraint', async () => {
    const userData = {
      name: 'Test User',
      email: 'test@example.com',
      passwordHash: 'hashedpassword'
    };

    // Create first user
    await new UserModel(userData).save();

    // Try to create second user with same email
    const duplicateUser = new UserModel({
      ...userData,
      name: 'Another User'
    });

    await expect(duplicateUser.save()).rejects.toThrow(/duplicate key/);
  });

  it('excludes password hash in JSON output', async () => {
    const userData = {
      name: 'Test User',
      email: 'test@example.com',
      passwordHash: 'hashedpassword'
    };

    const user = new UserModel(userData);
    await user.save();

    const userJson = user.toJSON();

    expect(userJson).not.toHaveProperty('passwordHash');
    expect(userJson).toHaveProperty('name');
    expect(userJson).toHaveProperty('email');
  });
});
```

### 3. Workflow Testing

#### Workflow Execution Testing
```typescript
// test/integration/workflow/execution.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { WorkflowEngine } from '@fastgpt/service/core/workflow/engine';
import { AIProviderManager } from '@fastgpt/service/core/ai/manager';

// Mock AI provider
vi.mock('@fastgpt/service/core/ai/manager', () => ({
  AIProviderManager: vi.fn().mockImplementation(() => ({
    chatCompletion: vi.fn().mockResolvedValue({
      content: 'Mock AI response',
      usage: { tokens: 10 }
    })
  }))
}));

describe('Workflow Engine', () => {
  let workflowEngine: WorkflowEngine;

  beforeEach(() => {
    workflowEngine = new WorkflowEngine();
  });

  it('executes simple chat workflow', async () => {
    const workflow = {
      id: 'test-workflow',
      nodes: [
        {
          id: 'start',
          type: 'workflowStart',
          data: {
            inputs: {
              message: 'Hello, world!'
            }
          }
        },
        {
          id: 'ai',
          type: 'ai',
          data: {
            inputs: {
              message: '{{start.message}}',
              model: 'gpt-3.5-turbo'
            }
          },
          position: { x: 200, y: 0 }
        }
      ],
      edges: [
        {
          id: 'e1',
          source: 'start',
          target: 'ai',
          sourceHandle: 'output',
          targetHandle: 'input'
        }
      ]
    };

    const result = await workflowEngine.execute(workflow, {
      message: 'Hello, world!'
    });

    expect(result.status).toBe('success');
    expect(result.outputs).toHaveProperty('response');
    expect(result.outputs.response).toBe('Mock AI response');
  });

  it('handles workflow errors gracefully', async () => {
    const workflow = {
      id: 'error-workflow',
      nodes: [
        {
          id: 'start',
          type: 'workflowStart',
          data: {}
        },
        {
          id: 'ai',
          type: 'ai',
          data: {
            inputs: {
              message: undefined // This should cause an error
            }
          }
        }
      ],
      edges: [
        {
          id: 'e1',
          source: 'start',
          target: 'ai'
        }
      ]
    };

    const result = await workflowEngine.execute(workflow);

    expect(result.status).toBe('error');
    expect(result.error).toBeDefined();
    expect(result.error.code).toBe('VALIDATION_ERROR');
  });
});
```

### 4. Component Testing

#### React Component Testing
```typescript
// test/unit/components/UserCard.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserCard } from '@fastgpt/web/components/UserCard';

const createTestQueryClient = () =>
  new QueryClient({
    defaultOptions: {
      queries: { retry: false },
      mutations: { retry: false }
    }
  });

const renderWithQuery = (component: React.ReactElement) => {
  const queryClient = createTestQueryClient();
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    avatar: 'https://example.com/avatar.jpg'
  };

  it('displays user information', () => {
    renderWithQuery(<UserCard user={mockUser} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
    expect(screen.getByAltText('John Doe')).toHaveAttribute('src', mockUser.avatar);
  });

  it('calls onEdit when edit button is clicked', async () => {
    const onEdit = vi.fn();
    renderWithQuery(<UserCard user={mockUser} onEdit={onEdit} />);

    fireEvent.click(screen.getByRole('button', { name: /edit/i }));

    await waitFor(() => {
      expect(onEdit).toHaveBeenCalledWith(mockUser);
    });
  });

  it('shows loading state', () => {
    renderWithQuery(<UserCard user={null} loading />);

    expect(screen.getByTestId('user-card-skeleton')).toBeInTheDocument();
  });

  it('handles missing user gracefully', () => {
    renderWithQuery(<UserCard user={null} />);

    expect(screen.getByText('User not available')).toBeInTheDocument();
  });
});
```

### 5. End-to-End Testing

#### Playwright E2E Tests
```typescript
// test/e2e/chat.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Chat Flow', () => {
  test.beforeEach(async ({ page }) => {
    // Login before each test
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'test@example.com');
    await page.fill('[data-testid="password-input"]', 'password123');
    await page.click('[data-testid="login-button"]');
    await expect(page).toHaveURL('/chat');
  });

  test('sends and receives chat messages', async ({ page }) => {
    // Type message
    await page.fill('[data-testid="chat-input"]', 'Hello, FastGPT!');

    // Send message
    await page.click('[data-testid="send-button"]');

    // Verify message appears in chat
    await expect(page.locator('[data-testid="message-user"]'))
      .toContainText('Hello, FastGPT!');

    // Wait for AI response
    await expect(page.locator('[data-testid="message-ai"]'))
      .toBeVisible({ timeout: 10000 });
  });

  test('creates new chat session', async ({ page }) => {
    // Click new chat button
    await page.click('[data-testid="new-chat-button"]');

    // Verify new chat is created
    await expect(page.locator('[data-testid="chat-input"]')).toBeVisible();
    await expect(page.locator('[data-testid="chat-history"]'))
      .toContainText('New Chat');
  });

  test('searches chat history', async ({ page }) => {
    // Send a message first
    await page.fill('[data-testid="chat-input"]', 'test message for search');
    await page.click('[data-testid="send-button"]');

    // Open search
    await page.click('[data-testid="search-button"]');
    await page.fill('[data-testid="search-input"]', 'test message');

    // Verify search results
    await expect(page.locator('[data-testid="search-result"]'))
      .toContainText('test message for search');
  });
});
```

## Test Organization

### Directory Structure
```
test/
├── unit/                     # Unit tests
│   ├── components/          # Component tests
│   ├── services/            # Service tests
│   ├── utils/               # Utility function tests
│   └── hooks/               # Custom hook tests
├── integration/             # Integration tests
│   ├── api/                 # API endpoint tests
│   ├── database/            # Database tests
│   └── workflow/            # Workflow tests
├── e2e/                     # End-to-end tests
│   ├── auth/                # Authentication flows
│   ├── chat/                # Chat functionality
│   └── workflow/            # Workflow creation/execution
├── fixtures/                # Test data
│   ├── users.json
│   ├── workflows.json
│   └── datasets.json
├── mocks/                   # Mock implementations
│   ├── aiProviders.ts
│   └── externalApis.ts
└── utils/                   # Test utilities
    ├── helpers.ts
    ├── factories.ts
    └── matchers.ts
```

### Test Factories
```typescript
// test/utils/factories.ts
import { faker } from '@faker-js/faker';

export class UserFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      name: faker.name.fullName(),
      email: faker.internet.email(),
      avatar: faker.image.avatar(),
      role: 'user',
      createdAt: faker.date.past(),
      ...overrides
    };
  }

  static createMany(count: number, overrides = {}) {
    return Array.from({ length: count }, () => this.create(overrides));
  }
}

export class WorkflowFactory {
  static create(overrides = {}) {
    return {
      id: faker.datatype.uuid(),
      name: faker.lorem.words(3),
      nodes: [
        {
          id: 'start',
          type: 'workflowStart',
          data: {}
        }
      ],
      edges: [],
      ...overrides
    };
  }
}
```

### Custom Matchers
```typescript
// test/utils/matchers.ts
import { expect } from 'vitest';

expect.extend({
  toBeValidUser(received) {
    const pass = (
      received &&
      typeof received.id === 'string' &&
      typeof received.name === 'string' &&
      typeof received.email === 'string' &&
      /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(received.email)
    );

    if (pass) {
      return {
        message: () => `expected ${received} not to be a valid user`,
        pass: true
      };
    } else {
      return {
        message: () => `expected ${received} to be a valid user`,
        pass: false
      };
    }
  },

  toBeWorkflowNode(received) {
    const pass = (
      received &&
      typeof received.id === 'string' &&
      typeof received.type === 'string' &&
      typeof received.data === 'object' &&
      typeof received.position === 'object'
    );

    if (pass) {
      return {
        message: () => `expected ${received} not to be a workflow node`,
        pass: true
      };
    } else {
      return {
        message: () => `expected ${received} to be a workflow node`,
        pass: false
      };
    }
  }
});

// Type extensions
declare module 'vitest' {
  interface Assertion<T = any> {
    toBeValidUser(): T;
    toBeWorkflowNode(): T;
  }
}
```

## Test Scripts

### Package.json Scripts
```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "test:ui": "vitest --ui",
    "test:run": "vitest run",
    "test:unit": "vitest run test/unit",
    "test:integration": "vitest run test/integration",
    "test:workflow": "vitest run test/integration/workflow",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

### CI/CD Testing
```yaml
# .github/workflows/test.yml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  unit-integration:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:5.0
        ports:
          - 27017:27017
      redis:
        image: redis:7
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Run linting
        run: pnpm lint

      - name: Run type checking
        run: pnpm type-check

      - name: Run unit tests
        run: pnpm test:unit

      - name: Run integration tests
        run: pnpm test:integration

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage/lcov.info

  e2e:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
          cache: 'pnpm'

      - name: Install dependencies
        run: pnpm install

      - name: Install Playwright
        run: pnpm exec playwright install --with-deps

      - name: Build application
        run: pnpm build

      - name: Run E2E tests
        run: pnpm test:e2e

      - name: Upload test results
        uses: actions/upload-artifact@v3
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

## Testing Best Practices

### 1. Writing Good Tests

#### AAA Pattern (Arrange, Act, Assert)
```typescript
// ✅ Good
it('calculates total price including tax', () => {
  // Arrange
  const price = 100;
  const tax = 0.1;
  const calculator = new PriceCalculator();

  // Act
  const total = calculator.calculateTotal(price, tax);

  // Assert
  expect(total).toBe(110);
});
```

#### Test One Thing
```typescript
// ✅ Good - Single assertion per test
it('validates email format', () => {
  const result = validator.email('user@example.com');
  expect(result).toBe(true);
});

it('rejects invalid email', () => {
  const result = validator.email('invalid-email');
  expect(result).toBe(false);
});

// ❌ Bad - Multiple assertions testing different things
it('validates email', () => {
  expect(validator.email('user@example.com')).toBe(true);
  expect(validator.email('invalid-email')).toBe(false);
  expect(validator.email('')).toBe(false); // Testing empty string
});
```

#### Use Descriptive Test Names
```typescript
// ✅ Good
it('returns 404 when user not found');
it('creates user with valid data');
it('prevents duplicate email registration');

// ❌ Bad
it('works');
it('test user creation');
it('returns error');
```

### 2. Managing Test Data

#### Use Builders for Complex Objects
```typescript
// ✅ Good with builder pattern
const user = UserFactory.create({
  email: 'specific@example.com',
  role: 'admin'
});

// ❌ Bad with inline object
const user = {
  id: '123',
  name: 'John Doe',
  email: 'john@example.com',
  role: 'user',
  createdAt: new Date('2024-01-01'),
  updatedAt: new Date('2024-01-01'),
  avatar: 'https://example.com/avatar.jpg',
  settings: {
    theme: 'dark',
    notifications: true
  }
};
```

### 3. Async Testing

#### Handle Promises Correctly
```typescript
// ✅ Good - Using await
it('fetches user data', async () => {
  const user = await userService.fetchUser('123');
  expect(user).toBeDefined();
});

// ✅ Good - Using returned promise
it('creates user', () => {
  return expect(userService.createUser(userData))
    .resolves.toMatchObject({ name: 'John Doe' });
});

// ❌ Bad - Not handling promise
it('fetches user data', () => {
  userService.fetchUser('123'); // Promise not handled
  expect(true).toBe(true); // Test passes even if promise rejects
});
```

### 4. Mocking Strategy

#### Mock External Dependencies
```typescript
// ✅ Good - Mock at the boundary
vi.mock('axios', () => ({
  default: {
    post: vi.fn()
  }
}));

// ❌ Bad - Mock internal implementation
vi.mock('./userService', () => ({
  userService: {
    createUser: vi.fn() // Testing implementation details
  }
}));
```

#### Use Partial Mocks When Needed
```typescript
// Keep original implementation, mock specific method
vi.spyOn(userService, 'validateEmail')
  .mockReturnValue(false);

await expect(userService.createUser(invalidUserData))
  .rejects.toThrow();

// Restore original
userService.validateEmail.mockRestore();
```

## Performance Testing

### 1. Load Testing
```typescript
// test/performance/api-load.test.ts
import { describe, it, expect } from 'vitest';
import { loadTest } from './utils/loadTest';

describe('API Load Tests', () => {
  it('handles concurrent chat requests', async () => {
    const results = await loadTest({
      url: '/api/v1/chat/completions',
      method: 'POST',
      body: { message: 'Hello' },
      concurrency: 100,
      duration: 10000 // 10 seconds
    });

    expect(results.successRate).toBeGreaterThan(0.95);
    expect(results.averageLatency).toBeLessThan(2000); // 2 seconds
    expect(results.errors).toHaveLength(0);
  });
});
```

### 2. Memory Testing
```typescript
// test/performance/memory.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { WorkflowEngine } from '@fastgpt/service/core/workflow/engine';

describe('Memory Usage', () => {
  let workflowEngine: WorkflowEngine;

  beforeEach(() => {
    workflowEngine = new WorkflowEngine();
  });

  it('does not leak memory during workflow execution', async () => {
    const initialMemory = process.memoryUsage().heapUsed;

    // Execute many workflows
    for (let i = 0; i < 1000; i++) {
      await workflowEngine.execute(testWorkflow);
    }

    // Force garbage collection if available
    if (global.gc) {
      global.gc();
    }

    const finalMemory = process.memoryUsage().heapUsed;
    const memoryIncrease = finalMemory - initialMemory;

    // Memory increase should be minimal (< 10MB)
    expect(memoryIncrease).toBeLessThan(10 * 1024 * 1024);
  });
});
```

## Debugging Tests

### 1. Test Debugging Tips

```typescript
// Use console.log for debugging
it('calculates complex value', () => {
  const input = { a: 1, b: 2, c: 3 };
  console.log('Input:', input);

  const result = calculate(input);
  console.log('Result:', result);

  expect(result).toBe(expected);
});

// Use test.only to run single test
it.only('this test only', () => {
  // This test will run alone
});

// Use test.skip to temporarily skip test
it.skip('not implemented yet', () => {
  // Test will be skipped
});
```

### 2. Visual Testing

```typescript
// test/unit/components/visual.test.tsx
import { test, expect } from '@playwright/experimental-ct-react';
import { UserCard } from '@fastgpt/web/components/UserCard';

test.use({ viewport: { width: 500, height: 500 } });

test('UserCard visual snapshot', async ({ mount }) => {
  const component = await mount(
    <UserCard user={{
      id: '1',
      name: 'John Doe',
      email: 'john@example.com'
    }} />
  );

  await expect(component).toHaveScreenshot('user-card.png');
});
```

This comprehensive testing strategy ensures that FastGPT maintains high code quality and reliability throughout development.