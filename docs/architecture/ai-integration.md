# AI Service Integration Architecture

## Introduction

FastGPT provides a unified abstraction layer for integrating multiple AI providers, enabling seamless switching between different models and services. This architecture supports various LLM providers, embedding models, and specialized AI services while maintaining a consistent interface throughout the platform.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    FastGPT Core                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Workflow      │  │  Knowledge Base │  │   Chat       │ │
│  │   Engine       │  │   System        │  │  Interface   │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 AI Service Layer                            │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │  LLM Manager    │  │  Embedding      │  │  Specialized │ │
│  │                 │  │  Manager        │  │  Services    │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                  Provider Adapters                          │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐ │
│  │   OpenAI  │  │ Anthropic │  │  Local    │  │ Custom   │ │
│  │  Adapter  │  │  Adapter  │  │  Adapter  │  │ Adapter  │ │
│  └───────────┘  └───────────┘  └───────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    External APIs                            │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌──────────┐ │
│  │ OpenAI API│  │Claude API │  │   Ollama  │  │   ...    │ │
│  └───────────┘  └───────────┘  └───────────┘  └──────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Core Interfaces

### Unified AI Interface

```typescript
// packages/global/ai/service.ts
export interface AIServiceConfig {
  provider: 'openai' | 'anthropic' | 'local' | 'custom';
  model: string;
  apiKey?: string;
  baseURL?: string;
  maxTokens?: number;
  temperature?: number;
  timeout?: number;
  retryConfig?: RetryConfig;
}

export interface ChatCompletionRequest {
  messages: ChatMessage[];
  stream?: boolean;
  temperature?: number;
  maxTokens?: number;
  stopSequences?: string[];
  tools?: ToolDefinition[];
}

export interface ChatMessage {
  role: 'system' | 'user' | 'assistant' | 'tool';
  content: string | ContentPart[];
  name?: string;
  toolCallId?: string;
  toolCalls?: ToolCall[];
}

export interface ToolCall {
  id: string;
  type: 'function';
  function: {
    name: string;
    arguments: string;
  };
}

export interface ContentPart {
  type: 'text' | 'image' | 'document';
  [key: string]: any;
}
```

### Base Provider Interface

```typescript
// packages/service/core/ai/provider.ts
export abstract class BaseAIProvider {
  protected config: AIServiceConfig;

  constructor(config: AIServiceConfig) {
    this.config = config;
  }

  abstract chatCompletion(request: ChatCompletionRequest): Promise<ChatCompletionResponse>;
  abstract streamChatCompletion(request: ChatCompletionRequest): AsyncIterable<ChatCompletionChunk>;
  abstract embeddings(input: string | string[]): Promise<EmbeddingResponse>;
  abstract countTokens(text: string): Promise<number>;

  // Common utilities
  protected async requestWithRetry<T>(
    fn: () => Promise<T>,
    retries = 3
  ): Promise<T> {
    for (let i = 0; i < retries; i++) {
      try {
        return await fn();
      } catch (error) {
        if (i === retries - 1) throw error;
        await this.delay(Math.pow(2, i) * 1000);
      }
    }
    throw new Error('Max retries exceeded');
  }

  private delay(ms: number): Promise<void> {
    return new Promise(resolve => setTimeout(resolve, ms));
  }
}
```

## Provider Implementations

### 1. OpenAI Provider

```typescript
// packages/service/core/ai/providers/openai.ts
export class OpenAIProvider extends BaseAIProvider {
  private client: OpenAI;

  constructor(config: AIServiceConfig) {
    super(config);
    this.client = new OpenAI({
      apiKey: config.apiKey,
      baseURL: config.baseURL,
      timeout: config.timeout || 30000
    });
  }

  async chatCompletion(request: ChatCompletionRequest): Promise<ChatCompletionResponse> {
    const response = await this.requestWithRetry(() =>
      this.client.chat.completions.create({
        model: this.config.model,
        messages: this.convertMessages(request.messages),
        temperature: request.temperature ?? this.config.temperature,
        max_tokens: request.maxTokens ?? this.config.maxTokens,
        stream: false,
        tools: request.tools?.map(tool => ({
          type: 'function',
          function: {
            name: tool.name,
            description: tool.description,
            parameters: tool.parameters
          }
        }))
      })
    );

    return this.convertResponse(response);
  }

  async *streamChatCompletion(
    request: ChatCompletionRequest
  ): AsyncIterable<ChatCompletionChunk> {
    const stream = await this.client.chat.completions.create({
      model: this.config.model,
      messages: this.convertMessages(request.messages),
      temperature: request.temperature ?? this.config.temperature,
      max_tokens: request.maxTokens ?? this.config.maxTokens,
      stream: true,
      tools: request.tools?.map(tool => ({
        type: 'function',
        function: {
          name: tool.name,
          description: tool.description,
          parameters: tool.parameters
        }
      }))
    });

    for await (const chunk of stream) {
      yield this.convertChunk(chunk);
    }
  }

  async embeddings(input: string | string[]): Promise<EmbeddingResponse> {
    const response = await this.client.embeddings.create({
      model: this.config.model,
      input: Array.isArray(input) ? input : [input]
    });

    return {
      data: response.data.map(item => ({
        embedding: item.embedding,
        index: item.index
      })),
      usage: response.usage
    };
  }

  async countTokens(text: string): Promise<number> {
    // Use tiktoken for accurate token counting
    const encoder = encodingForModel(this.config.model as TiktokenModel);
    const tokens = encoder.encode(text);
    encoder.free();
    return tokens.length;
  }

  private convertMessages(messages: ChatMessage[]): OpenAI.Chat.Completions.ChatCompletionMessageParam[] {
    return messages.map(msg => {
      if (typeof msg.content === 'string') {
        return {
          role: msg.role,
          content: msg.content,
          name: msg.name,
          tool_call_id: msg.toolCallId,
          tool_calls: msg.toolCalls?.map(tc => ({
            id: tc.id,
            type: tc.type,
            function: {
              name: tc.function.name,
              arguments: tc.function.arguments
            }
          }))
        };
      }
      // Handle multimodal content
      return {
        role: msg.role,
        content: msg.content as any,
        name: msg.name
      };
    });
  }
}
```

### 2. Anthropic Claude Provider

```typescript
// packages/service/core/ai/providers/anthropic.ts
export class AnthropicProvider extends BaseAIProvider {
  private client: Anthropic;

  constructor(config: AIServiceConfig) {
    super(config);
    this.client = new Anthropic({
      apiKey: config.apiKey,
      baseURL: config.baseURL,
      timeout: config.timeout || 30000
    });
  }

  async chatCompletion(request: ChatCompletionRequest): Promise<ChatCompletionResponse> {
    const systemMessage = request.messages.find(m => m.role === 'system');
    const messages = request.messages.filter(m => m.role !== 'system');

    const response = await this.requestWithRetry(() =>
      this.client.messages.create({
        model: this.config.model,
        max_tokens: request.maxTokens ?? this.config.maxTokens || 4096,
        temperature: request.temperature ?? this.config.temperature,
        system: systemMessage?.content as string,
        messages: this.convertMessages(messages),
        tools: request.tools?.map(tool => ({
          name: tool.name,
          description: tool.description,
          input_schema: tool.parameters
        }))
      })
    );

    return this.convertResponse(response);
  }

  async *streamChatCompletion(
    request: ChatCompletionRequest
  ): AsyncIterable<ChatCompletionChunk> {
    const systemMessage = request.messages.find(m => m.role === 'system');
    const messages = request.messages.filter(m => m.role !== 'system');

    const stream = await this.client.messages.create({
      model: this.config.model,
      max_tokens: request.maxTokens ?? this.config.maxTokens || 4096,
      temperature: request.temperature ?? this.config.temperature,
      system: systemMessage?.content as string,
      messages: this.convertMessages(messages),
      stream: true
    });

    for await (const chunk of stream) {
      yield this.convertChunk(chunk);
    }
  }

  // Anthropic doesn't provide embeddings, so we delegate to OpenAI
  async embeddings(input: string | string[]): Promise<EmbeddingResponse> {
    // Fallback to OpenAI for embeddings
    const openaiConfig = {
      ...this.config,
      model: 'text-embedding-3-small'
    };
    const openaiProvider = new OpenAIProvider(openaiConfig);
    return openaiProvider.embeddings(input);
  }

  private convertMessages(messages: ChatMessage[]): Anthropic.MessageParam[] {
    return messages.map(msg => ({
      role: msg.role as 'user' | 'assistant',
      content: msg.content as string
    }));
  }
}
```

### 3. Local Models Provider

```typescript
// packages/service/core/ai/providers/local.ts
export class LocalModelProvider extends BaseAIProvider {
  private endpoint: string;
  private modelType: 'ollama' | 'custom' | 'llamacpp';

  constructor(config: AIServiceConfig) {
    super(config);
    this.endpoint = config.baseURL || 'http://localhost:11434';
    this.modelType = this.detectModelType(config.model);
  }

  async chatCompletion(request: ChatCompletionRequest): Promise<ChatCompletionResponse> {
    const response = await this.requestWithRetry(async () => {
      const res = await fetch(`${this.endpoint}/api/chat`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          model: this.config.model,
          messages: this.convertMessages(request.messages),
          stream: false,
          options: {
            temperature: request.temperature ?? this.config.temperature,
            num_predict: request.maxTokens ?? this.config.maxTokens
          }
        })
      });

      if (!res.ok) {
        throw new Error(`Local model request failed: ${res.statusText}`);
      }

      return res.json();
    });

    return this.convertResponse(response);
  }

  async *streamChatCompletion(
    request: ChatCompletionRequest
  ): AsyncIterable<ChatCompletionChunk> {
    const response = await fetch(`${this.endpoint}/api/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: this.config.model,
        messages: this.convertMessages(request.messages),
        stream: true,
        options: {
          temperature: request.temperature ?? this.config.temperature,
          num_predict: request.maxTokens ?? this.config.maxTokens
        }
      })
    });

    const reader = response.body?.getReader();
    const decoder = new TextDecoder();

    if (!reader) {
      throw new Error('No response body');
    }

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;

      const chunk = decoder.decode(value);
      const lines = chunk.split('\n').filter(line => line.trim());

      for (const line of lines) {
        try {
          const data = JSON.parse(line);
          yield this.convertChunk(data);
        } catch (e) {
          // Ignore invalid JSON
        }
      }
    }
  }

  private detectModelType(model: string): 'ollama' | 'custom' | 'llamacpp' {
    if (model.includes('llama')) return 'llamacpp';
    if (this.endpoint.includes('ollama')) return 'ollama';
    return 'custom';
  }
}
```

## Specialized AI Services

### 1. OCR Services

```typescript
// packages/service/core/ai/services/ocr.ts
export interface OCRService {
  extractText(image: Buffer | string): Promise<OCRResult>;
  extractTextWithLayout(image: Buffer | string): Promise<OCRLayoutResult>;
}

export class SuryaOCR implements OCRService {
  async extractText(image: Buffer | string): Promise<OCRResult> {
    // Call Surya OCR plugin
    const response = await fetch('http://localhost:8000/ocr', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/octet-stream'
      },
      body: image
    });

    return response.json();
  }
}

export class TesseractOCR implements OCRService {
  async extractText(image: Buffer | string): Promise<OCRResult> {
    const worker = await createWorker('eng');
    const { data } = await worker.recognize(image);
    await worker.terminate();

    return {
      text: data.text,
      confidence: data.confidence,
      words: data.words.map(w => ({
        text: w.text,
        confidence: w.confidence,
        bbox: w.bbox
      }))
    };
  }
}
```

### 2. Speech-to-Text Services

```typescript
// packages/service/core/ai/services/stt.ts
export interface STTService {
  transcribe(audio: Buffer, options?: TranscribeOptions): Promise<TranscriptionResult>;
  transcribeStream(audio: AsyncIterable<Buffer>): AsyncIterable<TranscriptionChunk>;
}

export class SenseVoiceSTT implements STTService {
  private endpoint: string;

  constructor(endpoint = 'http://localhost:8001') {
    this.endpoint = endpoint;
  }

  async transcribe(audio: Buffer, options?: TranscribeOptions): Promise<TranscriptionResult> {
    const formData = new FormData();
    formData.append('audio', new Blob([audio]), 'audio.wav');
    if (options?.language) {
      formData.append('language', options.language);
    }

    const response = await fetch(`${this.endpoint}/transcribe`, {
      method: 'POST',
      body: formData
    });

    return response.json();
  }
}
```

### 3. Reranking Services

```typescript
// packages/service/core/ai/services/rerank.ts
export interface RerankService {
  rerank(query: string, documents: string[], topK?: number): Promise<RerankResult>;
}

export class BGEReranker implements RerankService {
  private endpoint: string;
  private model: string;

  constructor(model = 'BAAI/bge-reranker-base') {
    this.endpoint = 'http://localhost:8002';
    this.model = model;
  }

  async rerank(query: string, documents: string[], topK = 10): Promise<RerankResult> {
    const response = await fetch(`${this.endpoint}/rerank`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: this.model,
        query,
        documents,
        top_k: topK
      })
    });

    const result = await response.json();
    return {
      results: result.results.map((r: any) => ({
        index: r.index,
        score: r.score
      }))
    };
  }
}
```

## AI Service Manager

```typescript
// packages/service/core/ai/manager.ts
export class AIServiceManager {
  private providers: Map<string, BaseAIProvider> = new Map();
  private embeddings: Map<string, EmbeddingProvider> = new Map();
  private ocrServices: Map<string, OCRService> = new Map();
  private sttServices: Map<string, STTService> = new Map();
  private rerankers: Map<string, RerankService> = new Map();

  constructor() {
    this.initializeProviders();
    this.initializeSpecializedServices();
  }

  private async initializeProviders(): Promise<void> {
    // Load from configuration
    const configs = await this.loadProviderConfigs();

    for (const config of configs) {
      const provider = this.createProvider(config);
      this.providers.set(`${config.provider}:${config.model}`, provider);
    }
  }

  private createProvider(config: AIServiceConfig): BaseAIProvider {
    switch (config.provider) {
      case 'openai':
        return new OpenAIProvider(config);
      case 'anthropic':
        return new AnthropicProvider(config);
      case 'local':
        return new LocalModelProvider(config);
      default:
        throw new Error(`Unknown provider: ${config.provider}`);
    }
  }

  getProvider(providerKey: string): BaseAIProvider {
    const provider = this.providers.get(providerKey);
    if (!provider) {
      throw new Error(`Provider not found: ${providerKey}`);
    }
    return provider;
  }

  async chatCompletion(
    providerKey: string,
    request: ChatCompletionRequest
  ): Promise<ChatCompletionResponse> {
    const provider = this.getProvider(providerKey);
    return provider.chatCompletion(request);
  }

  async *streamChatCompletion(
    providerKey: string,
    request: ChatCompletionRequest
  ): AsyncIterable<ChatCompletionChunk> {
    const provider = this.getProvider(providerKey);
    yield* provider.streamChatCompletion(request);
  }

  async embeddings(
    providerKey: string,
    input: string | string[]
  ): Promise<EmbeddingResponse> {
    const provider = this.getProvider(providerKey);
    return provider.embeddings(input);
  }

  // Specialized service methods
  async extractText(serviceKey: string, image: Buffer): Promise<OCRResult> {
    const service = this.ocrServices.get(serviceKey);
    if (!service) throw new Error(`OCR service not found: ${serviceKey}`);
    return service.extractText(image);
  }

  async transcribeAudio(
    serviceKey: string,
    audio: Buffer,
    options?: TranscribeOptions
  ): Promise<TranscriptionResult> {
    const service = this.sttServices.get(serviceKey);
    if (!service) throw new Error(`STT service not found: ${serviceKey}`);
    return service.transcribe(audio, options);
  }

  async rerank(
    serviceKey: string,
    query: string,
    documents: string[],
    topK?: number
  ): Promise<RerankResult> {
    const service = this.rerankers.get(serviceKey);
    if (!service) throw new Error(`Reranker not found: ${serviceKey}`);
    return service.rerank(query, documents, topK);
  }
}
```

## Configuration Management

```typescript
// packages/service/core/ai/config.ts
export interface AIConfig {
  providers: ProviderConfig[];
  defaultProvider: string;
  embeddings: EmbeddingConfig;
  specialized: SpecializedServiceConfig;
}

export interface ProviderConfig {
  id: string;
  type: 'openai' | 'anthropic' | 'local';
  models: ModelConfig[];
  apiKey?: string;
  baseURL?: string;
  enabled: boolean;
}

export interface ModelConfig {
  name: string;
  displayName: string;
  type: 'chat' | 'embedding' | 'rerank';
  maxTokens?: number;
  costPerToken?: number;
  capabilities: string[];
}

// Load configuration from file or environment
export async function loadAIConfig(): Promise<AIConfig> {
  const configPath = process.env.AI_CONFIG_PATH || './ai-config.json';
  try {
    const configData = await fs.readFile(configPath, 'utf-8');
    return JSON.parse(configData);
  } catch (error) {
    // Fallback to environment variables
    return loadFromEnv();
  }
}

function loadFromEnv(): AIConfig {
  return {
    providers: [
      {
        id: 'openai',
        type: 'openai',
        models: [
          { name: 'gpt-4', displayName: 'GPT-4', type: 'chat', capabilities: ['chat', 'tools'] },
          { name: 'gpt-3.5-turbo', displayName: 'GPT-3.5 Turbo', type: 'chat', capabilities: ['chat'] }
        ],
        apiKey: process.env.OPENAI_API_KEY,
        baseURL: process.env.OPENAI_BASE_URL,
        enabled: !!process.env.OPENAI_API_KEY
      },
      {
        id: 'anthropic',
        type: 'anthropic',
        models: [
          { name: 'claude-3-sonnet', displayName: 'Claude 3 Sonnet', type: 'chat', capabilities: ['chat', 'vision'] }
        ],
        apiKey: process.env.ANTHROPIC_API_KEY,
        enabled: !!process.env.ANTHROPIC_API_KEY
      }
    ],
    defaultProvider: process.env.DEFAULT_AI_PROVIDER || 'openai',
    embeddings: {
      provider: 'openai',
      model: 'text-embedding-3-small',
      dimensions: 1536
    },
    specialized: {
      ocr: {
        default: 'surya',
        services: [
          { id: 'surya', endpoint: 'http://localhost:8000' },
          { id: 'tesseract', type: 'local' }
        ]
      },
      stt: {
        default: 'sensevoice',
        services: [
          { id: 'sensevoice', endpoint: 'http://localhost:8001' }
        ]
      },
      rerank: {
        default: 'bge',
        services: [
          { id: 'bge', model: 'BAAI/bge-reranker-base' }
        ]
      }
    }
  };
}
```

## Performance Optimization

### 1. Request Pooling

```typescript
// packages/service/core/ai/pool.ts
export class AIProviderPool {
  private pools: Map<string, Pool<BaseAIProvider>> = new Map();

  async acquire(providerKey: string): Promise<PoolItem<BaseAIProvider>> {
    let pool = this.pools.get(providerKey);
    if (!pool) {
      pool = this.createPool(providerKey);
      this.pools.set(providerKey, pool);
    }

    return pool.acquire();
  }

  private createPool(providerKey: string): Pool<BaseAIProvider> {
    const config = this.getProviderConfig(providerKey);
    const factory = {
      create: async () => this.createProvider(config),
      destroy: async (provider: BaseAIProvider) => {
        // Cleanup if needed
      }
    };

    return new Pool({
      factory,
      max: 10,
      min: 2,
      acquireTimeoutMillis: 30000,
      createTimeoutMillis: 5000
    });
  }
}
```

### 2. Caching Layer

```typescript
// packages/service/core/ai/cache.ts
export class AICache {
  private cache: Redis;

  constructor(redis: Redis) {
    this.cache = redis;
  }

  async getCachedEmbedding(text: string, model: string): Promise<number[] | null> {
    const key = this.generateEmbeddingKey(text, model);
    const cached = await this.cache.get(key);
    return cached ? JSON.parse(cached) : null;
  }

  async cacheEmbedding(text: string, model: string, embedding: number[]): Promise<void> {
    const key = this.generateEmbeddingKey(text, model);
    await this.cache.setex(key, 86400, JSON.stringify(embedding)); // 24 hours
  }

  private generateEmbeddingKey(text: string, model: string): string {
    const hash = crypto.createHash('sha256').update(text).digest('hex');
    return `embedding:${model}:${hash}`;
  }
}
```

## Error Handling and Monitoring

### 1. Error Handling

```typescript
// packages/service/core/ai/error-handler.ts
export class AIErrorHandler {
  async handleError(error: Error, context: ErrorContext): Promise<ErrorHandlingResult> {
    // Log error
    await this.logError(error, context);

    // Determine if retry is appropriate
    if (this.shouldRetry(error)) {
      return {
        action: 'retry',
        delay: this.calculateRetryDelay(context.attempt),
        maxRetries: 3
      };
    }

    // Check for fallback
    const fallbackProvider = this.getFallbackProvider(context.provider);
    if (fallbackProvider) {
      return {
        action: 'fallback',
        fallbackProvider
      };
    }

    // Fail
    return {
      action: 'fail',
      error: this.formatError(error, context)
    };
  }

  private shouldRetry(error: Error): boolean {
    // Retry on network errors, rate limits, temporary server errors
    return (
      error.message.includes('ECONNRESET') ||
      error.message.includes('rate_limit') ||
      error.message.includes('503')
    );
  }

  private calculateRetryDelay(attempt: number): number {
    // Exponential backoff with jitter
    return Math.min(1000 * Math.pow(2, attempt) + Math.random() * 1000, 30000);
  }
}
```

### 2. Monitoring

```typescript
// packages/service/core/ai/monitoring.ts
export class AIMonitoring {
  private metrics: MetricsCollector;

  constructor(metrics: MetricsCollector) {
    this.metrics = metrics;
  }

  recordAPICall(
    provider: string,
    model: string,
    endpoint: string,
    duration: number,
    success: boolean
  ): void {
    this.metrics.counter('ai_api_calls', {
      provider,
      model,
      endpoint,
      success: success.toString()
    }).increment();

    this.metrics.histogram('ai_api_duration', {
      provider,
      model
    }).record(duration);
  }

  recordTokenUsage(
    provider: string,
    model: string,
    promptTokens: number,
    completionTokens: number
  ): void {
    this.metrics.counter('ai_tokens_used', {
      provider,
      model,
      type: 'prompt'
    }).increment(promptTokens);

    this.metrics.counter('ai_tokens_used', {
      provider,
      model,
      type: 'completion'
    }).increment(completionTokens);
  }

  recordError(provider: string, errorType: string, error: Error): void {
    this.metrics.counter('ai_errors', {
      provider,
      errorType
    }).increment();

    this.metrics.logger().error('AI Service Error', {
      provider,
      errorType,
      message: error.message,
      stack: error.stack
    });
  }
}
```

## Best Practices

### 1. Provider Selection
- Choose appropriate provider based on use case
- Consider cost, speed, and quality trade-offs
- Use default providers for consistency
- Configure fallbacks for reliability

### 2. Configuration Management
- Store API keys securely
- Use environment variables for sensitive data
- Support multiple environments
- Validate configurations at startup

### 3. Performance Optimization
- Implement request pooling
- Cache embeddings and responses
- Use streaming for long responses
- Monitor token usage

### 4. Error Handling
- Implement robust error handling
- Use exponential backoff for retries
- Provide meaningful error messages
- Log errors for debugging

The AI integration architecture provides a flexible, extensible system for working with various AI providers while maintaining consistency and reliability across the platform.