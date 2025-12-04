# Coding Standards and Best Practices

## Introduction

This document outlines the coding standards, patterns, and best practices for contributing to FastGPT. Following these guidelines ensures consistency, maintainability, and quality across the codebase.

## Language Policy

**IMPORTANT**: All code comments, documentation, variable names, and commit messages must be in English only. This is strictly enforced to maintain consistency for all developers working on the project.

## TypeScript Standards

### 1. Type Definitions

#### Use `type` for object types
```typescript
// ✅ Good
type User = {
  id: string;
  name: string;
  email: string;
};

// ❌ Avoid
interface User {
  id: string;
  name: string;
  email: string;
}
```

#### Use interfaces for class contracts and extendable types
```typescript
// ✅ Good
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  save(entity: T): Promise<T>;
}

// ✅ Good for class implementation
class MongoRepository<T> implements Repository<T> {
  // Implementation
}
```

#### Prefer union types over enums when possible
```typescript
// ✅ Good
type Status = 'pending' | 'processing' | 'completed' | 'error';

// ❌ Avoid
enum Status {
  Pending = 'pending',
  Processing = 'processing',
  Completed = 'completed',
  Error = 'error'
}
```

### 2. Generic Types

#### Use descriptive generic names
```typescript
// ✅ Good
interface Repository<TEntity> {
  find(id: string): Promise<TEntity | null>;
}

class Cache<TKey, TValue> {
  // Implementation
}

// ❌ Avoid
interface Repository<T> {
  find(id: string): Promise<T | null>;
}

class Cache<K, V> {
  // Implementation
}
```

#### Use generic constraints
```typescript
// ✅ Good
interface Identifiable {
  id: string;
}

function findById<T extends Identifiable>(
  collection: T[],
  id: string
): T | null {
  return collection.find(item => item.id === id) || null;
}
```

### 3. Utility Types

#### Leverage TypeScript utility types
```typescript
// ✅ Good
type UserWithoutPassword = Omit<User, 'password'>;
type PartialUser = Partial<User>;
type UserRoleKeys = keyof Pick<User, 'role' | 'permissions'>;

// ✅ Good for conditional types
type ApiResponse<T> = T extends string
  ? { message: T }
  : { data: T };
```

### 4. Function Types

#### Use explicit return types for public APIs
```typescript
// ✅ Good
export async function createUser(
  userData: CreateUserData
): Promise<User> {
  // Implementation
}

// ✅ Internal functions can be inferred
const hashPassword = async (password: string) => {
  // Implementation
};
```

## React/Component Standards

### 1. Component Structure

#### Use functional components with hooks
```typescript
// ✅ Good
const UserCard: React.FC<UserCardProps> = ({ user, onEdit }) => {
  const [isEditing, setIsEditing] = useState(false);
  const theme = useTheme();

  const handleEdit = useCallback(() => {
    setIsEditing(true);
    onEdit?.(user);
  }, [user, onEdit]);

  return (
    <Card>
      <Heading>{user.name}</Heading>
      <Button onClick={handleEdit}>Edit</Button>
    </Card>
  );
};

interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
}

// ❌ Avoid class components
class UserCard extends React.Component {
  // ...
}
```

#### Extract custom hooks for complex logic
```typescript
// ✅ Good
const useUserData = (userId: string) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [userId]);

  return { user, loading, error };
};

// Usage in component
const UserProfile = ({ userId }: { userId: string }) => {
  const { user, loading, error } = useUserData(userId);

  if (loading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <div>User not found</div>;

  return <UserDetails user={user} />;
};
```

### 2. Prop Types

#### Define interfaces for all component props
```typescript
// ✅ Good
interface DataTableProps<T> {
  data: T[];
  columns: ColumnDef<T>[];
  onRowClick?: (row: T) => void;
  loading?: boolean;
  emptyMessage?: string;
}

export const DataTable = <T,>({
  data,
  columns,
  onRowClick,
  loading,
  emptyMessage = 'No data available'
}: DataTableProps<T>) => {
  // Implementation
};
```

#### Use default props for optional values
```typescript
// ✅ Good
const Button: React.FC<ButtonProps> = ({
  variant = 'primary',
  size = 'md',
  disabled = false,
  children,
  ...props
}) => {
  return (
    <ChakraButton
      variant={variant}
      size={size}
      disabled={disabled}
      {...props}
    >
      {children}
    </ChakraButton>
  );
};

// ❌ Avoid defaultProps
Button.defaultProps = {
  variant: 'primary',
  size: 'md',
  disabled: false
};
```

### 3. State Management

#### Use Zustand for global state
```typescript
// ✅ Good
interface AppState {
  user: User | null;
  theme: 'light' | 'dark';
  setUser: (user: User | null) => void;
  toggleTheme: () => void;
}

export const useAppStore = create<AppState>((set) => ({
  user: null,
  theme: 'light',
  setUser: (user) => set({ user }),
  toggleTheme: () => set((state) => ({
    theme: state.theme === 'light' ? 'dark' : 'light'
  }))
}));

// Usage
const Header = () => {
  const { user, theme, toggleTheme } = useAppStore();

  return (
    <Box>
      <Text>{user?.name}</Text>
      <Switch isChecked={theme === 'dark'} onChange={toggleTheme} />
    </Box>
  );
};
```

#### Use TanStack Query for server state
```typescript
// ✅ Good
const useUsers = (filters: UserFilters) => {
  return useQuery({
    queryKey: ['users', filters],
    queryFn: () => userService.fetchUsers(filters),
    staleTime: 5 * 60 * 1000, // 5 minutes
    select: (data) => data.map(transformUser)
  });
};

const useCreateUser = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: userService.createUser,
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
      toast.success('User created successfully');
    },
    onError: (error) => {
      toast.error(`Failed to create user: ${error.message}`);
    }
  });
};
```

## API Design Standards

### 1. RESTful API Design

#### Use resource-based endpoints
```typescript
// ✅ Good
GET    /api/v1/users          // List users
GET    /api/v1/users/:id      // Get specific user
POST   /api/v1/users          // Create user
PUT    /api/v1/users/:id      // Update user
DELETE /api/v1/users/:id      // Delete user

// Nested resources
GET    /api/v1/users/:id/chats     // User's chats
POST   /api/v1/users/:id/chats     // Create chat for user

// ❌ Avoid
GET    /api/v1/getAllUsers
POST   /api/v1/userCreate
```

#### Use proper HTTP status codes
```typescript
// ✅ Good
if (!user) {
  return res.status(404).json({
    error: 'User not found',
    code: 'USER_NOT_FOUND'
  });
}

if (userData.email && !isValidEmail(userData.email)) {
  return res.status(400).json({
    error: 'Invalid email format',
    code: 'INVALID_EMAIL'
  });
}

return res.status(200).json({ data: user });
```

### 2. Response Format

#### Use consistent response structure
```typescript
// ✅ Success response
{
  "data": {
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "version": "1.0"
  }
}

// ✅ Error response
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": {
      "field": "email",
      "reason": "Invalid format"
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_123456"
  }
}
```

### 3. Input Validation

#### Validate all inputs
```typescript
// ✅ Good
import { z } from 'zod';

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  password: z.string().min(8).max(100),
  role: z.enum(['user', 'admin']).optional()
});

type CreateUserInput = z.infer<typeof createUserSchema>;

export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const userData = createUserSchema.parse(req.body);

    // Process valid data
    const user = await userService.createUser(userData);

    res.status(201).json({ data: user });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input',
          details: error.errors
        }
      });
    }

    // Handle other errors
    res.status(500).json({ error: 'Internal server error' });
  }
}
```

## Database Patterns

### 1. MongoDB Patterns

#### Use Mongoose with TypeScript
```typescript
// ✅ Good
import { Schema, model, Document } from 'mongoose';

interface IUser extends Document {
  name: string;
  email: string;
  passwordHash: string;
  createdAt: Date;
  updatedAt: Date;
}

const userSchema = new Schema<IUser>({
  name: { type: String, required: true, trim: true },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    trim: true
  },
  passwordHash: { type: String, required: true, select: false }
}, {
  timestamps: true,
  toJSON: {
    transform: (doc, ret) => {
      delete ret.passwordHash;
      return ret;
    }
  }
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ createdAt: -1 });

export const UserModel = model<IUser>('User', userSchema);
```

#### Use aggregation pipelines for complex queries
```typescript
// ✅ Good
const userStats = await UserModel.aggregate([
  { $match: { status: 'active' } },
  {
    $group: {
      _id: '$role',
      count: { $sum: 1 },
      avgAge: { $avg: '$age' }
    }
  },
  { $sort: { count: -1 } }
]);
```

### 2. Vector Database Patterns

#### Use the vector abstraction layer
```typescript
// ✅ Good
import { VectorStore } from '@fastgpt/service/common/vector';

const vectorStore = new VectorStore({
  provider: 'pg', // or 'milvus'
  config: {
    connectionString: process.env.PG_URI,
    dimensions: 1536
  }
});

// Insert with metadata
await vectorStore.insert('documents', [
  {
    id: 'doc1',
    content: 'Document content here',
    embedding: [0.1, 0.2, 0.3, ...],
    metadata: {
      source: 'pdf',
      author: 'John Doe',
      tags: ['important', 'reference']
    }
  }
]);

// Search with filters
const results = await vectorStore.search('documents', {
  vector: queryEmbedding,
  topK: 5,
  filters: {
    source: 'pdf',
    tags: { $in: ['important'] }
  }
});
```

## Error Handling

### 1. Custom Error Classes

```typescript
// ✅ Good
export class ValidationError extends Error {
  constructor(
    message: string,
    public field?: string,
    public code?: string
  ) {
    super(message);
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends Error {
  constructor(resource: string, id?: string) {
    super(`${resource}${id ? ` with id ${id}` : ''} not found`);
    this.name = 'NotFoundError';
  }
}

// Usage
if (!user) {
  throw new NotFoundError('User', userId);
}

if (!isValidEmail(email)) {
  throw new ValidationError('Invalid email format', 'email', 'INVALID_FORMAT');
}
```

### 2. Async Error Handling

```typescript
// ✅ Good with Result pattern
type Result<T, E = Error> = {
  success: true;
  data: T;
} | {
  success: false;
  error: E;
};

const fetchUser = async (id: string): Promise<Result<User>> => {
  try {
    const user = await UserModel.findById(id);

    if (!user) {
      return {
        success: false,
        error: new NotFoundError('User', id)
      };
    }

    return {
      success: true,
      data: user
    };
  } catch (error) {
    return {
      success: false,
      error: error as Error
    };
  }
};

// Usage
const result = await fetchUser(userId);
if (result.success) {
  // Use result.data
} else {
  // Handle result.error
}
```

## Testing Standards

### 1. Test Structure

```typescript
// ✅ Good test file
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { UserCard } from './UserCard';

// Test utilities
const createQueryClient = () => new QueryClient({
  defaultOptions: {
    queries: { retry: false },
    mutations: { retry: false }
  }
});

const renderWithQuery = (component: React.ReactElement) => {
  const queryClient = createQueryClient();
  return render(
    <QueryClientProvider client={queryClient}>
      {component}
    </QueryClientProvider>
  );
};

// Tests
describe('UserCard', () => {
  const mockUser = {
    id: '1',
    name: 'John Doe',
    email: 'john@example.com'
  };

  it('displays user information correctly', () => {
    renderWithQuery(<UserCard user={mockUser} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('john@example.com')).toBeInTheDocument();
  });

  it('calls onEdit when edit button is clicked', async () => {
    const onEdit = jest.fn();
    renderWithQuery(<UserCard user={mockUser} onEdit={onEdit} />);

    fireEvent.click(screen.getByRole('button', { name: /edit/i }));

    await waitFor(() => {
      expect(onEdit).toHaveBeenCalledWith(mockUser);
    });
  });

  it('handles missing user data gracefully', () => {
    renderWithQuery(<UserCard user={null as any} />);

    expect(screen.getByText('User not available')).toBeInTheDocument();
  });
});
```

### 2. Mock Management

```typescript
// ✅ Good mocking
import { jest } from '@jest/globals';

// Mock entire module
jest.mock('@fastgpt/service/common/mongo', () => ({
  UserModel: {
    findById: jest.fn(),
    save: jest.fn()
  }
}));

// Or mock individual functions
jest.mock('./userService', () => ({
  userService: {
    fetchUser: jest.fn(),
    createUser: jest.fn()
  }
}));

// Setup mocks before each test
beforeEach(() => {
  jest.clearAllMocks();
});

// Use mocks in tests
it('fetches and displays user', async () => {
  const mockUser = { id: '1', name: 'John' };
  (userService.fetchUser as jest.Mock).mockResolvedValue(mockUser);

  // Test code

  expect(userService.fetchUser).toHaveBeenCalledWith('1');
});
```

## Performance Guidelines

### 1. React Performance

#### Use React optimization patterns
```typescript
// ✅ Good with memo
const ExpensiveComponent = React.memo<ExpensiveComponentProps>(
  ({ data, onSelect }) => {
    const processedData = useMemo(() => {
      return data.map(item => ({
        ...item,
        computed: expensiveCalculation(item)
      }));
    }, [data]);

    return (
      <div>
        {processedData.map(item => (
          <Item key={item.id} item={item} onSelect={onSelect} />
        ))}
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Custom comparison
    return prevProps.data.length === nextProps.data.length;
  }
);
```

#### Lazy load components and routes
```typescript
// ✅ Good lazy loading
const ChatInterface = lazy(() => import('./ChatInterface'));
const WorkflowEditor = lazy(() => import('./WorkflowEditor'));

const App = () => {
  return (
    <Router>
      <Suspense fallback={<Spinner />}>
        <Routes>
          <Route path="/chat" element={<ChatInterface />} />
          <Route path="/workflow" element={<WorkflowEditor />} />
        </Routes>
      </Suspense>
    </Router>
  );
};
```

### 2. Backend Performance

#### Implement caching
```typescript
// ✅ Good with Redis cache
import { Redis } from 'ioredis';

class CacheService {
  private redis = new Redis(process.env.REDIS_URL);

  async get<T>(key: string): Promise<T | null> {
    const cached = await this.redis.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async set(key: string, value: any, ttl = 3600): Promise<void> {
    await this.redis.setex(key, ttl, JSON.stringify(value));
  }

  async invalidate(pattern: string): Promise<void> {
    const keys = await this.redis.keys(pattern);
    if (keys.length > 0) {
      await this.redis.del(...keys);
    }
  }
}

// Usage
const userService = {
  async getUser(id: string): Promise<User | null> {
    // Try cache first
    const cached = await cache.get<User>(`user:${id}`);
    if (cached) return cached;

    // Fetch from database
    const user = await UserModel.findById(id);

    // Cache result
    if (user) {
      await cache.set(`user:${id}`, user, 300); // 5 minutes
    }

    return user;
  }
};
```

## Security Guidelines

### 1. Input Sanitization

```typescript
// ✅ Good sanitization
import DOMPurify from 'dompurify';
import validator from 'validator';

const sanitizeHtml = (html: string): string => {
  return DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'strong', 'em', 'u'],
    ALLOWED_ATTR: []
  });
};

const sanitizeInput = (input: string): string => {
  return validator.escape(input.trim());
};
```

### 2. Password Handling

```typescript
// ✅ Good password handling
import bcrypt from 'bcrypt';
import crypto from 'crypto';

const hashPassword = async (password: string): Promise<string> => {
  const salt = await bcrypt.genSalt(12);
  return bcrypt.hash(password, salt);
};

const verifyPassword = async (
  password: string,
  hash: string
): Promise<boolean> => {
  return bcrypt.compare(password, hash);
};

// Reset tokens
const generateResetToken = (): string => {
  return crypto.randomBytes(32).toString('hex');
};
```

## Documentation Standards

### 1. Code Documentation

```typescript
// ✅ Good with JSDoc
/**
 * Fetches user data from the database with optional related information.
 *
 * @param userId - The unique identifier of the user
 * @param options - Configuration options for the fetch operation
 * @param options.includeChats - Whether to include user's chat history
 * @param options.includeTeams - Whether to include user's team memberships
 *
 * @returns Promise resolving to user object or null if not found
 *
 * @throws {ValidationError} When userId is invalid
 * @throws {DatabaseError} When database operation fails
 *
 * @example
 * ```typescript
 * const user = await fetchUser('123', { includeChats: true });
 * console.log(user.name);
 * ```
 */
export async function fetchUser(
  userId: string,
  options: FetchUserOptions = {}
): Promise<User | null> {
  // Implementation
}

interface FetchUserOptions {
  includeChats?: boolean;
  includeTeams?: boolean;
}
```

### 2. API Documentation

```typescript
// ✅ Good with OpenAPI annotations
/**
 * @swagger
 * /api/v1/users:
 *   get:
 *     summary: List all users
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           minimum: 1
 *           default: 1
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           minimum: 1
 *           maximum: 100
 *           default: 10
 *     responses:
 *       200:
 *         description: List of users
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 meta:
 *                   $ref: '#/components/schemas/PaginationMeta'
 */
export default async function handler(
  req: NextApiRequest,
  res: NextApiResponse
) {
  // Implementation
}
```

## Git Workflow

### 1. Commit Messages

Follow conventional commits format:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Types:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation
- `style`: Formatting, missing semicolons
- `refactor`: Refactoring code
- `test`: Adding tests
- `chore`: Updating build tasks, package manager configs

Examples:
```
feat(auth): add two-factor authentication

- Implement TOTP generation
- Add QR code display
- Update login flow

Closes #123
```

### 2. Branch Strategy

```
main                    // Production code
├── develop            // Integration branch
├── feature/xyz        // Feature development
├── bugfix/xyz         // Bug fixes
└── hotfix/xyz         // Production hotfixes
```

## Code Review Checklist

### Before Submitting PR:
- [ ] All tests pass
- [ ] Code follows TypeScript standards
- [ ] Components are properly typed
- [ ] Error handling is implemented
- [ ] Security considerations are addressed
- [ ] Performance implications are considered
- [ ] Documentation is updated
- [ ] Commit messages follow conventions

### During Review:
- [ ] Code is readable and maintainable
- [ ] No sensitive data is exposed
- [ ] Proper error handling exists
- [ ] Tests cover edge cases
- [ ] No unnecessary dependencies added
- [ ] Code is DRY (Don't Repeat Yourself)
- [ ] Appropriate abstractions are used

Following these coding standards ensures that FastGPT remains a high-quality, maintainable, and scalable codebase that all developers can work with effectively.