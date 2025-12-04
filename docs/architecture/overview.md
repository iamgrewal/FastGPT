# System Architecture Overview

## Introduction

FastGPT is built on a sophisticated layered architecture that provides clear separation of concerns, enabling scalability, maintainability, and extensibility. This document provides a high-level view of the system architecture, its components, and how they interact to deliver a comprehensive AI agent building platform.

## High-Level Architecture

The FastGPT architecture follows a four-tier pattern:

```
┌─────────────────────────────────────────────────────────────┐
│                    Frontend Layer                           │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Web App       │  │   Marketplace   │  │  Documentation│ │
│  │  (NextJS)       │  │   (NextJS)      │  │     Site      │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                    API Gateway Layer                        │
│              (NextJS API Routes + Express)                  │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                   Service Layer                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   AI Services   │  │  Workflow Engine│  │  Plugin Mgmt │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│                   Data Layer                                │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │    MongoDB      │  │  PostgreSQL     │  │    Milvus    │ │
│  │  (Core Data)    │  │  (Vector DB)    │  │ (Vector DB)  │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

#### 1. Frontend Layer
- **Web Application** (`projects/app/`): Main user interface for building and managing AI workflows
- **Marketplace** (`projects/marketplace/`): Template sharing and discovery platform
- **Documentation Site** (`document/`): NextJS-based documentation website

#### 2. API Gateway Layer
- Routes incoming requests to appropriate services
- Handles authentication and authorization
- Provides RESTful API endpoints
- Manages WebSocket connections for real-time features

#### 3. Service Layer
- **AI Services**: Unified interface for multiple AI providers
- **Workflow Engine**: Visual workflow orchestration and execution
- **Plugin Management**: Dynamic loading and execution of plugins
- **Code Sandbox**: Isolated code execution environment

#### 4. Data Layer
- **MongoDB**: Core application data (users, workflows, configurations)
- **PostgreSQL**: Relational data with vector support (optional)
- **Milvus**: High-performance vector database (optional)

## Core Architectural Principles

### 1. Modularity and Separation of Concerns
Each component has a single responsibility:
- Frontend focuses on user experience
- API layer handles request routing and auth
- Services contain business logic
- Data layer abstracts storage concerns

### 2. Extensibility through Plugins
The plugin architecture allows for:
- Easy addition of new AI providers
- Custom processing modules
- Third-party integrations
- Independent scaling of features

### 3. Unified Abstractions
- Single interface for multiple AI providers
- Common workflow execution model
- Consistent data access patterns
- Standardized plugin interface

### 4. Scalability Design
- Stateless services for horizontal scaling
- Microservice-style plugin architecture
- Asynchronous processing where appropriate
- Caching at multiple levels

## Component Deep Dive

### Frontend Architecture

```
Frontend (NextJS)
├── Pages/
│   ├── API Routes (Backend endpoints)
│   └── Static Pages (SSG/SSR)
├── Components/
│   ├── Workflow Designer
│   ├── Chat Interface
│   └── Knowledge Base Manager
├── State Management
│   ├── Zustand (Client state)
│   └── TanStack Query (Server state)
└── Styling
    └── Chakra UI + Custom Theme
```

### Backend Architecture

```
Backend Services
├── API Gateway
│   ├── Authentication
│   ├── Rate Limiting
│   └── Request Routing
├── Core Services
│   ├── AI Service Layer
│   ├── Workflow Engine
│   ├── Knowledge Base
│   └── Plugin Manager
├── Supporting Services
│   ├── Code Sandbox (NestJS)
│   ├── MCP Server
│   └── Background Jobs
└── Data Access
    ├── MongoDB Models
    ├── Vector DB Abstraction
    └── Cache Layer (Redis)
```

## Data Flow Architecture

### Workflow Execution Flow

```
User Interaction
       │
┌──────▼──────┐
│   API       │
│  Gateway    │
└──────┬──────┘
       │
┌──────▼──────┐
│   Workflow  │
│   Engine    │
└──────┬──────┘
       │
┌──────▼──────┐    ┌──────────────┐
│  AI Service │◄───│  AI Providers│
│  Layer      │    │ (Multiple)   │
└──────┬──────┘    └──────────────┘
       │
┌──────▼──────┐    ┌──────────────┐
│   Data      │◄───│  Vector DB   │
│   Layer     │    │  Search      │
└──────┬──────┘    └──────────────┘
       │
┌──────▼──────┐
│  Response   │
│  Assembly   │
└──────┬──────┘
       │
┌──────▼──────┐
│   User      │
│  Interface │
└────────────┘
```

### Knowledge Base Processing Flow

```
Document Upload
       │
┌──────▼──────┐
│   File      │
│ Processing │
└──────┬──────┘
       │
┌──────▼──────┐
│   Text      │
│ Extraction │
└──────┬──────┘
       │
┌──────▼──────┐
│   Chunking  │
│   Strategy  │
└──────┬──────┘
       │
┌──────▼──────┐    ┌──────────────┐
│  Embedding  │◄───│  AI Service  │
│ Generation │    │   Layer      │
└──────┬──────┘    └──────────────┘
       │
┌──────▼──────┐
│  Vector     │
│  Storage    │
└──────┬──────┘
       │
┌──────▼──────┐
│   Indexing  │
│  & Search   │
└────────────┘
```

## Integration Points

### External Integrations

1. **AI Providers**
   - OpenAI API
   - Anthropic Claude API
   - Local model deployments
   - Custom model endpoints

2. **Vector Databases**
   - PostgreSQL with pgvector
   - Milvus
   - Custom vector stores

3. **Authentication Providers**
   - JWT-based auth
   - OAuth 2.0
   - SSO integration

4. **Storage Services**
   - Local file system
   - AWS S3
   - MinIO

### Internal Service Communication

```
┌─────────────┐    HTTP API    ┌─────────────┐
│   Frontend  │◄──────────────►│ API Gateway │
└─────────────┘               └──────┬──────┘
                                     │
                              ┌──────▼──────┐
                              │  Core       │
                              │  Services   │
                              └──────┬──────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                │                │
          ┌─────────▼─────────┐     │     ┌──────────▼─────────┐
          │    Code Sandbox   │     │     │   Plugin Manager   │
          │   (NestJS App)    │     │     │ (Docker Containers)│
          └───────────────────┘     │     └────────────────────┘
                                   │
                          ┌────────▼─────────┐
                          │   Data Layer     │
                          │ (MongoDB + Vector│
                          │     Databases)   │
                          └──────────────────┘
```

## Security Architecture

### Security Boundaries

```
┌─────────────────────────────────────────┐
│           Internet                      │
└────────────────┬────────────────────────┘
                 │
        ┌────────▼────────┐
        │  Load Balancer  │
        │   + CDN         │
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │  API Gateway    │
        │ (Auth + Rate    │
        │   Limiting)     │
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │  Service Layer  │
        │ (Business Logic)│
        └────────┬────────┘
                 │
        ┌────────▼────────┐
        │   Data Layer    │
        │ (Encrypted DB)  │
        └─────────────────┘
```

## Performance Considerations

### Optimization Strategies

1. **Frontend**
   - Code splitting and lazy loading
   - Image optimization
   - Bundle size optimization
   - Caching strategies

2. **Backend**
   - Database indexing
   - Connection pooling
   - Redis caching
   - Batch processing

3. **AI Services**
   - Model caching
   - Request batching
   - Response streaming
   - Fallback providers

### Scalability Patterns

```
┌────────────────┐    ┌────────────────┐    ┌────────────────┐
│   Frontend     │    │   Frontend     │    │   Frontend     │
│  (CDN Edge)    │    │  (CDN Edge)    │    │  (CDN Edge)    │
└──────┬─────────┘    └──────┬─────────┘    └──────┬─────────┘
       │                    │                    │
       └────────────────────┼────────────────────┘
                            │
                    ┌───────▼───────┐
                    │  Load Balancer│
                    └───────┬───────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
  ┌─────▼─────┐       ┌─────▼─────┐       ┌─────▼─────┐
  │  Service  │       │  Service  │       │  Service  │
  │ Instance │       │ Instance │       │ Instance │
  └─────┬─────┘       └─────┬─────┘       └─────┬─────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────▼───────┐
                    │   Database    │
                    │  Cluster      │
                    └───────────────┘
```

## Technology Stack Summary

| Layer | Technology | Purpose |
|-------|------------|---------|
| **Frontend** | NextJS 14.2.32 | React framework with SSR/SSG |
| | React 18.3.1 | UI library |
| | TypeScript 5.1.3 | Static typing |
| | Chakra UI 2.10.7 | Component library |
| | Zustand | State management |
| | TanStack Query | Server state |
| **Backend** | Node.js 20+ | Runtime |
| | Express | HTTP server |
| | NestJS 10.4.16 | Sandbox service |
| | MongoDB | Document database |
| | PostgreSQL | Relational + vector DB |
| | Milvus | Vector database |
| **AI/ML** | Multiple Providers | LLM integration |
| | OpenAI APIs | Embeddings |
| | Custom models | Specialized tasks |
| **Infrastructure** | Docker | Containerization |
| | Kubernetes | Orchestration |
| | Redis | Caching |
| | Helm | Deployment |

## Design Decisions and Rationale

### 1. Monorepo Structure
**Decision**: Use pnpm workspaces
**Rationale**:
- Shared code reuse
- Atomic commits across packages
- Simplified dependency management
- Consistent tooling

### 2. Multi-Database Support
**Decision**: Support MongoDB, PostgreSQL, and Milvus
**Rationale**:
- Flexibility for different use cases
- Optimized performance for specific data types
- Gradual migration paths
- Vendor independence

### 3. Plugin Architecture
**Decision**: Microservice-based plugins
**Rationale**:
- Independent scaling
- Technology diversity
- Isolation and security
- Easier maintenance

### 4. TypeScript Everywhere
**Decision**: TypeScript for all code
**Rationale**:
- Type safety
- Better IDE support
- Self-documenting code
- Fewer runtime errors

## Future Architecture Evolution

The architecture is designed to evolve with these planned enhancements:

1. **Event-Driven Architecture**: Moving towards async event processing
2. **GraphQL API**: More flexible data fetching
3. **WebAssembly**: Client-side processing capabilities
4. **Microservices Decomposition**: Further service splitting
5. **Edge Computing**: Closer-to-user processing

---

This architectural overview provides the foundation for understanding FastGPT's design. For more detailed information on specific components, refer to the specialized architecture documents.