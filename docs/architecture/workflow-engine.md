# Workflow Engine Architecture

## Introduction

The workflow engine is the heart of FastGPT, enabling visual orchestration of AI-powered workflows through a drag-and-drop interface. It provides a powerful, flexible system for building complex AI applications without writing code, while also supporting custom code execution for advanced use cases.

## Overview

The workflow engine consists of:
- **Visual Editor**: Drag-and-drop interface for building workflows
- **Execution Engine**: Core engine that runs workflows
- **Node System**: Extensible node types for different operations
- **Runtime Environment**: Sandboxed execution with safety controls

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Visual Layer                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Flow Canvas   │  │  Node Palette   │  │ Properties  │ │
│  │  (React Flow)   │  │                 │  │   Panel     │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ (Workflow JSON)
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                Workflow Engine Core                         │
│  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐ │
│  │   Parser        │  │  Executor       │  │   Context    │ │
│  │ (JSON → Graph)  │  │ (Run Nodes)     │  │  Management  │ │
│  └─────────────────┘  └─────────────────┘  └──────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Node System                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │   AI Nodes   │  │ Data Nodes   │  │   Control Nodes  │ │
│  │ (LLM, Chat)  │  │(File, Search)│  │ (If, Loop, Var) │ │
│  └──────────────┘  └──────────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   Runtime Environment                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐ │
│  │   Services   │  │   Sandbox    │  │   Storage        │ │
│  │ (AI, DB)     │  │ (Code Exec)  │  │ (Variables, Log)│ │
│  └──────────────┘  └──────────────┘  └──────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Node System

### Core Node Types

#### 1. System Nodes
- **workflowStart**: Entry point for workflow execution
  - Initializes global variables
  - Sets up execution context
  - Handles input parameters

- **systemInput**: Global input parameter handler
  - Accepts external inputs
  - Validates input data
  - Makes inputs available to workflow

#### 2. AI/LLM Nodes
- **ai**: Primary LLM interaction node
  - Configurable prompt templates
  - Supports multiple AI providers
  - Token management and streaming
  - Response parsing and validation

- **chatNode**: Conversational AI node
  - Context management
  - Conversation history
  - Memory persistence
  - Turn-based interaction

- **answerNode**: Response formatting node
  - Template-based formatting
  - Markdown processing
  - Output styling
  - Response validation

#### 3. Data Processing Nodes
- **datasetSearchNode**: Knowledge base search
  - Vector similarity search
  - Keyword search
  - Hybrid search
  - Result filtering

- **textEditor**: Text manipulation
  - Find and replace
  - Regex operations
  - Text transformation
  - String manipulation

- **readFiles**: File processing node
  - Multi-format support (PDF, DOCX, TXT)
  - OCR for images
  - Content extraction
  - Metadata handling

#### 4. Control Flow Nodes
- **ifElseNode**: Conditional branching
  - Expression evaluation
  - Multiple conditions
  - Nested conditions
  - Default branches

- **loop**: Iterative processing
  - Fixed iteration loops
  - Conditional loops
  - Loop variable management
  - Break and continue support

#### 5. Utility Nodes
- **httpRequest468**: External API integration
  - REST API calls
  - Authentication handling
  - Response parsing
  - Error handling

- **code**: Custom code execution
  - JavaScript/TypeScript support
  - Sandboxed execution
  - Library imports
  - Performance monitoring

- **pluginModule**: External plugin integration
  - Plugin discovery
  - Dynamic loading
  - Configuration management
  - Execution monitoring

### Node Implementation Structure

```typescript
// Base node interface
interface WorkflowNode {
  id: string;
  type: string;
  data: {
    inputs: Record<string, any>;
    outputs: Record<string, any>;
    config: NodeConfig;
  };
  position: { x: number; y: number };
}

// Node execution result
interface NodeExecutionResult {
  status: 'success' | 'error' | 'running';
  outputs: Record<string, any>;
  error?: Error;
  metadata?: Record<string, any>;
}

// Node executor interface
interface NodeExecutor {
  execute(
    node: WorkflowNode,
    context: ExecutionContext,
    runtime: RuntimeEnvironment
  ): Promise<NodeExecutionResult>;
}
```

## Execution Engine

### Core Components

#### 1. Workflow Parser
Converts visual workflow into executable graph:

```typescript
// packages/service/core/workflow/engine/parser.ts
export class WorkflowParser {
  parse(workflowJson: string): WorkflowGraph {
    const workflow = JSON.parse(workflowJson);

    // Validate workflow structure
    this.validate(workflow);

    // Build execution graph
    return this.buildGraph(workflow);
  }

  private validate(workflow: any): void {
    // Node validation
    // Connection validation
    // Cycle detection
  }

  private buildGraph(workflow: any): WorkflowGraph {
    // Create nodes
    // Build edges
    // Optimize execution order
  }
}
```

#### 2. Execution Scheduler
Manages node execution order and parallelization:

```typescript
// packages/service/core/workflow/engine/scheduler.ts
export class ExecutionScheduler {
  async execute(
    graph: WorkflowGraph,
    context: ExecutionContext
  ): Promise<WorkflowResult> {
    // Build execution plan
    const plan = this.createExecutionPlan(graph);

    // Execute nodes in order
    for (const batch of plan) {
      await this.executeBatch(batch, context);
    }

    return context.getResult();
  }

  private createExecutionPlan(graph: WorkflowGraph): ExecutionBatch[] {
    // Topological sort
    // Identify parallelizable nodes
    // Create execution batches
  }
}
```

#### 3. Context Manager
Manages execution state and data flow:

```typescript
// packages/service/core/workflow/engine/context.ts
export class ExecutionContext {
  private variables: Map<string, any>;
  private history: ExecutionStep[];

  getVariable(name: string): any {
    return this.variables.get(name);
  }

  setVariable(name: string, value: any): void {
    this.variables.set(name, value);
    this.history.push({
      type: 'variable_set',
      name,
      value,
      timestamp: Date.now()
    });
  }

  getNodeOutput(nodeId: string, outputName: string): any {
    // Retrieve node output from history
  }
}
```

### Execution Model

#### 1. Execution Flow
```
Start → Parse → Validate → Schedule → Execute → Log → Finish
  ↓      ↓        ↓         ↓        ↓       ↓       ↓
Init  JSON    Nodes   Batches   Nodes   Steps  Result
```

#### 2. Execution Limits
- **Maximum Workflow Runs**: 500 per session
- **Maximum Loop Iterations**: 50 per loop
- **Node Timeout**: Configurable per node type
- **Memory Limits**: Configured per execution
- **CPU Limits**: Enforced via sandbox

#### 3. Error Handling
```typescript
// packages/service/core/workflow/engine/error-handler.ts
export class ErrorHandler {
  handleError(
    error: Error,
    node: WorkflowNode,
    context: ExecutionContext
  ): ErrorHandlingResult {
    // Log error
    this.logError(error, node);

    // Check for retry configuration
    if (this.shouldRetry(node, error)) {
      return this.scheduleRetry(node, context);
    }

    // Execute error workflow
    if (node.data.config.errorWorkflow) {
      return this.executeErrorWorkflow(node, error, context);
    }

    // Fail fast
    return this.failWorkflow(error, context);
  }
}
```

## Visual Editor

### Components

#### 1. Flow Canvas
Built on React Flow:

```typescript
// packages/web/components/workflow/FlowCanvas/index.tsx
export const FlowCanvas: FC<FlowCanvasProps> = ({ nodes, edges, onNodesChange, onEdgesChange }) => {
  const [reactFlow, setReactFlow] = useState<Node[]>([]);
  const [instance, setInstance] = useState<ReactFlowInstance | null>(null);

  return (
    <ReactFlow
      nodes={reactFlow}
      edges={edges}
      onNodesChange={onNodesChange}
      onEdgesChange={onEdgesChange}
      nodeTypes={customNodeTypes}
      connectionMode={ConnectionMode.Loose}
      snapToGrid={true}
      snapGrid={[20, 20]}
    >
      <Controls />
      <MiniMap />
      <Background />
    </ReactFlow>
  );
};
```

#### 2. Node Palette
Draggable node types:

```typescript
// packages/web/components/workflow/NodePalette/index.tsx
export const NodePalette: FC = () => {
  const nodeCategories = [
    {
      name: 'AI',
      nodes: ['ai', 'chatNode', 'answerNode']
    },
    {
      name: 'Data',
      nodes: ['datasetSearchNode', 'textEditor', 'readFiles']
    },
    {
      name: 'Control',
      nodes: ['ifElseNode', 'loop', 'pluginModule']
    }
  ];

  return (
    <Box>
      {nodeCategories.map(category => (
        <NodeCategory key={category.name} category={category} />
      ))}
    </Box>
  );
};
```

#### 3. Properties Panel
Dynamic property editing:

```typescript
// packages/web/components/workflow/PropertiesPanel/index.tsx
export const PropertiesPanel: FC<PropertiesPanelProps> = ({ selectedNode }) => {
  const renderNodeConfig = () => {
    switch (selectedNode.type) {
      case 'ai':
        return <AINodeConfig node={selectedNode} />;
      case 'datasetSearchNode':
        return <DatasetSearchConfig node={selectedNode} />;
      default:
        return <DefaultNodeConfig node={selectedNode} />;
    }
  };

  return (
    <Box>
      <Heading size="sm">{selectedNode.type}</Heading>
      {renderNodeConfig()}
    </Box>
  );
};
```

## Runtime Environment

### Sandbox Execution

#### 1. Code Sandbox
Isolated code execution using NestJS:

```typescript
// projects/sandbox/src/app/modules/code/executor.ts
export class CodeExecutor {
  async execute(
    code: string,
    inputs: Record<string, any>
  ): Promise<any> {
    // Create isolated VM
    const vm = new NodeVM({
      console: 'inherit',
      sandbox: {
        inputs,
        console: createSafeConsole(),
        setTimeout: createSafeTimeout(),
      },
      require: {
        external: ['lodash', 'moment'],
        builtin: ['crypto', 'util']
      }
    });

    // Execute with timeout
    return await Promise.race([
      vm.run(code),
      this.timeout(5000)
    ]);
  }
}
```

#### 2. Resource Management
```typescript
// projects/sandbox/src/app/guards/resource.guard.ts
export class ResourceGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();

    // Check memory usage
    if (process.memoryUsage().heapUsed > MAX_MEMORY) {
      throw new Error('Memory limit exceeded');
    }

    // Check execution time
    if (Date.now() - request.startTime > MAX_EXECUTION_TIME) {
      throw new Error('Execution timeout');
    }

    return true;
  }
}
```

### Service Integration

#### 1. AI Service Integration
```typescript
// packages/service/core/workflow/nodes/ai.ts
export class AINodeExecutor implements NodeExecutor {
  async execute(
    node: WorkflowNode,
    context: ExecutionContext,
    runtime: RuntimeEnvironment
  ): Promise<NodeExecutionResult> {
    const config = node.data.config;

    // Get AI service
    const aiService = runtime.getAIService(config.provider);

    // Build prompt from template
    const prompt = this.buildPrompt(config.prompt, context);

    // Execute with streaming
    const response = await aiService.chat({
      messages: [{ role: 'user', content: prompt }],
      stream: config.stream,
      ...config.options
    });

    // Parse response
    const output = this.parseResponse(response, config.outputFormat);

    return {
      status: 'success',
      outputs: { response: output }
    };
  }
}
```

#### 2. Database Integration
```typescript
// packages/service/core/workflow/nodes/dataset-search.ts
export class DatasetSearchExecutor implements NodeExecutor {
  async execute(
    node: WorkflowNode,
    context: ExecutionContext,
    runtime: RuntimeEnvironment
  ): Promise<NodeExecutionResult> {
    const config = node.data.config;
    const query = context.getVariable(config.queryVariable);

    // Get dataset service
    const datasetService = runtime.getDatasetService();

    // Perform search
    const results = await datasetService.search({
      query,
      datasetId: config.datasetId,
      limit: config.limit,
      similarity: config.similarity
    });

    return {
      status: 'success',
      outputs: { results }
    };
  }
}
```

## Performance Optimization

### 1. Parallel Execution
- Identify independent nodes
- Execute in batches
- Manage concurrency limits
- Aggregate results

### 2. Caching
```typescript
// packages/service/core/workflow/engine/cache.ts
export class WorkflowCache {
  private cache: LRUCache<string, any>;

  async get(key: string): Promise<any> {
    return this.cache.get(key);
  }

  async set(key: string, value: any, ttl?: number): Promise<void> {
    this.cache.set(key, value, { ttl });
  }

  generateCacheKey(node: WorkflowNode, context: ExecutionContext): string {
    // Create deterministic key from node config and inputs
  }
}
```

### 3. Streaming
- Stream AI responses
- Progress updates
- Real-time logging
- Cancelable operations

## Monitoring and Debugging

### 1. Execution Logging
```typescript
// packages/service/core/workflow/engine/logger.ts
export class WorkflowLogger {
  log(nodeId: string, level: string, message: string, data?: any): void {
    const logEntry = {
      nodeId,
      level,
      message,
      data,
      timestamp: Date.now(),
      executionId: this.executionId
    };

    // Store in database
    this.saveLog(logEntry);

    // Send to client if streaming
    if (this.wsClient) {
      this.wsClient.send(JSON.stringify({
        type: 'log',
        ...logEntry
      }));
    }
  }
}
```

### 2. Debug Mode
- Step-by-step execution
- Variable inspection
- Node output preview
- Execution timeline

### 3. Metrics Collection
```typescript
// packages/service/core/workflow/engine/metrics.ts
export class WorkflowMetrics {
  collectNodeMetrics(nodeId: string, duration: number, status: string): void {
    this.metrics.record('node_execution', {
      nodeId,
      duration,
      status,
      timestamp: Date.now()
    });
  }

  collectWorkflowMetrics(workflowId: string, totalDuration: number): void {
    this.metrics.record('workflow_execution', {
      workflowId,
      totalDuration,
      nodeCount: this.nodeCount,
      timestamp: Date.now()
    });
  }
}
```

## Testing

### 1. Unit Tests
```typescript
// test/unit/workflow/nodes/ai.test.ts
describe('AINodeExecutor', () => {
  it('should execute AI node correctly', async () => {
    const node = createMockAINode();
    const context = createMockContext();
    const runtime = createMockRuntime();

    const executor = new AINodeExecutor();
    const result = await executor.execute(node, context, runtime);

    expect(result.status).toBe('success');
    expect(result.outputs).toHaveProperty('response');
  });
});
```

### 2. Integration Tests
```typescript
// test/integration/workflow/engine.test.ts
describe('Workflow Engine', () => {
  it('should execute complete workflow', async () => {
    const workflow = createTestWorkflow();
    const inputs = { message: 'Hello, world!' };

    const engine = new WorkflowEngine();
    const result = await engine.execute(workflow, inputs);

    expect(result.status).toBe('success');
    expect(result.outputs).toBeDefined();
  });
});
```

## Security Considerations

### 1. Input Validation
- Sanitize all inputs
- Validate JSON structure
- Check for malicious code
- Enforce size limits

### 2. Resource Limits
- Memory allocation limits
- CPU time restrictions
- Network access controls
- File system permissions

### 3. Isolation
- Separate process for execution
- Network isolation
- File system sandbox
- Permission restrictions

## Best Practices

### 1. Node Design
- Single responsibility
- Clear inputs/outputs
- Comprehensive error handling
- Well-documented configuration

### 2. Workflow Design
- Modular structure
- Clear data flow
- Proper error handling
- Performance considerations

### 3. Performance
- Minimize data transfer
- Use streaming for large outputs
- Cache expensive operations
- Optimize node ordering

The workflow engine provides a robust foundation for building complex AI applications while maintaining performance, security, and ease of use.