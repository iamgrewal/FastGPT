# FastGPT Plugins

This directory contains auxiliary FastGPT subprojects that provide optional extensions and enhanced capabilities for the main FastGPT platform. These plugins are designed to be deployed as independent microservices that communicate with FastGPT via REST APIs.

## Overview

The plugins architecture allows FastGPT to be extended with additional capabilities without modifying the core system. Each plugin runs as an independent service and can be deployed, scaled, and maintained separately.

## Plugin Types

### Model Plugins (`/model/`)

Model plugins provide specialized AI capabilities and private model deployments. These include:

#### Large Language Models (LLMs)
- **llm-ChatGLM2** - ChatGLM2 language model with OpenAI-compatible API
- **llm-Baichuan2** - Baichuan2 language model with OpenAI-compatible API

#### Document Processing
- **pdf-marker** - High-performance PDF to Markdown conversion service with GPU acceleration
- **pdf-mineru** - PDF parsing and extraction service
- **pdf-mistral** - PDF processing using Mistral-based models
- **ocr-surya** - Optical Character Recognition using Surya models

#### Speech Processing
- **stt-sensevoice** - Speech-to-Text conversion using SenseVoice
- **tts-cosevoice** - Text-to-Speech conversion using CosyVoice

#### Text Processing
- **rerank-bge** - Text reranking using BGE (Beijing Academy of Artificial Intelligence) models
  - Multiple variants: base, large, and v2-m3 models

### Web Crawler Plugin (`/webcrawler/`)

Advanced web scraping and search capabilities that integrate with FastGPT for real-time information retrieval.

**Features:**
- **Web Search**: Multi-engine search support (Baidu, SearchXNG)
- **Page Content Extraction**: Extract structured content from web pages
- **Deep Search**: Comprehensive crawling with configurable depth
- **Authentication**: Token-based API security
- **Caching**: Intelligent caching for performance optimization

**Architecture:**
- **SPIDER/**: TypeScript/Node.js web crawler service
- **Search Engines**: Pluggable search engine implementations
- **Docker Support**: Containerized deployment with SearXNG integration

## Plugin Architecture

### Service Communication
- All plugins expose REST APIs that follow standard HTTP conventions
- Authentication uses Bearer tokens for security
- Plugins are stateless and can be horizontally scaled
- JSON-based request/response format

### Deployment Model
- **Docker Containerization**: Each plugin includes Dockerfile for container deployment
- **Independent Scaling**: Plugins can be scaled based on individual resource requirements
- **Environment Configuration**: Plugins use environment variables for configuration
- **Health Checks**: Built-in health monitoring capabilities

### Integration Pattern
1. **Service Discovery**: FastGPT discovers plugins via configuration
2. **API Calls**: FastGPT makes HTTP requests to plugin endpoints
3. **Response Processing**: Plugins return structured data that FastGPT can process
4. **Error Handling**: Comprehensive error reporting and fallback mechanisms

## Plugin Development

### Structure Requirements
Each plugin should follow this structure:
```
plugin-name/
├── README.md           # Plugin-specific documentation
├── Dockerfile          # Container deployment configuration
├── requirements.txt    # Python dependencies (if applicable)
├── package.json        # Node.js dependencies (if applicable)
├── .env.example        # Environment variable template
└── src/               # Source code
    ├── main.py/main.ts # Entry point
    └── routes/         # API route definitions
```

### API Standards
- Use standard HTTP status codes
- Implement Bearer token authentication
- Return JSON responses with consistent structure
- Include comprehensive error messages
- Support health check endpoints

### Configuration Management
- Use environment variables for all configuration
- Provide `.env.example` files for documentation
- Support Docker environment variable injection
- Include sensible defaults

## Deployment

### Docker Deployment (Recommended)
```bash
# Build plugin image
cd plugins/plugin-name
docker build -t fastgpt-plugin-name .

# Run plugin
docker run -d -p PORT:PORT --name plugin-name fastgpt-plugin-name
```

### Development Setup
```bash
# Model plugins (Python)
cd plugins/plugin-name
pip install -r requirements.txt
python main.py

# Web crawler (Node.js)
cd plugins/webcrawler/SPIDER
pnpm install
pnpm dev
```

### Docker Compose
Multiple plugins can be orchestrated using Docker Compose:
```yaml
version: '3.8'
services:
  pdf-marker:
    build: ./plugins/model/pdf-marker
    ports:
      - "7231:7231"
    environment:
      - PROCESSES_PER_GPU=2

  webcrawler:
    build: ./plugins/webcrawler
    ports:
      - "3000:3000"
    environment:
      - ACCESS_TOKEN=your_token
```

## Configuration

### FastGPT Integration
Configure plugins in FastGPT's main configuration:
```json
{
  "plugins": {
    "pdf-marker": {
      "endpoint": "http://pdf-marker:7231",
      "token": "your_access_token"
    },
    "webcrawler": {
      "endpoint": "http://webcrawler:3000",
      "token": "your_access_token"
    }
  }
}
```

### Environment Variables
Common environment variables:
- `PORT`: Service port (default varies by plugin)
- `ACCESS_TOKEN`: Authentication token for API access
- `GPU_ENABLED`: Enable GPU acceleration (for ML plugins)
- `LOG_LEVEL`: Logging verbosity (debug, info, warn, error)

## API Examples

### PDF Processing
```bash
# Convert PDF to Markdown
curl -X POST "http://localhost:7231/v2/parse/file" \
  -H "Authorization: Bearer your_token" \
  -F "file=@document.pdf"
```

### Web Search
```bash
# Search the web
curl "http://localhost:3000/api/search?query=example&pageCount=5&engine=baidu" \
  -H "Authorization: Bearer your_token"
```

### Page Content Extraction
```bash
# Extract page content
curl "http://localhost:3000/api/read?queryUrl=https://example.com" \
  -H "Authorization: Bearer your_token"
```

## Resource Requirements

### GPU Requirements
- **PDF Processing**: Minimum 11GB VRAM (recommended 24GB VRAM)
- **OCR/ML Models**: Varies by model size
- **Speech Processing**: GPU acceleration optional but recommended

### CPU/Memory Requirements
- **Web Crawler**: 2-4 CPU cores, 4-8GB RAM
- **API Services**: 1-2 CPU cores, 2-4GB RAM minimum

## Security Considerations

- All plugins implement token-based authentication
- Network traffic should be secured in production environments
- File uploads are validated and sanitized
- Rate limiting is implemented for API endpoints
- Container isolation limits system access

## Troubleshooting

### Common Issues
1. **Model Download Failures**: Configure HuggingFace mirrors for Chinese environments
2. **GPU Memory**: Adjust `PROCESSES_PER_GPU` based on available VRAM
3. **Network Connectivity**: Ensure FastGPT can reach plugin endpoints
4. **Authentication**: Verify ACCESS_TOKEN configuration matches

### Performance Optimization
- Use GPU acceleration where available
- Implement connection pooling for database operations
- Enable caching for frequently accessed content
- Monitor resource usage and scale accordingly

## Contributing

When contributing new plugins:

1. **Follow Standards**: Adhere to the established plugin structure and API conventions
2. **Documentation**: Provide comprehensive README and API documentation
3. **Testing**: Include unit tests and integration tests
4. **Docker Support**: Include Dockerfile and deployment instructions
5. **Error Handling**: Implement robust error handling and logging

## License

Plugins are licensed under the same terms as the main FastGPT project. Please refer to the root LICENSE file for details.