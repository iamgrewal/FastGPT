# FastGPT Documentation

Welcome to the FastGPT documentation suite. FastGPT is an AI Agent building platform that provides comprehensive out-of-the-box capabilities including data processing, model invocation, and visual workflow orchestration.

## Quick Start

- **New to FastGPT?** Start with [Getting Started](development/getting-started.md)
- **Want to understand the architecture?** Read the [Architecture Overview](architecture/overview.md)
- **Ready to deploy?** Check [Deployment Guide](deployment/overview.md)
- **Building with APIs?** See [API Documentation](api/overview.md)

## Documentation Structure

### 📋 Architecture Documentation
Understanding the system design and architectural decisions.

- [Architecture Overview](architecture/overview.md) - High-level system architecture
- [Monorepo Structure](architecture/monorepo-structure.md) - Understanding the codebase organization
- [Workflow Engine](architecture/workflow-engine.md) - Deep dive into workflow orchestration
- [AI Integration](architecture/ai-integration.md) - AI service integration and providers
- [Database Design](architecture/database-design.md) - Data modeling and storage architecture
- [Plugin System](architecture/plugin-system.md) - Extensible plugin architecture

### 💻 Development Documentation
Everything you need to develop with FastGPT.

- [Getting Started](development/getting-started.md) - Developer setup and onboarding
- [Coding Standards](development/coding-standards.md) - Code patterns and best practices
- [Testing](development/testing.md) - Testing architecture and practices
- [Contributing](development/contributing.md) - Contribution guidelines

### 🚀 Deployment Documentation
Production and deployment strategies.

- [Deployment Overview](deployment/overview.md) - Deployment strategies and options
- [Docker Deployment](deployment/docker.md) - Container-based deployment
- [Kubernetes Deployment](deployment/kubernetes.md) - K8s deployment with Helm
- [Environment Configuration](deployment/environment-configuration.md) - Configuration management

### 🔌 API Documentation
API reference and integration guides.

- [API Overview](api/overview.md) - API architecture and design principles
- [Authentication](api/authentication.md) - Auth mechanisms and API keys
- [Workflow API](api/workflow-api.md) - Workflow management API
- [Plugin API](api/plugin-api.md) - Plugin development API

### 🔒 Security Documentation
Security best practices and implementation.

- [Security Overview](security/overview.md) - Security architecture and principles
- [Authentication](security/authentication.md) - Authentication mechanisms
- [Data Protection](security/data-protection.md) - Data security measures

## Technology Stack

### Frontend
- **Framework**: NextJS 14.2.32 with React 18.3.1
- **Language**: TypeScript 5.1.3
- **UI Library**: Chakra UI 2.10.7
- **State Management**: Zustand, TanStack Query
- **Styling**: Custom Chakra theme with Framer Motion animations

### Backend
- **Runtime**: Node.js 20+
- **Main Framework**: NextJS API routes + Express
- **Sandbox**: NestJS 10.4.16
- **Languages**: TypeScript, Python (for sandbox)
- **Database**: MongoDB, PostgreSQL, Milvus

### AI/ML Integration
- **LLM Providers**: OpenAI, Anthropic Claude, Local Models
- **Vectorization**: OpenAI Embeddings, Local Models
- **File Processing**: PDF, DOCX, TXT, HTML, CSV
- **OCR**: Tesseract, Surya, Custom OCR

## Key Features

### 🎯 Visual Workflow Builder
- Drag-and-drop workflow designer
- 15+ node types including AI, data processing, and control flow
- Real-time execution monitoring
- Template marketplace

### 🔌 Plugin System
- Extensible architecture for custom functionality
- Model plugins for various AI providers
- Crawler plugins for web scraping
- Independent microservice deployment

### 📊 Knowledge Base
- Multi-format document import
- Intelligent text chunking
- Vector similarity search
- Real-time indexing

### 🛡️ Enterprise Ready
- Role-based access control (RBAC)
- API key management
- Audit logging
- Multi-tenancy support

## Getting Help

### Resources
- **Official Documentation**: https://doc.fastgpt.io/
- **GitHub Repository**: https://github.com/labring/FastGPT
- **Community**: Discord server
- **Online Platform**: https://fastgpt.io/

### Development Support
- **Bug Reports**: Create an issue on GitHub
- **Feature Requests**: Use discussions on GitHub
- **Security Issues**: Follow security reporting guidelines

### Quick Commands

```bash
# Install dependencies
pnpm install

# Start development environment
pnpm dev

# Run tests
pnpm test

# Build all projects
pnpm build

# Format code
pnpm format-code
```

## Documentation Standards

This documentation follows these principles:

1. **Language**: All documentation is in English
2. **Clarity**: Clear, concise explanations with examples
3. **Practicality**: Actionable guides and code examples
4. **Consistency**: Standardized structure and terminology
5. **Accessibility**: Suitable for various technical audiences

## Contributing to Documentation

We welcome contributions to improve the documentation:

1. Fork the repository
2. Create a documentation branch
3. Make your changes
4. Add examples where relevant
5. Submit a pull request

When contributing:
- Follow the existing documentation structure
- Use clear, simple language
- Include code examples with syntax highlighting
- Update related documents if needed
- Test any code examples you provide

## Version Information

This documentation corresponds to:
- **FastGPT Version**: Latest main branch
- **Last Updated**: 2025-12-03
- **Documentation Version**: 1.0.0

---

*For the most up-to-date information, always check the [official FastGPT documentation](https://doc.fastgpt.io/).*