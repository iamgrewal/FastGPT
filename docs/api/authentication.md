# Authentication and Authorization API

## Overview

FastGPT provides multiple authentication mechanisms to secure API access and user sessions. This guide covers all authentication methods, token management, and authorization patterns.

## Authentication Methods

### 1. API Key Authentication

For server-to-server communication and application-level access.

#### Generating API Keys

**Request**:
```http
POST /api/v1/user/api-keys
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "name": "Production API Key",
  "permissions": [
    "app:read",
    "workflow:execute",
    "dataset:read"
  ],
  "expiresAt": "2025-01-15T00:00:00Z"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "key_123456",
    "name": "Production API Key",
    "key": "fastgpt_live_51e2c8b9...4d7f",
    "prefix": "fastgpt_live_",
    "permissions": [
      "app:read",
      "workflow:execute",
      "dataset:read"
    ],
    "expiresAt": "2025-01-15T00:00:00Z",
    "lastUsed": null,
    "createdAt": "2024-01-15T10:30:00Z"
  }
}
```

#### Using API Keys

```http
GET /api/v1/apps
Authorization: Bearer fastgpt_live_51e2c8b9...4d7f
```

### 2. JWT Token Authentication

For user sessions and interactive applications.

#### Login

**Request**:
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password",
  "rememberMe": true
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "user_123456",
      "email": "user@example.com",
      "name": "John Doe",
      "role": "user"
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    }
  }
}
```

#### Refreshing Tokens

**Request**:
```http
POST /api/v1/auth/refresh
Content-Type: application/json

{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": 3600
  }
}
```

### 3. OAuth 2.0 Integration

For third-party authentication providers.

#### Authorization Code Flow

1. **Authorization Request**:
```http
GET /api/v1/auth/oauth/authorize?
  response_type=code&
  client_id=your_client_id&
  redirect_uri=https://your-app.com/callback&
  scope=app:read workflow:execute&
  state=random_string
```

2. **Token Exchange**:
```http
POST /api/v1/auth/oauth/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=authorization_code&
client_id=your_client_id&
client_secret=your_client_secret&
redirect_uri=https://your-app.com/callback
```

## Authorization Model

### 1. Role-Based Access Control (RBAC)

#### User Roles

| Role | Description | Permissions |
|------|-------------|-------------|
| `admin` | System administrator | All permissions |
| `owner` | Team owner | Team management |
| `developer` | App developer | App management |
| `user` | Regular user | Basic usage |
| `viewer` | Read-only access | View only |

#### Permission Structure

```typescript
interface Permission {
  resource: string;     // Resource type (app, workflow, dataset)
  action: string;       // Action (read, write, delete, execute)
  scope: string;        // Scope (own, team, all)
  conditions?: Record<string, any>; // Additional conditions
}

// Example permissions
const permissions = [
  { resource: 'app', action: 'read', scope: 'team' },
  { resource: 'app', action: 'write', scope: 'own' },
  { resource: 'workflow', action: 'execute', scope: 'own' }
];
```

### 2. Resource-Based Permissions

#### Application Permissions
- `app:create`: Create new applications
- `app:read`: View application details
- `app:write`: Modify applications
- `app:delete`: Delete applications
- `app:execute`: Run applications

#### Workflow Permissions
- `workflow:create`: Create workflows
- `workflow:read`: View workflows
- `workflow:write`: Modify workflows
- `workflow:delete`: Delete workflows
- `workflow:execute`: Execute workflows

#### Dataset Permissions
- `dataset:create`: Create datasets
- `dataset:read`: View datasets
- `dataset:write`: Modify datasets
- `dataset:delete`: Delete datasets
- `dataset:search`: Search in datasets

## Team Management

### 1. Creating Teams

**Request**:
```http
POST /api/v1/teams
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "name": "AI Research Team",
  "description": "Team for AI research projects",
  "settings": {
    "defaultRole": "developer",
    "allowInvites": true
  }
}
```

### 2. Team Membership

**Inviting Members**:
```http
POST /api/v1/teams/:teamId/members
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "email": "newmember@example.com",
  "role": "developer",
  "permissions": [
    "app:read",
    "workflow:execute"
  ]
}
```

**Managing Roles**:
```http
PUT /api/v1/teams/:teamId/members/:memberId
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "role": "owner",
  "permissions": [
    "app:read",
    "app:write",
    "workflow:read",
    "workflow:write",
    "workflow:execute"
  ]
}
```

## API Key Management

### 1. Key Types

#### Live Keys
- Prefix: `fastgpt_live_`
- Used for production
- Full permissions based on scope

#### Test Keys
- Prefix: `fastgpt_test_`
- Used for development/testing
- Limited permissions

### 2. Key Restrictions

#### IP Whitelisting
```json
{
  "id": "key_123456",
  "restrictions": {
    "ipWhitelist": [
      "192.168.1.100",
      "10.0.0.50",
      "203.0.113.0/24"
    ]
  }
}
```

#### Domain Restrictions
```json
{
  "id": "key_123456",
  "restrictions": {
    "allowedDomains": [
      "yourdomain.com",
      "sub.yourdomain.com"
    ]
  }
}
```

#### Rate Limiting
```json
{
  "id": "key_123456",
  "restrictions": {
    "rateLimit": {
      "requestsPerMinute": 1000,
      "requestsPerHour": 50000,
      "requestsPerDay": 1000000
    }
  }
}
```

### 3. Revoking Keys

**Request**:
```http
DELETE /api/v1/user/api-keys/:keyId
Authorization: Bearer YOUR_JWT_TOKEN
```

## Session Management

### 1. Session Configuration

#### Token Expiration
```json
{
  "auth": {
    "accessToken": {
      "expiresIn": 3600,        // 1 hour
      "refreshable": true
    },
    "refreshToken": {
      "expiresIn": 86400 * 30,  // 30 days
      "rotatable": true
    },
    "session": {
      "maxAge": 86400 * 7,     // 7 days
      "revokeOnLogout": true
    }
  }
}
```

### 2. Active Sessions

**Get Active Sessions**:
```http
GET /api/v1/user/sessions
Authorization: Bearer YOUR_JWT_TOKEN
```

**Response**:
```json
{
  "success": true,
  "data": {
    "sessions": [
      {
        "id": "sess_123456",
        "device": "Chrome on macOS",
        "ip": "203.0.113.1",
        "location": "San Francisco, CA",
        "lastActive": "2024-01-15T10:30:00Z",
        "createdAt": "2024-01-15T09:00:00Z"
      }
    ]
  }
}
```

**Revoke Session**:
```http
DELETE /api/v1/user/sessions/:sessionId
Authorization: Bearer YOUR_JWT_TOKEN
```

## Security Best Practices

### 1. API Key Security

#### Environment Variables
```bash
# .env
FASTGPT_API_KEY=fastgpt_live_51e2c8b9...4d7f
FASTGPT_SECRET_KEY=your_secret_key_here
```

#### Key Rotation
```javascript
// Implement key rotation
const rotateApiKey = async (keyId) => {
  // Create new key
  const newKey = await createApiKey({
    name: "Rotated Key",
    permissions: oldKey.permissions
  });

  // Update services
  await updateServicesWithNewKey(newKey.key);

  // Deactivate old key
  await deactivateApiKey(keyId);

  return newKey;
};
```

### 2. JWT Security

#### Token Validation
```typescript
// Middleware for JWT validation
const validateJWT = async (req, res, next) => {
  const token = req.headers.authorization?.replace('Bearer ', '');

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const decoded = jwt.verify(token, process.env.SECRET_KEY);
    const user = await User.findById(decoded.userId);

    if (!user) {
      return res.status(401).json({ error: 'Invalid token' });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};
```

#### Secure Storage
```typescript
// Store tokens securely
const tokenStorage = {
  // Use HttpOnly cookies for web
  setCookie: (res, token) => {
    res.cookie('token', token, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 3600000 // 1 hour
    });
  },

  // Use secure storage for mobile
  setSecureStorage: (token) => {
    if (typeof window !== 'undefined' && window.secureStorage) {
      window.secureStorage.setItem('token', token);
    }
  }
};
```

### 3. CSRF Protection

#### CSRF Tokens
```typescript
// CSRF middleware
const csrfProtection = (req, res, next) => {
  if (req.method === 'GET') {
    // Generate CSRF token for GET requests
    const token = generateCSRFToken();
    res.cookie('csrf-token', token, {
      httpOnly: false,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict'
    });
    req.csrfToken = token;
  } else {
    // Validate CSRF token for POST/PUT/DELETE
    const token = req.headers['x-csrf-token'] || req.body._csrf;
    const cookieToken = req.cookies['csrf-token'];

    if (!token || token !== cookieToken) {
      return res.status(403).json({ error: 'Invalid CSRF token' });
    }
  }
  next();
};
```

## Multi-Factor Authentication (MFA)

### 1. Enabling MFA

**Setup MFA**:
```http
POST /api/v1/user/mfa/setup
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "type": "totp"
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qrCode": "data:image/png;base64,iVBORw0KGgo...",
    "backupCodes": [
      "12345678",
      "87654321",
      "11223344",
      "44332211"
    ]
  }
}
```

### 2. MFA Verification

**Verify MFA**:
```http
POST /api/v1/user/mfa/verify
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "code": "123456"
}
```

### 3. MFA During Login

**Login with MFA**:
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "secure_password",
  "mfaCode": "123456"
}
```

## OAuth 2.0 Provider Setup

### 1. Register Application

**Create OAuth App**:
```http
POST /api/v1/oauth/apps
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "name": "My Integration",
  "description": "Integration with FastGPT",
  "redirectUris": [
    "https://myapp.com/callback",
    "https://myapp.com/oauth/callback"
  ],
  "scopes": [
    "app:read",
    "workflow:execute"
  ],
  "grantTypes": [
    "authorization_code",
    "refresh_token"
  ]
}
```

### 2. OAuth Scopes

| Scope | Description | Access |
|-------|-------------|--------|
| `app:read` | Read applications | Basic |
| `app:write` | Create/modify apps | Write |
| `workflow:read` | Read workflows | Basic |
| `workflow:execute` | Execute workflows | Execute |
| `dataset:read` | Read datasets | Basic |
| `dataset:write` | Modify datasets | Write |
| `user:read` | Read user profile | Profile |
| `admin:*` | Administrative access | Admin |

## Webhook Authentication

### 1. Webhook Secrets

**Configure Webhook**:
```http
POST /api/v1/webhooks
Authorization: Bearer YOUR_JWT_TOKEN
Content-Type: application/json

{
  "url": "https://myapp.com/webhook",
  "secret": "whsec_1234567890abcdef",
  "events": ["workflow.completed", "dataset.processed"]
}
```

### 2. Signature Verification

```typescript
// Verify webhook signature
import crypto from 'crypto';

const verifyWebhookSignature = (
  payload: string,
  signature: string,
  secret: string
): boolean => {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');

  const receivedSignature = signature.replace('sha256=', '');

  return crypto.timingSafeEqual(
    Buffer.from(expectedSignature),
    Buffer.from(receivedSignature)
  );
};
```

## Error Handling

### Authentication Errors

| Error | Code | Description |
|-------|------|-------------|
| Invalid credentials | `AUTH_INVALID_CREDENTIALS` | Wrong email/password |
| Account locked | `AUTH_ACCOUNT_LOCKED` | Too many failed attempts |
| Token expired | `AUTH_TOKEN_EXPIRED` | JWT token has expired |
| Invalid token | `AUTH_INVALID_TOKEN` | Token is malformed/invalid |
| Insufficient permissions | `AUTH_INSUFFICIENT_PERMISSIONS` | User lacks required permission |

### Authorization Errors

| Error | Code | Description |
|-------|------|-------------|
| Resource not found | `RESOURCE_NOT_FOUND` | User doesn't have access |
| Access denied | `ACCESS_DENIED` | Permission denied |
| Rate limit exceeded | `RATE_LIMIT_EXCEEDED` | Too many requests |
| IP not allowed | `IP_NOT_ALLOWED` | IP not in whitelist |
| Invalid API key | `INVALID_API_KEY` | API key is invalid/expired |

## SDK Examples

### JavaScript/TypeScript

```typescript
import { FastGPTClient } from '@fastgpt/sdk';

const client = new FastGPTClient({
  apiKey: 'fastgpt_live_51e2c8b9...4d7f',
  baseURL: 'https://api.fastgpt.io'
});

// The SDK automatically handles authentication
const apps = await client.apps.list();
```

### Python

```python
from fastgpt import FastGPTClient

client = FastGPTClient(
    api_key='fastgpt_live_51e2c8b9...4d7f',
    base_url='https://api.fastgpt.io'
)

# The SDK automatically handles authentication
apps = client.apps.list()
```

This authentication guide provides comprehensive information about securing FastGPT APIs and managing user access.