# Plugin Architecture

## Introduction

FastGPT features a sophisticated plugin architecture that enables extensibility and modularity. This system allows developers to easily add new AI models, processing capabilities, and integrations without modifying the core system. Each plugin runs as an independent microservice, providing isolation, scalability, and independent deployment capabilities.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    FastGPT Core                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │  Plugin Manager │  │  Workflow       │  │   API        │ │
│  │                 │  │  Engine         │  │  Gateway     │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Plugin Communication Layer                   │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   HTTP API      │  │  WebSocket      │  │  Message     │ │
│  │   Client        │  │   Client        │  │   Queue      │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Plugin Ecosystem                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Model         │  │   Crawler       │  │   Custom     │ │
│  │   Plugins       │  │   Plugins       │  │   Plugins    │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Plugin Types

### 1. Model Plugins (`plugins/model/`)
Provide AI/ML capabilities including LLMs, embeddings, OCR, speech processing, etc.

#### Available Model Plugins
- **LLM Providers**:
  - Baichuan2: Chinese language model
  - ChatGLM2: Bilingual chat model
  - Ollama Integration: Local model support

- **OCR Services**:
  - Surya: Multi-language OCR
  - PaddleOCR: Chinese OCR
  - EasyOCR: General-purpose OCR

- **PDF Processing**:
  - PDF-MinerU: Advanced PDF extraction
  - PDF-Marker: Structured PDF processing
  - PDF-Mistral: AI-powered PDF understanding

- **Audio Processing**:
  - STT-SenseVoice: Speech recognition
  - TTS-CoSEVoice: Speech synthesis

- **Reranking**:
  - BGE-Reranker: Result refinement
  - Cohere Rerank: Commercial reranking

### 2. Crawler Plugins (`plugins/webcrawler/`)
Provide web scraping and content extraction capabilities.

#### Available Crawler Plugins
- **Web Crawlers**:
  - Playwright Crawler: Dynamic content
  - Scrapy Crawler: High-performance scraping
  - Selenium Crawler: Legacy browser support

- **Search Integrations**:
  - Google Search: Web search API
  - Bing Search: Microsoft search
  - Custom Search Engines

## Plugin Architecture

### Core Components

#### 1. Plugin Manager
```typescript
// packages/service/core/plugin/manager.ts
export interface PluginConfig {
  id: string;
  name: string;
  version: string;
  type: 'model' | 'crawler' | 'custom';
  endpoint: string;
  status: 'active' | 'inactive' | 'error';
  capabilities: PluginCapability[];
  health: PluginHealth;
  metadata: {
    description: string;
    author: string;
    tags: string[];
    dependencies: string[];
  };
}

export class PluginManager {
  private plugins: Map<string, PluginInstance> = new Map();
  private registry: PluginRegistry;
  private healthChecker: PluginHealthChecker;

  constructor() {
    this.registry = new PluginRegistry();
    this.healthChecker = new PluginHealthChecker();
    this.initializePlugins();
  }

  async loadPlugin(config: PluginConfig): Promise<void> {
    // Validate plugin
    await this.validatePlugin(config);

    // Create plugin instance
    const instance = await this.createPluginInstance(config);

    // Register capabilities
    for (const capability of config.capabilities) {
      this.registry.registerCapability(config.id, capability);
    }

    // Start health checking
    this.healthChecker.startMonitoring(config.id, config.endpoint);

    // Store plugin
    this.plugins.set(config.id, instance);
  }

  async unloadPlugin(pluginId: string): Promise<void> {
    const plugin = this.plugins.get(pluginId);
    if (!plugin) return;

    // Stop health checking
    this.healthChecker.stopMonitoring(pluginId);

    // Unregister capabilities
    this.registry.unregisterCapabilities(pluginId);

    // Cleanup plugin
    await plugin.cleanup();

    // Remove from registry
    this.plugins.delete(pluginId);
  }

  async executePlugin(
    pluginId: string,
    capability: string,
    input: any,
    options?: ExecutionOptions
  ): Promise<any> {
    const plugin = this.plugins.get(pluginId);
    if (!plugin) {
      throw new Error(`Plugin not found: ${pluginId}`);
    }

    // Check if plugin supports capability
    if (!plugin.hasCapability(capability)) {
      throw new Error(`Plugin ${pluginId} does not support ${capability}`);
    }

    // Execute with timeout and retry
    return this.executeWithTimeout(
      () => plugin.execute(capability, input),
      options?.timeout || 30000
    );
  }

  findPlugins(capability: string): PluginInstance[] {
    return Array.from(this.plugins.values()).filter(p =>
      p.hasCapability(capability)
    );
  }
}
```

#### 2. Plugin Interface
```typescript
// packages/service/core/plugin/interface.ts
export interface PluginInstance {
  config: PluginConfig;
  client: PluginClient;
  hasCapability(capability: string): boolean;
  execute(capability: string, input: any): Promise<any>;
  cleanup(): Promise<void>;
}

export interface PluginCapability {
  name: string;
  description: string;
  inputSchema: JSONSchema;
  outputSchema: JSONSchema;
  parameters?: PluginParameter[];
}

export interface PluginParameter {
  name: string;
  type: 'string' | 'number' | 'boolean' | 'object' | 'array';
  required: boolean;
  default?: any;
  description?: string;
  validation?: ValidationRule[];
}

export class PluginClient {
  constructor(
    private endpoint: string,
    private timeout = 30000,
    private retries = 3
  ) {}

  async call(endpoint: string, data: any): Promise<any> {
    const url = `${this.endpoint}${endpoint}`;

    for (let attempt = 0; attempt < this.retries; attempt++) {
      try {
        const response = await fetch(url, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'X-Plugin-Version': '1.0'
          },
          body: JSON.stringify(data),
          signal: AbortSignal.timeout(this.timeout)
        });

        if (!response.ok) {
          throw new PluginError(
            `Plugin request failed: ${response.status} ${response.statusText}`,
            response.status
          );
        }

        return await response.json();
      } catch (error) {
        if (attempt === this.retries - 1) throw error;

        // Exponential backoff
        await this.delay(Math.pow(2, attempt) * 1000);
      }
    }

    throw new Error('Max retries exceeded');
  }

  async callStream(endpoint: string, data: any): Promise<AsyncIterable<any>> {
    const url = `${this.endpoint}${endpoint}`;

    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'text/event-stream'
      },
      body: JSON.stringify(data)
    });

    if (!response.ok) {
      throw new PluginError(
        `Plugin stream request failed: ${response.status}`,
        response.status
      );
    }

    const reader = response.body?.getReader();
    const decoder = new TextDecoder();

    if (!reader) {
      throw new Error('No response body for streaming');
    }

    return this.createStreamIterator(reader, decoder);
  }

  private async *createStreamIterator(
    reader: ReadableStreamDefaultReader,
    decoder: TextDecoder
  ): AsyncIterable<any> {
    let buffer = '';

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      buffer += decoder.decode(value, { stream: true });
      const lines = buffer.split('\n');
      buffer = lines.pop() || '';

      for (const line of lines) {
        if (line.startsWith('data: ')) {
          const data = line.slice(6);
          if (data === '[DONE]') return;

          try {
            yield JSON.parse(data);
          } catch (e) {
            // Ignore invalid JSON
          }
        }
      }
    }
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## Plugin Development

### Creating a New Plugin

#### 1. Plugin Structure
```
plugins/my-plugin/
├── package.json
├── Dockerfile
├── src/
│   ├── index.ts         # Plugin entry point
│   ├── server.ts        # Express server
│   ├── handlers/        # Capability handlers
│   │   └── capability.ts
│   └── utils/           # Plugin utilities
├── config/
│   └── plugin.json      # Plugin configuration
└── docs/
    └── README.md        # Plugin documentation
```

#### 2. Plugin Configuration
```json
// plugins/my-plugin/config/plugin.json
{
  "id": "my-plugin",
  "name": "My Custom Plugin",
  "version": "1.0.0",
  "type": "model",
  "description": "A custom plugin for FastGPT",
  "author": "Your Name",
  "tags": ["ai", "custom"],
  "capabilities": [
    {
      "name": "process",
      "description": "Process input data",
      "endpoint": "/process",
      "inputSchema": {
        "type": "object",
        "properties": {
          "text": { "type": "string" },
          "options": { "type": "object" }
        },
        "required": ["text"]
      },
      "outputSchema": {
        "type": "object",
        "properties": {
          "result": { "type": "string" },
          "confidence": { "type": "number" }
        }
      }
    }
  ],
  "dependencies": [],
  "resources": {
    "cpu": "500m",
    "memory": "512Mi",
    "gpu": "0"
  }
}
```

#### 3. Plugin Server Implementation
```typescript
// plugins/my-plugin/src/server.ts
import express from 'express';
import cors from 'cors';
import { ProcessHandler } from './handlers/process';

const app = express();
const port = process.env.PORT || 8080;

// Middleware
app.use(cors());
app.use(express.json({ limit: '10mb' }));

// Health check
app.get('/health', (req, res) => {
  res.json({ status: 'healthy', timestamp: new Date().toISOString() });
});

// Plugin info
app.get('/plugin', (req, res) => {
  const config = require('../config/plugin.json');
  res.json(config);
});

// Capability endpoints
app.post('/process', async (req, res) => {
  try {
    const handler = new ProcessHandler();
    const result = await handler.handle(req.body);
    res.json(result);
  } catch (error) {
    res.status(500).json({
      error: error.message,
      code: 'PROCESS_ERROR'
    });
  }
});

// Streaming endpoint (optional)
app.post('/process/stream', async (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  try {
    const handler = new ProcessHandler();
    for await (const chunk of handler.handleStream(req.body)) {
      res.write(`data: ${JSON.stringify(chunk)}\n\n`);
    }
    res.write('data: [DONE]\n\n');
  } catch (error) {
    res.write(`data: ${JSON.stringify({ error: error.message })}\n\n`);
  } finally {
    res.end();
  }
});

app.listen(port, () => {
  console.log(`Plugin server listening on port ${port}`);
});
```

#### 4. Capability Handler
```typescript
// plugins/my-plugin/src/handlers/process.ts
export class ProcessHandler {
  async handle(input: ProcessInput): Promise<ProcessOutput> {
    // Validate input
    this.validateInput(input);

    // Process data
    const result = await this.processText(input.text, input.options);

    return {
      result: result.text,
      confidence: result.confidence,
      metadata: {
        processingTime: Date.now(),
        version: '1.0.0'
      }
    };
  }

  async *handleStream(input: ProcessInput): AsyncIterable<ProcessChunk> {
    // Stream processing implementation
    for (const chunk of input.text.split(' ')) {
      const processed = await this.processChunk(chunk);
      yield {
        chunk: processed,
        progress: this.calculateProgress(chunk, input.text),
        done: false
      };
    }
    yield { chunk: '', progress: 100, done: true };
  }

  private validateInput(input: ProcessInput): void {
    if (!input.text || input.text.length === 0) {
      throw new Error('Text is required');
    }
  }

  private async processText(text: string, options?: any): Promise<any> {
    // Your processing logic here
    return {
      text: text.toUpperCase(),
      confidence: 0.95
    };
  }

  private async processChunk(chunk: string): Promise<string> {
    // Process individual chunk
    return chunk.toUpperCase();
  }

  private calculateProgress(current: string, total: string): number {
    return Math.round((current.length / total.length) * 100);
  }
}

interface ProcessInput {
  text: string;
  options?: any;
}

interface ProcessOutput {
  result: string;
  confidence: number;
  metadata: Record<string, any>;
}

interface ProcessChunk {
  chunk: string;
  progress: number;
  done: boolean;
}
```

#### 5. Dockerfile
```dockerfile
# plugins/my-plugin/Dockerfile
FROM node:18-alpine

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy source code
COPY src/ ./src/
COPY config/ ./config/

# Build
RUN npm run build

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1

# Run
CMD ["node", "dist/server.js"]
```

## Plugin Deployment

### 1. Docker Compose Deployment
```yaml
# docker-compose.plugins.yml
version: '3.8'

services:
  plugin-registry:
    image: fastgpt/plugin-registry
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=${DATABASE_URL}
    volumes:
      - ./plugins:/app/plugins

  plugin-proxy:
    image: fastgpt/plugin-proxy
    ports:
      - "3001:3001"
    depends_on:
      - plugin-registry

  # Individual plugins
  surya-ocr:
    image: fastgpt/surya-ocr
    ports:
      - "8000:8000"
    environment:
      - PLUGIN_ID=surya-ocr
      - PLUGIN_ENDPOINT=http://localhost:8000
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1Gi

  bge-reranker:
    image: fastgpt/bge-reranker
    ports:
      - "8001:8001"
    environment:
      - PLUGIN_ID=bge-reranker
      - MODEL_PATH=/models/bge-reranker-base
    volumes:
      - ./models:/models
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2Gi
```

### 2. Kubernetes Deployment
```yaml
# k8s/plugin-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: surya-ocr-plugin
  namespace: fastgpt
spec:
  replicas: 3
  selector:
    matchLabels:
      app: surya-ocr-plugin
  template:
    metadata:
      labels:
        app: surya-ocr-plugin
    spec:
      containers:
      - name: plugin
        image: fastgpt/surya-ocr:latest
        ports:
        - containerPort: 8000
        env:
        - name: PLUGIN_ID
          value: "surya-ocr"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: fastgpt-secrets
              key: redis-url
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 1Gi
        livenessProbe:
          httpGet:
            path: /health
            port: 8000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8000
          initialDelaySeconds: 5
          periodSeconds: 5

---
apiVersion: v1
kind: Service
metadata:
  name: surya-ocr-service
  namespace: fastgpt
spec:
  selector:
    app: surya-ocr-plugin
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8000
  type: ClusterIP
```

### 3. Helm Chart
```yaml
# helm/fastgpt-plugin/templates/deployment.yaml
{{- range .Values.plugins }}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .name }}
  labels:
    app: {{ .name }}
spec:
  replicas: {{ .replicas | default 1 }}
  selector:
    matchLabels:
      app: {{ .name }}
  template:
    metadata:
      labels:
        app: {{ .name }}
    spec:
      containers:
      - name: {{ .name }}
        image: {{ .image }}
        ports:
        - containerPort: {{ .port | default 8080 }}
        env:
        {{- range $key, $value := .env }}
        - name: {{ $key }}
          value: {{ $value }}
        {{- end }}
        resources:
          requests:
            cpu: {{ .resources.requests.cpu | default "100m" }}
            memory: {{ .resources.requests.memory | default "256Mi" }}
          limits:
            cpu: {{ .resources.limits.cpu | default "500m" }}
            memory: {{ .resources.limits.memory | default "1Gi" }}
{{- end }}
```

## Plugin Communication

### 1. Message Format
```typescript
interface PluginMessage {
  id: string;
  type: 'request' | 'response' | 'event';
  capability: string;
  data: any;
  timestamp: number;
  correlationId?: string;
}
```

### 2. Event System
```typescript
// packages/service/core/plugin/events.ts
export class PluginEventBus {
  private handlers: Map<string, EventHandler[]> = new Map();

  subscribe(event: string, handler: EventHandler): void {
    if (!this.handlers.has(event)) {
      this.handlers.set(event, []);
    }
    this.handlers.get(event)!.push(handler);
  }

  async publish(event: PluginEvent): Promise<void> {
    const handlers = this.handlers.get(event.type) || [];
    await Promise.all(
      handlers.map(handler => handler(event))
    );
  }
}

// Example event types
export enum PluginEventType {
  PLUGIN_LOADED = 'plugin.loaded',
  PLUGIN_UNLOADED = 'plugin.unloaded',
  PLUGIN_ERROR = 'plugin.error',
  CAPABILITY_EXECUTED = 'capability.executed'
}
```

## Best Practices

### 1. Design Principles
- **Single Responsibility**: Each plugin does one thing well
- **Stateless**: Avoid storing state between requests
- **Idempotent**: Same input should produce same output
- **Graceful Degradation**: Handle failures gracefully

### 2. Performance
- **Connection Pooling**: Reuse connections for external services
- **Batch Processing**: Process multiple items together when possible
- **Caching**: Cache expensive computations
- **Streaming**: Use streaming for large data

### 3. Security
- **Input Validation**: Validate all inputs
- **Output Sanitization**: Sanitize outputs
- **Authentication**: Use secure authentication
- **Rate Limiting**: Implement rate limiting

### 4. Monitoring
- **Health Checks**: Provide health endpoints
- **Metrics**: Expose performance metrics
- **Logging**: Log important events
- **Error Tracking**: Track and report errors

### 5. Testing
- **Unit Tests**: Test individual components
- **Integration Tests**: Test plugin integration
- **Load Tests**: Test under load
- **Contract Tests**: Test API contracts

## Troubleshooting

### Common Issues

1. **Plugin Not Starting**
   - Check Docker logs: `docker logs <plugin-name>`
   - Verify port availability
   - Check resource limits

2. **Plugin Not Responding**
   - Check health endpoint
   - Verify network connectivity
   - Check timeout settings

3. **Performance Issues**
   - Monitor resource usage
   - Check for memory leaks
   - Optimize algorithms

4. **Integration Problems**
   - Verify API contracts
   - Check version compatibility
   - Review plugin configuration

The plugin architecture provides a flexible, scalable system for extending FastGPT's capabilities while maintaining security and performance.