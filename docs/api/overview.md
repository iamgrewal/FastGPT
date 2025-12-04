# API Overview

## Introduction

FastGPT provides a comprehensive RESTful API that allows developers to integrate AI capabilities into their applications. This document provides an overview of the API architecture, design principles, and available endpoints.

## API Architecture

### Base URL
- **Development**: `http://localhost:3000/api/v1`
- **Production**: `https://your-domain.com/api/v1`

### Versioning
The API is versioned to ensure backward compatibility:
- **v1**: Current stable version
- **v2**: Beta features (when available)

### Architecture Pattern

```
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  AuthN/AuthZ│  │ Rate Limit  │  │  Request Validation │ │
│  │             │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                    API Routes                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Apps      │  │  Workflows  │  │  Knowledge Base     │ │
│  │             │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                 Service Layer                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ AI Services │  │   Database  │  │  External APIs      │ │
│  │             │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Design Principles

### 1. RESTful Design
- **Resource-based URLs**: Nouns, not verbs
- **HTTP Methods**: Proper use of GET, POST, PUT, DELETE
- **Status Codes**: Meaningful HTTP status codes
- **Stateless**: Each request contains all necessary information

### 2. Consistent Response Format
```json
{
  "success": true,
  "data": {
    // Response data
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "version": "1.0",
    "requestId": "req_123456"
  }
}
```

### 3. Error Handling
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ]
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_123456"
  }
}
```

## Authentication

### API Key Authentication
```http
GET /api/v1/apps
Authorization: Bearer YOUR_API_KEY
```

### JWT Token Authentication
```http
GET /api/v1/user/profile
Authorization: Bearer YOUR_JWT_TOKEN
```

### Authentication Types
1. **App API Keys**: For application-level access
2. **User Tokens**: For user-specific operations
3. **Service Tokens**: For server-to-server communication

## Rate Limiting

### Default Limits
- **Free Tier**: 100 requests/minute
- **Pro Tier**: 1000 requests/minute
- **Enterprise**: Custom limits

### Rate Limit Headers
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1642694400
```

## Available APIs

### 1. Authentication API
Manage user authentication and authorization

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/login` | POST | User login |
| `/auth/register` | POST | User registration |
| `/auth/refresh` | POST | Refresh access token |
| `/auth/logout` | POST | User logout |
| `/auth/forgot-password` | POST | Reset password |
| `/auth/verify-email` | POST | Verify email address |

### 2. Application API
Manage FastGPT applications

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/apps` | GET | List applications |
| `/apps` | POST | Create application |
| `/apps/:id` | GET | Get application details |
| `/apps/:id` | PUT | Update application |
| `/apps/:id` | DELETE | Delete application |
| `/apps/:id/clone` | POST | Clone application |
| `/apps/:id/export` | GET | Export application |

### 3. Workflow API
Manage and execute workflows

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/workflows` | GET | List workflows |
| `/workflows` | POST | Create workflow |
| `/workflows/:id` | GET | Get workflow details |
| `/workflows/:id` | PUT | Update workflow |
| `/workflows/:id` | DELETE | Delete workflow |
| `/workflows/:id/run` | POST | Execute workflow |
| `/workflows/:id/status` | GET | Get execution status |

### 4. Knowledge Base API
Manage knowledge bases and datasets

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/datasets` | GET | List datasets |
| `/datasets` | POST | Create dataset |
| `/datasets/:id` | GET | Get dataset details |
| `/datasets/:id` | PUT | Update dataset |
| `/datasets/:id` | DELETE | Delete dataset |
| `/datasets/:id/upload` | POST | Upload documents |
| `/datasets/:id/search` | POST | Search in dataset |
| `/datasets/:id/embeddings` | GET | Get embedding status |

### 5. Chat API
Real-time chat interactions

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/chat/completions` | POST | Send chat message |
| `/chat/history` | GET | Get chat history |
| `/chat/history/:id` | DELETE | Delete chat |
| `/chat/feedback` | POST | Submit feedback |
| `/chat/stream` | WebSocket | Stream responses |

### 6. Plugin API
Manage plugins and extensions

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/plugins` | GET | List available plugins |
| `/plugins/:id/config` | GET | Get plugin config |
| `/plugins/:id/execute` | POST | Execute plugin |
| `/plugins/:id/status` | GET | Get plugin status |

### 7. User API
User management and settings

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/user/profile` | GET | Get user profile |
| `/user/profile` | PUT | Update profile |
| `/user/settings` | GET | Get user settings |
| `/user/settings` | PUT | Update settings |
| `/user/api-keys` | GET | List API keys |
| `/user/api-keys` | POST | Create API key |
| `/user/api-keys/:id` | DELETE | Delete API key |

### 8. Analytics API
Usage analytics and metrics

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/analytics/usage` | GET | Get usage statistics |
| `/analytics/performance` | GET | Get performance metrics |
| `/analytics/costs` | GET | Get cost breakdown |
| `/analytics/export` | POST | Export analytics data |

## Request/Response Examples

### 1. Creating an Application

**Request**:
```http
POST /api/v1/apps
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "name": "Customer Support Bot",
  "description": "AI-powered customer service assistant",
  "type": "workflow",
  "config": {
    "model": "gpt-3.5-turbo",
    "temperature": 0.7,
    "maxTokens": 2000
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "id": "app_123456",
    "name": "Customer Support Bot",
    "description": "AI-powered customer service assistant",
    "type": "workflow",
    "status": "draft",
    "createdAt": "2024-01-15T10:30:00Z",
    "config": {
      "model": "gpt-3.5-turbo",
      "temperature": 0.7,
      "maxTokens": 2000
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_123456"
  }
}
```

### 2. Executing a Workflow

**Request**:
```http
POST /api/v1/workflows/wf_789/run
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "inputs": {
    "message": "Hello, I need help with my order",
    "userId": "user_456"
  },
  "options": {
    "stream": false,
    "timeout": 30000
  }
}
```

**Response**:
```json
{
  "success": true,
  "data": {
    "runId": "run_456789",
    "status": "completed",
    "outputs": {
      "response": "I'd be happy to help you with your order! Can you please provide your order number?",
      "confidence": 0.95,
      "actions": []
    },
    "executionTime": 2500,
    "tokenUsage": {
      "prompt": 45,
      "completion": 23,
      "total": 68
    }
  },
  "meta": {
    "timestamp": "2024-01-15T10:31:00Z",
    "requestId": "req_123457"
  }
}
```

### 3. Streaming Chat Completion

**Request**:
```http
POST /api/v1/chat/completions
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "appId": "app_123456",
  "message": "What are your business hours?",
  "stream": true
}
```

**Response** (Server-Sent Events):
```http
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

data: {"type": "start", "runId": "run_456789"}

data: {"type": "token", "content": "Our"}

data: {"type": "token", "content": " business"}

data: {"type": "token", "content": " hours"}

data: {"type": "token", "content": " are"}

data: {"type": "token", "content": " 9 AM"}

data: {"type": "token", "content": " to"}

data: {"type": "token", "content": " 6 PM"}

data: {"type": "token", "content": " Monday"}

data: {"type": "token", "content": " through"}

data: {"type": "token", "content": " Friday"}

data: {"type": "end", "runId": "run_456789", "tokenUsage": {"total": 48}}
```

## Pagination

For list endpoints, the API supports cursor-based pagination:

**Request**:
```http
GET /api/v1/apps?limit=20&cursor=next_page_token
Authorization: Bearer YOUR_API_KEY
```

**Response**:
```json
{
  "success": true,
  "data": [
    // Array of items
  ],
  "pagination": {
    "hasMore": true,
    "nextCursor": "next_page_token",
    "limit": 20
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "req_123456"
  }
}
```

## Filtering and Sorting

Most list endpoints support filtering and sorting:

**Request**:
```http
GET /api/v1/apps?status=published&sortBy=createdAt&sortOrder=desc&search=chatbot
Authorization: Bearer YOUR_API_KEY
```

## Webhooks

FastGPT supports webhooks for real-time notifications:

### Webhook Events
- `workflow.completed`: Workflow execution finished
- `workflow.failed`: Workflow execution failed
- `dataset.completed`: Dataset processing completed
- `user.created`: New user registered

### Webhook Configuration

**Request**:
```http
POST /api/v1/webhooks
Authorization: Bearer YOUR_API_KEY
Content-Type: application/json

{
  "url": "https://your-app.com/webhook",
  "events": ["workflow.completed", "workflow.failed"],
  "secret": "webhook_secret"
}
```

**Webhook Payload**:
```json
{
  "event": "workflow.completed",
  "data": {
    "runId": "run_456789",
    "workflowId": "wf_123",
    "status": "completed",
    "outputs": {
      "response": "Success message"
    }
  },
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## SDKs and Client Libraries

### Official SDKs

#### JavaScript/TypeScript
```bash
npm install @fastgpt/sdk
```

```typescript
import { FastGPTClient } from '@fastgpt/sdk';

const client = new FastGPTClient({
  apiKey: 'YOUR_API_KEY',
  baseURL: 'https://api.fastgpt.io'
});

// Create an application
const app = await client.apps.create({
  name: 'My App',
  type: 'workflow'
});

// Execute a workflow
const result = await client.workflows.run('wf_123', {
  inputs: { message: 'Hello' }
});
```

#### Python
```bash
pip install fastgpt-sdk
```

```python
from fastgpt import FastGPTClient

client = FastGPTClient(
    api_key='YOUR_API_KEY',
    base_url='https://api.fastgpt.io'
)

# Create an application
app = client.apps.create({
    'name': 'My App',
    'type': 'workflow'
})

# Execute a workflow
result = client.workflows.run('wf_123', {
    'inputs': {'message': 'Hello'}
})
```

### Third-party Integrations

#### Postman Collection
Import the FastGPT API collection to test endpoints in Postman.

#### OpenAPI Specification
The API is fully documented with OpenAPI 3.0 specifications available at:
`/api/v1/docs` or `/api/v1/openapi.json`

## Error Codes

| Code | Description |
|------|-------------|
| `INVALID_REQUEST` | Malformed request |
| `UNAUTHORIZED` | Authentication failed |
| `FORBIDDEN` | Permission denied |
| `NOT_FOUND` | Resource not found |
| `VALIDATION_ERROR` | Input validation failed |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `QUOTA_EXCEEDED` | Usage quota exceeded |
| `WORKFLOW_ERROR` | Workflow execution failed |
| `AI_SERVICE_ERROR` | AI service unavailable |
| `INTERNAL_ERROR` | Server error |

## Best Practices

### 1. Security
- Use HTTPS for all API calls
- Never expose API keys in client-side code
- Validate webhook signatures
- Implement proper error handling

### 2. Performance
- Use appropriate pagination
- Cache responses when possible
- Use streaming for large responses
- Implement retry logic with exponential backoff

### 3. Monitoring
- Monitor rate limit headers
- Track API usage and costs
- Set up alerts for errors
- Log request IDs for debugging

### 4. Rate Limiting
- Respect rate limits
- Implement client-side throttling
- Use batch operations when possible
- Cache frequently accessed data

## API Roadmap

### Upcoming Features
- GraphQL API for flexible queries
- Real-time WebSocket API
- Advanced analytics endpoints
- Multi-tenant management APIs
- Plugin marketplace APIs

### Version Updates
- v1.1: Enhanced webhook capabilities
- v1.2: Advanced filtering options
- v2.0: GraphQL support (breaking changes)

For detailed endpoint documentation, refer to the specific API guides.