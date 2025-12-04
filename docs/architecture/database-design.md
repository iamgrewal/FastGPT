# Database Architecture and Design

## Introduction

FastGPT employs a multi-database architecture to efficiently handle different types of data: structured application data, semi-structured documents, and high-dimensional vectors for similarity search. This design leverages the strengths of each database type while providing a unified abstraction layer.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Web App       │  │  Workflow       │  │   API        │ │
│  │   Services      │  │   Engine        │  │  Gateway     │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                 Database Abstraction Layer                  │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │  MongoDB ORM    │  │  PostgreSQL     │  │  Milvus      │ │
│  │  (Mongoose)     │  │  (TypeORM)      │  │  (Client)    │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Physical Databases                       │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │    MongoDB      │  │  PostgreSQL     │  │    Milvus    │ │
│  │  (Document DB)  │  │ (Relational +   │  │ (Vector DB)  │ │
│  │                 │  │  Vector Store)  │  │              │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Database Responsibilities

### 1. MongoDB - Primary Data Store
- **User Management**: Authentication, permissions, settings
- **Application Config**: Workflows, apps, templates
- **Metadata**: File metadata, plugin configurations
- **Audit Logs**: Activity logs, execution history
- **Session Data**: Chat sessions, temporary data

### 2. PostgreSQL - Structured & Vector Data (Optional)
- **Relational Data**: When ACID compliance is required
- **Vector Storage**: Using pgvector extension
- **Analytics**: Complex queries and reporting
- **Transactions**: Financial data, inventory

### 3. Milvus - High-Performance Vector Store (Optional)
- **Large-scale Vectors**: Millions to billions of vectors
- **Advanced Indexing**: HNSW, IVF_FLAT, PQ
- **Distributed**: Horizontal scaling
- **Performance**: Sub-millisecond queries

## MongoDB Schema Design

### Core Collections

#### 1. Users Collection
```typescript
// packages/service/common/mongo/models/User.ts
interface User {
  _id: ObjectId;
  username: string;
  email: string;
  passwordHash: string;
  avatar?: string;
  timezone: string;
  language: 'en' | 'zh' | 'ja';
  status: 'active' | 'inactive' | 'suspended';
  team?: TeamReference;
  roles: Role[];
  permissions: Permission[];
  createdAt: Date;
  updatedAt: Date;
  lastLoginAt?: Date;
  settings: {
    notifications: NotificationSettings;
    privacy: PrivacySettings;
    ui: UISettings;
  };
}

const userSchema = new Schema<User>({
  username: { type: String, required: true, unique: true },
  email: { type: String, required: true, unique: true },
  passwordHash: { type: String, required: true },
  team: { type: Schema.Types.ObjectId, ref: 'Team' },
  roles: [{ type: String, enum: ['admin', 'user', 'guest'] }],
  permissions: [{
    name: { type: String },
    resource: { type: String },
    actions: [{ type: String }]
  }],
  settings: {
    notifications: {
      email: { type: Boolean, default: true },
      push: { type: Boolean, default: true },
      inApp: { type: Boolean, default: true }
    },
    privacy: {
      profileVisible: { type: Boolean, default: true },
      activityVisible: { type: Boolean, default: true }
    },
    ui: {
      theme: { type: String, enum: ['light', 'dark', 'auto'], default: 'auto' },
      language: { type: String, enum: ['en', 'zh', 'ja'], default: 'en' }
    }
  }
}, {
  timestamps: true
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ username: 1 });
userSchema.index({ team: 1 });
```

#### 2. Apps Collection
```typescript
// packages/service/common/mongo/models/App.ts
interface App {
  _id: ObjectId;
  name: string;
  description?: string;
  avatar?: string;
  type: 'simple' | 'workflow' | 'plugin';
  owner: UserReference;
  team?: TeamReference;
  status: 'draft' | 'published' | 'archived';
  version: number;
  config: {
    // For simple apps
    chatConfig?: ChatConfig;
    // For workflow apps
    workflow?: WorkflowDefinition;
    // For plugin apps
    pluginConfig?: PluginConfig;
  };
  metrics: {
    totalChats: number;
    totalTokens: number;
    avgResponseTime: number;
    satisfaction: number;
  };
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
  publishedAt?: Date;
}

const appSchema = new Schema<App>({
  name: { type: String, required: true },
  owner: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  team: { type: Schema.Types.ObjectId, ref: 'Team' },
  config: {
    chatConfig: {
      model: { type: String, required: true },
      temperature: { type: Number, default: 0.7 },
      maxTokens: { type: Number, default: 2000 },
      systemPrompt: { type: String },
      welcomeMessage: { type: String }
    },
    workflow: {
      nodes: [WorkflowNodeSchema],
      edges: [WorkflowEdgeSchema],
      version: { type: String, default: '1.0' }
    }
  },
  metrics: {
    totalChats: { type: Number, default: 0 },
    totalTokens: { type: Number, default: 0 },
    avgResponseTime: { type: Number, default: 0 },
    satisfaction: { type: Number, default: 0 }
  }
}, {
  timestamps: true
});

// Indexes
appSchema.index({ owner: 1 });
appSchema.index({ team: 1 });
appSchema.index({ status: 1 });
appSchema.index({ type: 1 });
appSchema.index({ tags: 1 });
appSchema.index({ createdAt: -1 });
```

#### 3. Datasets Collection
```typescript
// packages/service/common/mongo/models/Dataset.ts
interface Dataset {
  _id: ObjectId;
  name: string;
  description?: string;
  type: 'dataset' | 'website' | 'api';
  owner: UserReference;
  team?: TeamReference;
  status: 'training' | 'active' | 'error';
  vectorDb: 'pg' | 'milvus';
  collections: DatasetCollection[];
  metrics: {
    documentCount: number;
    totalTokens: number;
    vectorCount: number;
    size: number; // in bytes
  };
  permissions: DatasetPermission[];
  createdAt: Date;
  updatedAt: Date;
}

const datasetSchema = new Schema<Dataset>({
  name: { type: String, required: true },
  type: { type: String, enum: ['dataset', 'website', 'api'], required: true },
  vectorDb: { type: String, enum: ['pg', 'milvus'], required: true },
  collections: [{
    name: { type: String, required: true },
    fileId: { type: Schema.Types.ObjectId, ref: 'File' },
    metadata: Schema.Types.Mixed,
    status: { type: String, enum: ['pending', 'processing', 'completed', 'error'] },
    chunkCount: { type: Number, default: 0 },
    vectorCount: { type: Number, default: 0 },
    error?: { type: String }
  }],
  metrics: {
    documentCount: { type: Number, default: 0 },
    totalTokens: { type: Number, default: 0 },
    vectorCount: { type: Number, default: 0 },
    size: { type: Number, default: 0 }
  }
}, {
  timestamps: true
});

// Indexes
datasetSchema.index({ owner: 1 });
datasetSchema.index({ team: 1 });
datasetSchema.index({ status: 1 });
datasetSchema.index({ vectorDb: 1 });
```

#### 4. Chats Collection
```typescript
// packages/service/common/mongo/models/Chat.ts
interface Chat {
  _id: ObjectId;
  appId: AppReference;
  userId: UserReference;
  sessionId: string;
  title?: string;
  status: 'active' | 'completed';
  messages: ChatMessage[];
  variables: Record<string, any>;
  metadata: {
    source: string;
    userAgent?: string;
    ip?: string;
    location?: string;
  };
  feedback?: ChatFeedback;
  createdAt: Date;
  updatedAt: Date;
}

const chatSchema = new Schema<Chat>({
  appId: { type: Schema.Types.ObjectId, ref: 'App', required: true },
  userId: { type: Schema.Types.ObjectId, ref: 'User', required: true },
  sessionId: { type: String, required: true },
  messages: [{
    id: { type: String, required: true },
    role: { type: String, enum: ['user', 'assistant', 'system'], required: true },
    content: { type: Schema.Types.Mixed, required: true },
    createdAt: { type: Date, default: Date.now },
    feedback: {
      rating: { type: Number, min: 1, max: 5 },
      comment: { type: String }
    }
  }],
  variables: { type: Schema.Types.Mixed, default: {} },
  feedback: {
    overall: { type: Number, min: 1, max: 5 },
    comment: { type: String },
    tags: [{ type: String }]
  }
}, {
  timestamps: true
});

// Indexes
chatSchema.index({ appId: 1 });
chatSchema.index({ userId: 1 });
chatSchema.index({ sessionId: 1 });
chatSchema.index({ createdAt: -1 });
```

#### 5. Files Collection
```typescript
// packages/service/common/mongo/models/File.ts
interface File {
  _id: ObjectId;
  filename: string;
  originalName: string;
  mimeType: string;
  size: number;
  path: string;
  storageType: 'local' | 's3' | 'minio';
  status: 'uploading' | 'processing' | 'completed' | 'error';
  metadata: {
    extractedText?: string;
    pageCount?: number;
    imageCount?: number;
    tableCount?: number;
    languages?: string[];
  };
  processing: {
    step: 'pending' | 'ocr' | 'extraction' | 'chunking' | 'embedding' | 'completed';
    progress: number;
    error?: string;
  };
  uploadedBy: UserReference;
  team?: TeamReference;
  createdAt: Date;
  updatedAt: Date;
}

const fileSchema = new Schema<File>({
  filename: { type: String, required: true },
  originalName: { type: String, required: true },
  mimeType: { type: String, required: true },
  size: { type: Number, required: true },
  storageType: { type: String, enum: ['local', 's3', 'minio'], required: true },
  processing: {
    step: { type: String, enum: ['pending', 'ocr', 'extraction', 'chunking', 'embedding', 'completed'], default: 'pending' },
    progress: { type: Number, default: 0 }
  },
  uploadedBy: { type: Schema.Types.ObjectId, ref: 'User', required: true }
}, {
  timestamps: true
});

// Indexes
fileSchema.index({ uploadedBy: 1 });
fileSchema.index({ team: 1 });
fileSchema.index({ status: 1 });
fileSchema.index({ createdAt: -1 });
```

## PostgreSQL Schema (When Used)

### Core Tables

#### 1. Vector Embeddings Table
```sql
-- PostgreSQL with pgvector
CREATE TABLE embeddings (
    id SERIAL PRIMARY KEY,
    dataset_id INTEGER NOT NULL,
    collection_id INTEGER NOT NULL,
    chunk_id VARCHAR(255) NOT NULL,
    content TEXT NOT NULL,
    metadata JSONB,
    embedding vector(1536), -- Dimension depends on model
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Vector index for similarity search
CREATE INDEX ON embeddings USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);

-- Metadata index
CREATE INDEX ON embeddings USING gin (metadata);

-- Content index for full-text search
CREATE INDEX ON embeddings USING gin (to_tsvector('english', content));
```

#### 2. Analytics Table
```sql
CREATE TABLE analytics (
    id BIGSERIAL PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    event_data JSONB,
    user_id VARCHAR(255),
    app_id VARCHAR(255),
    session_id VARCHAR(255),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address INET,
    user_agent TEXT
);

-- Partition by month for performance
CREATE TABLE analytics_y2024m01 PARTITION OF analytics
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- Indexes
CREATE INDEX ON analytics (event_type, timestamp DESC);
CREATE INDEX ON analytics (user_id, timestamp DESC);
CREATE INDEX ON analytics (app_id, timestamp DESC);
```

## Milvus Schema (When Used)

### Collection Design

```typescript
// packages/service/common/vector/milvus.ts
interface MilvusCollectionConfig {
  name: string;
  dimension: number;
  metricType: 'L2' | 'IP' | 'COSINE';
  indexType: 'IVF_FLAT' | 'IVF_SQ8' | 'HNSW' | 'ANNOY';
  indexParams: Record<string, any>;
}

export class MilvusVectorStore {
  async createCollection(config: MilvusCollectionConfig): Promise<void> {
    // Define collection schema
    const schema = {
      name: config.name,
      description: `Collection for ${config.name}`,
      fields: [
        {
          name: "id",
          data_type: DataType.VarChar,
          type_params: {
            max_length: 255,
          },
          autoID: false,
          is_primary: true,
        },
        {
          name: "dataset_id",
          data_type: DataType.VarChar,
          type_params: {
            max_length: 255,
          },
        },
        {
          name: "content",
          data_type: DataType.VarChar,
          type_params: {
            max_length: 65535,
          },
        },
        {
          name: "metadata",
          data_type: DataType.JSON,
        },
        {
          name: "embedding",
          data_type: DataType.FloatVector,
          type_params: {
            dim: config.dimension,
          },
        },
      ],
    };

    // Create collection
    await this.client.createCollection({
      collection_name: config.name,
      fields: schema.fields,
    });

    // Create index
    await this.client.createIndex({
      collection_name: config.name,
      field_name: "embedding",
      index_type: config.indexType,
      metric_type: config.metricType,
      params: config.indexParams,
    });

    // Load collection
    await this.client.loadCollection({
      collection_name: config.name,
    });
  }

  async insert(
    collectionName: string,
    data: VectorData[]
  ): Promise<string[]> {
    const ids = data.map(d => d.id);
    const datasetIds = data.map(d => d.datasetId);
    const contents = data.map(d => d.content);
    const metadata = data.map(d => d.metadata);
    const embeddings = data.map(d => d.embedding);

    await this.client.insert({
      collection_name: collectionName,
      data: [
        { name: "id", data: ids },
        { name: "dataset_id", data: datasetIds },
        { name: "content", data: contents },
        { name: "metadata", data: metadata },
        { name: "embedding", data: embeddings },
      ],
    });

    return ids;
  }

  async search(
    collectionName: string,
    queryVector: number[],
    topK: number = 10,
    filters?: string
  ): Promise<SearchResult[]> {
    const results = await this.client.search({
      collection_name: collectionName,
      vectors: [queryVector],
      limit: topK,
      expr: filters,
      output_fields: ["id", "dataset_id", "content", "metadata"],
    });

    return results[0].results.map(result => ({
      id: result.id,
      score: result.score,
      content: result.content,
      metadata: result.metadata,
    }));
  }
}
```

## Database Abstraction Layer

### Unified Vector Interface

```typescript
// packages/service/common/vector/controller.ts
export interface VectorData {
  id: string;
  datasetId: string;
  content: string;
  metadata: Record<string, any>;
  embedding: number[];
}

export interface VectorQuery {
  vector: number[];
  topK?: number;
  datasetId?: string;
  filters?: Record<string, any>;
  includeContent?: boolean;
}

export interface SearchResult {
  id: string;
  score: number;
  content?: string;
  metadata?: Record<string, any>;
}

export abstract class VectorDBController {
  abstract createCollection(name: string, dimension: number): Promise<void>;
  abstract insert(collectionName: string, data: VectorData[]): Promise<void>;
  abstract search(collectionName: string, query: VectorQuery): Promise<SearchResult[]>;
  abstract delete(collectionName: string, ids: string[]): Promise<void>;
  abstract update(collectionName: string, id: string, data: Partial<VectorData>): Promise<void>;
  abstract dropCollection(name: string): Promise<void>;
}
```

### MongoDB Vector Implementation
```typescript
// packages/service/common/vector/mongodb.ts
export class MongoDBVectorController extends VectorDBController {
  constructor(private db: Db) {
    super();
  }

  async createCollection(name: string, dimension: number): Promise<void> {
    await this.db.createCollection(name, {
      validator: {
        $jsonSchema: {
          bsonType: "object",
          required: ["id", "datasetId", "content", "embedding"],
          properties: {
            embedding: {
              bsonType: "array",
              items: { bsonType: "double" },
              minItems: dimension,
              maxItems: dimension
            }
          }
        }
      }
    });

    // Create vector index (MongoDB 6.0+)
    await this.db.collection(name).createIndex({
      embedding: "vectorSearch",
      type: "cosine"
    });
  }

  async insert(collectionName: string, data: VectorData[]): Promise<void> {
    await this.db.collection(collectionName).insertMany(data);
  }

  async search(collectionName: string, query: VectorQuery): Promise<SearchResult[]> {
    const pipeline: any[] = [
      {
        $vectorSearch: {
          queryVector: query.vector,
          path: "embedding",
          numCandidates: query.topK || 10,
          limit: query.topK || 10,
          index: collectionName + "_vector_index"
        }
      }
    ];

    if (query.filters) {
      pipeline.unshift({
        $match: {
          ...Object.entries(query.filters).map(([key, value]) => ({
            [`metadata.${key}`]: value
          })).reduce((acc, curr) => ({ ...acc, ...curr }), {})
        }
      });
    }

    const results = await this.db.collection(collectionName).aggregate(pipeline).toArray();

    return results.map(doc => ({
      id: doc.id,
      score: doc.score,
      content: doc.content,
      metadata: doc.metadata
    }));
  }
}
```

### PostgreSQL Vector Implementation
```typescript
// packages/service/common/vector/postgresql.ts
export class PostgreSQLVectorController extends VectorDBController {
  constructor(private pool: Pool) {
    super();
  }

  async createCollection(name: string, dimension: number): Promise<void> {
    await this.pool.query(`
      CREATE TABLE IF NOT EXISTS ${name} (
        id VARCHAR(255) PRIMARY KEY,
        dataset_id VARCHAR(255) NOT NULL,
        content TEXT NOT NULL,
        metadata JSONB,
        embedding vector(${dimension}),
        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
      )
    `);

    await this.pool.query(`
      CREATE INDEX IF NOT EXISTS idx_${name}_embedding
      ON ${name} USING ivfflat (embedding vector_cosine_ops)
      WITH (lists = 100)
    `);
  }

  async insert(collectionName: string, data: VectorData[]): Promise<void> {
    const values = data.map(d =>
      `('${d.id}', '${d.datasetId}', '${d.content.replace(/'/g, "''")}', '${JSON.stringify(d.metadata)}', '[${d.embedding.join(', ')}]')`
    ).join(', ');

    await this.pool.query(`
      INSERT INTO ${collectionName} (id, dataset_id, content, metadata, embedding)
      VALUES ${values}
    `);
  }

  async search(collectionName: string, query: VectorQuery): Promise<SearchResult[]> {
    let sql = `
      SELECT id, content, metadata, 1 - (embedding <=> $1) as score
      FROM ${collectionName}
    `;

    const params: any[] = [query.vector];
    let paramCount = 2;

    if (query.datasetId) {
      sql += ` WHERE dataset_id = $${paramCount}`;
      params.push(query.datasetId);
      paramCount++;
    }

    if (query.filters) {
      const whereClause = Object.entries(query.filters)
        .map(([key, value], index) => {
          if (index === 0 && !query.datasetId) {
            sql += ` WHERE metadata->>'${key}' = $${paramCount}`;
          } else {
            sql += ` AND metadata->>'${key}' = $${paramCount}`;
          }
          params.push(value);
          paramCount++;
          return '';
        });
    }

    sql += ` ORDER BY score DESC LIMIT $${paramCount}`;
    params.push(query.topK || 10);

    const result = await this.pool.query(sql, params);

    return result.rows.map(row => ({
      id: row.id,
      score: parseFloat(row.score),
      content: row.content,
      metadata: row.metadata
    }));
  }
}
```

## Performance Optimization

### 1. Indexing Strategy

#### MongoDB Indexes
```typescript
// Compound indexes for common queries
db.apps.createIndex({ owner: 1, status: 1, createdAt: -1 });
db.chats.createIndex({ appId: 1, userId: 1, sessionId: 1 });
db.datasets.createIndex({ owner: 1, status: 1, vectorDb: 1 });

// Text indexes for search
db.files.createIndex({ originalName: "text", "metadata.extractedText": "text" });
```

#### PostgreSQL Indexes
```sql
-- Partial indexes for better performance
CREATE INDEX ON embeddings (dataset_id)
WHERE dataset_id IS NOT NULL;

-- Expression indexes
CREATE INDEX ON embeddings (to_tsvector('english', content));

-- BRIN indexes for time-series data
CREATE INDEX ON analytics USING brin (timestamp);
```

### 2. Connection Pooling

```typescript
// packages/service/common/database/connection.ts
export class DatabaseManager {
  private mongoPool: MongoClient;
  private pgPool: Pool;

  constructor() {
    // MongoDB connection with pooling
    this.mongoPool = new MongoClient(process.env.MONGODB_URI, {
      maxPoolSize: 50,
      minPoolSize: 5,
      maxIdleTimeMS: 30000,
      waitQueueTimeoutMS: 5000
    });

    // PostgreSQL connection pool
    this.pgPool = new Pool({
      connectionString: process.env.POSTGRES_URI,
      max: 50,
      min: 5,
      idleTimeoutMillis: 30000,
      connectionTimeoutMillis: 5000
    });
  }

  async getMongoConnection(): Promise<Db> {
    if (!this.mongoPool) {
      await this.mongoPool.connect();
    }
    return this.mongoPool.db();
  }

  async getPGConnection(): Promise<PoolClient> {
    return this.pgPool.connect();
  }
}
```

### 3. Caching Layer

```typescript
// packages/service/common/database/cache.ts
export class DatabaseCache {
  private cache: Redis;

  constructor(redis: Redis) {
    this.cache = redis;
  }

  async getDocument(collection: string, id: string): Promise<any | null> {
    const cacheKey = `mongo:${collection}:${id}`;
    const cached = await this.cache.get(cacheKey);
    return cached ? JSON.parse(cached) : null;
  }

  async cacheDocument(collection: string, id: string, document: any, ttl = 3600): Promise<void> {
    const cacheKey = `mongo:${collection}:${id}`;
    await this.cache.setex(cacheKey, ttl, JSON.stringify(document));
  }

  async invalidateCollection(collection: string): Promise<void> {
    const pattern = `mongo:${collection}:*`;
    const keys = await this.cache.keys(pattern);
    if (keys.length > 0) {
      await this.cache.del(...keys);
    }
  }

  // Vector search caching
  async cacheVectorSearch(
    collectionName: string,
    queryHash: string,
    results: SearchResult[],
    ttl = 300
  ): Promise<void> {
    const cacheKey = `vector:${collectionName}:${queryHash}`;
    await this.cache.setex(cacheKey, ttl, JSON.stringify(results));
  }

  async getCachedVectorSearch(
    collectionName: string,
    queryHash: string
  ): Promise<SearchResult[] | null> {
    const cacheKey = `vector:${collectionName}:${queryHash}`;
    const cached = await this.cache.get(cacheKey);
    return cached ? JSON.parse(cached) : null;
  }
}
```

## Migration Strategy

### 1. Migration Framework

```typescript
// packages/service/common/database/migration.ts
export interface Migration {
  version: string;
  description: string;
  up: () => Promise<void>;
  down: () => Promise<void>;
}

export class MigrationManager {
  private migrations: Map<string, Migration> = new Map();

  async runMigrations(): Promise<void> {
    const currentVersion = await this.getCurrentVersion();
    const pendingMigrations = this.getPendingMigrations(currentVersion);

    for (const migration of pendingMigrations) {
      try {
        await migration.up();
        await this.recordMigration(migration.version, migration.description);
        console.log(`Migration ${migration.version} completed successfully`);
      } catch (error) {
        console.error(`Migration ${migration.version} failed:`, error);
        throw error;
      }
    }
  }

  async rollback(targetVersion: string): Promise<void> {
    const currentVersion = await this.getCurrentVersion();
    const migrationsToRollback = this.getMigrationsToRollback(currentVersion, targetVersion);

    for (const migration of migrationsToRollback) {
      try {
        await migration.down();
        await this.removeMigration(migration.version);
        console.log(`Rolled back migration ${migration.version}`);
      } catch (error) {
        console.error(`Rollback of ${migration.version} failed:`, error);
        throw error;
      }
    }
  }
}
```

### 2. Database Health Monitoring

```typescript
// packages/service/common/database/health.ts
export class DatabaseHealth {
  async checkMongoDB(): Promise<HealthStatus> {
    try {
      const start = Date.now();
      await this.mongo.db().admin().ping();
      const latency = Date.now() - start;

      return {
        status: 'healthy',
        latency,
        details: {
          connections: await this.getConnectionCount(),
          collections: await this.getCollectionCount()
        }
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        error: error.message
      };
    }
  }

  async checkPostgreSQL(): Promise<HealthStatus> {
    try {
      const start = Date.now();
      await this.pg.query('SELECT 1');
      const latency = Date.now() - start;

      const [connectionsResult] = await this.pg.query(
        'SELECT count(*) as count FROM pg_stat_activity'
      );

      return {
        status: 'healthy',
        latency,
        details: {
          connections: connectionsResult.rows[0].count,
          size: await this.getDatabaseSize()
        }
      };
    } catch (error) {
      return {
        status: 'unhealthy',
        error: error.message
      };
    }
  }
}
```

## Best Practices

### 1. Data Modeling
- Use appropriate data types
- Normalize when necessary, denormalize for performance
- Implement soft deletes with audit trails
- Use versioning for critical data

### 2. Performance
- Create indexes based on query patterns
- Use compound indexes for multi-field queries
- Monitor and optimize slow queries
- Implement appropriate caching strategies

### 3. Scalability
- Use connection pooling
- Implement read replicas for read-heavy workloads
- Partition large collections/tables
- Consider sharding for very large datasets

### 4. Security
- Encrypt sensitive data at rest
- Use parameterized queries
- Implement row-level security
- Regular backups and testing

The database architecture provides a robust foundation for storing and retrieving all types of data efficiently while maintaining flexibility and scalability.