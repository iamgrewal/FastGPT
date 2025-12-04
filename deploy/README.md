# FastGPT Deployment Directory

This directory contains Docker Compose scripts, Helm charts, and deployment configurations for FastGPT. The deployment system supports multiple regions, vector databases, and container registries through a template-based generation approach.

## Overview

The deployment system is designed to:
- Generate region-specific Docker Compose files (China vs Global)
- Support multiple vector databases (PostgreSQL pgvector, Milvus, Zilliz, OceanBase)
- Template-based configuration using variable substitution
- Automated build and deployment pipeline

## Directory Structure

```
deploy/
├── args.json                    # Version and image registry configuration
├── init.mjs                     # Template generation script
├── templates/                   # Template files for Docker Compose
│   ├── docker-compose.dev.yml   # Development environment template
│   ├── docker-compose.prod.yml  # Production environment template
│   └── vector/                  # Vector database service templates
│       ├── pg.txt              # PostgreSQL pgvector configuration
│       ├── milvus.txt          # Milvus vector database configuration
│       └── ob.txt              # OceanBase vector database configuration
├── docker/                      # Generated production Docker Compose files
│   ├── cn/                      # China region configurations
│   │   ├── docker-compose.pg.yml
│   │   ├── docker-compose.milvus.yml
│   │   ├── docker-compose.zilliz.yml
│   │   └── docker-compose.oceanbase.yml
│   └── global/                  # Global region configurations
│       ├── docker-compose.pg.yml
│       ├── docker-compose.milvus.yml
│       ├── docker-compose.ziliiz.yml
│       └── docker-compose.oceanbase.yml
├── dev/                         # Generated development Docker Compose files
│   ├── docker-compose.cn.yml
│   └── docker-compose.yml
└── helm/                        # Kubernetes Helm charts
    └── fastgpt/
```

## Configuration Management

### args.json Structure

The `args.json` file contains version numbers and container image registry configurations:

```json
{
  "tags": {
    "fastgpt": "v4.14.3",
    "fastgpt-sandbox": "v4.14.3",
    "fastgpt-mcp_server": "v4.14.3",
    "fastgpt-plugin": "v0.3.3",
    "aiproxy": "v0.3.2",
    // ... additional versions
  },
  "images": {
    "cn": {
      "fastgpt": "registry.cn-hangzhou.aliyuncs.com/fastgpt/fastgpt",
      "fastgpt-plugin": "registry.cn-hangzhou.aliyuncs.com/fastgpt/fastgpt-plugin",
      // ... China registry images
    },
    "global": {
      "fastgpt": "ghcr.io/labring/fastgpt",
      "fastgpt-plugin": "ghcr.io/labring/fastgpt-plugin",
      // ... Global registry images
    }
  }
}
```

## Template System

The deployment system uses a template-based approach with variable substitution. Templates use `${{variable.property}}` syntax that gets replaced during generation.

### Variable Substitution Pattern

- `${{service.image}}` → Container image URL for the specified region
- `${{service.tag}}` → Container version tag
- `${{vec.config}}` → Vector database configuration
- `${{vec.db}}` → Vector database service definition

## Updating Docker Compose Scripts

### Normal Updates (Version Changes Only)

For simple version updates without changing services:

1. Update version numbers in `args.json`:
   ```json
   {
     "tags": {
       "fastgpt": "v4.14.4",  // Updated version
       "fastgpt-sandbox": "v4.14.4"
     }
   }
   ```

2. Generate new Docker Compose files:
   ```bash
   cd FastGPT
   pnpm run gen:deploy
   ```

### Adding New Services

To add a new service (e.g., `example` service):

1. **Add service to Services Enum** in `init.mjs`:
   ```javascript
   const Services = {
     fastgpt: 'fastgpt',
     fastgptPlugin: 'fastgpt-plugin',
     // ... existing services
     fastgptExample: 'fastgpt-example'  // Add new service
   };
   ```

2. **Add configuration to args.json**:
   ```json
   {
     "tags": {
       "fastgpt-example": "v1.0.0"
     },
     "images": {
       "cn": {
         "fastgpt-example": "registry.cn-hangzhou.aliyuncs.com/fastgpt/fastgpt-example"
       },
       "global": {
         "fastgpt-example": "ghcr.io/labring/fastgpt-example"
       }
     }
   }
   ```
   **Important**: The key in `args.json` must match the value in the Services enum.

3. **Update template files** (`templates/docker-compose.[dev|prod].yml`):
   ```yaml
   services:
     example:
       image: ${{example.image}}:${{example.tag}}
       container_name: example
       # ... rest of service configuration
   ```

### Adding New Vector Databases

To add a new vector database (e.g., `exampleDB`):

1. **Create vector service template** at `templates/vector/exampleDB.txt`:
   ```yaml
   # templates/vector/exampleDB.txt
   vectorDB:
     image: ${{exampleDB.image}}:${{exampleDB.tag}}
     container_name: exampleDB
     restart: always
     networks:
       - fastgpt
     # ... vector database specific configuration
   ```
   **Important**:
   - Maintain proper YAML indentation
   - Use `${{exampleDB.image}}:${{exampleDB.tag}}` syntax
   - Service name must be `vectorDB`

2. **Add configuration to args.json**:
   ```json
   {
     "tags": {
       "exampleDB": "v2.0.0"
     },
     "images": {
       "cn": {
         "exampleDB": "registry.cn-hangzhou.aliyuncs.com/fastgpt/exampleDB"
       },
       "global": {
         "exampleDB": "exampleDB/exampleDB"
       }
     }
   }
   ```

3. **Add to VectorEnum** in `init.mjs`:
   ```javascript
   const VectorEnum = {
     pg: 'pg',
     milvus: 'milvus',
     zilliz: 'zilliz',
     ob: 'ob',
     exampleDB: 'exampleDB'  // Add new vector database
   };
   ```

4. **Add vector configuration** in `init.mjs`:
   ```javascript
   const vector = {
     exampleDB: {
       db: '', // Will be loaded from template file
       config: `\
   EXAMPLE_URL: exampledb://connection_string
   EXAMPLE_TOKEN: authentication_token
       `,
       extra: '' // Additional configuration (e.g., init scripts)
     }
   };
   ```

5. **Read vector template** in the vector loading section:
   ```javascript
   {
     // read in Vectors
     const exampleDB = fs.readFileSync(path.join(process.cwd(), 'templates', 'vector', 'exampleDB.txt'));
     vector.exampleDB.db = String(exampleDB);
   }
   ```

6. **Add to file generation** in `generateProdFile()`:
   ```javascript
   await Promise.all([
     // ... existing vector databases
     fs.promises.writeFile(
       path.join(process.cwd(), 'docker', 'cn', 'docker-compose.exampleDB.yml'),
       replace(template, 'cn', VectorEnum.exampleDB)
     ),
     fs.promises.writeFile(
       path.join(process.cwd(), 'docker', 'global', 'docker-compose.exampleDB.yml'),
       replace(template, 'global', VectorEnum.exampleDB)
     )
   ]);
   ```

## Vector Database Configuration

### Supported Vector Databases

1. **PostgreSQL pgvector** (`pg`)
   - Uses `pgvector` extension for vector operations
   - Connection URL format: `postgresql://username:password@host:port/database`

2. **Milvus** (`milvus`)
   - Standalone Milvus deployment
   - HTTP API on port 19530
   - Optional token authentication

3. **Zilliz Cloud** (`zilliz`)
   - Managed Milvus-compatible service
   - Requires cloud address and authentication token

4. **OceanBase** (`ob`)
   - MySQL-compatible database with vector extensions
   - Requires additional configuration for vector memory limits

### Vector Database Template Examples

#### PostgreSQL Template (`templates/vector/pg.txt`)
```yaml
vectorDB:
  image: ${{pg.image}}:${{pg.tag}}
  container_name: pg
  restart: always
  networks:
    - fastgpt
  environment:
    # These settings only take effect on first run
    # To modify, delete persistent data and restart
    - POSTGRES_USER=username
    - POSTGRES_PASSWORD=password
    - POSTGRES_DB=postgres
  volumes:
    - ./pg/data:/var/lib/postgresql/data
  healthcheck:
    test: ['CMD', 'pg_isready', '-U', 'username', '-d', 'postgres']
    interval: 5s
    timeout: 5s
    retries: 10
```

## YAML Anchors and References

The deployment configuration uses YAML anchors and references for DRY (Don't Repeat Yourself) configuration.

### Anchor Definition

Use `&` to define an anchor:
```yaml
x-share-config: &x-share-config 'I am the config content'

x-share-config-list: &x-share-config-list
  key1: value
  key2: value
  key3:
    nested_key: nested_value
```

### Reference Usage

Use `*` to reference an anchor:
```yaml
service_one:
  config: *x-share-config

service_two:
  config_list: *x-share-config-list

# Can also merge with additional properties
service_three:
  <<: *x-share-config-list
  key4: additional_value  # Added to merged config
```

## Build and Generation Process

### Prerequisites

- Node.js >= 18.16.0
- pnpm package manager
- Docker (for running containers)

### Generation Commands

1. **Generate deployment files** (run from FastGPT root):
   ```bash
   pnpm run gen:deploy
   ```

2. **Manual generation** (from deploy directory):
   ```bash
   cd deploy
   node init.mjs
   ```

### Generation Process

The `init.mjs` script performs the following operations:

1. **Load Configuration**: Reads `args.json` for versions and image registries
2. **Load Templates**: Reads vector database templates from `templates/vector/`
3. **Generate Development Files**: Creates development Docker Compose files with exposed ports
4. **Generate Production Files**: Creates production Docker Compose files for each vector database
5. **Copy to Documentation**: Copies generated files to documentation directory

### Generated Files

**Development Files**:
- `dev/docker-compose.yml` (Global region)
- `dev/docker-compose.cn.yml` (China region)

**Production Files**:
- `docker/cn/docker-compose.{vector}.yml` (China region)
- `docker/global/docker-compose.{vector}.yml` (Global region)

## Development vs Production Configuration

### Development Environment
- Exposes all service ports to host
- Includes only essential services
- Uses pgvector as default vector database
- Suitable for local development and testing

### Production Environment
- Exposes only necessary ports
- Complete service stack including FastGPT
- Multiple vector database options
- Optimized for deployment with proper networking

## Container Images and Registries

### China Region (`cn`)
Uses Alibaba Cloud registry for faster access in China:
```bash
registry.cn-hangzhou.aliyuncs.com/fastgpt/{service}
```

### Global Region (`global`)
Uses GitHub Container Registry for global access:
```bash
ghcr.io/labring/{service}
```

## Network Configuration

All services use a shared Docker network named `fastgpt` for inter-service communication. The network is defined in the Docker Compose files and allows services to communicate using container names as hostnames.

## Data Persistence

Services that require data persistence use volume mounts:
- **MongoDB**: `./mongo/data:/data/db`
- **PostgreSQL**: `./pg/data:/var/lib/postgresql/data`
- **Milvus**: Various volumes for etcd, storage, and logs

## Health Checks

Database services include health checks to ensure they are ready before other services start:
- **PostgreSQL**: Uses `pg_isready` command
- **MongoDB**: Configured replica set health monitoring

## Troubleshooting

### Common Issues

1. **Template Generation Fails**:
   - Check `args.json` syntax
   - Ensure all required services are defined
   - Verify template file paths exist

2. **Container Startup Issues**:
   - Check volume permissions
   - Verify port availability
   - Review environment variables

3. **Vector Database Connection Issues**:
   - Verify network connectivity between containers
   - Check authentication credentials
   - Ensure proper configuration format

### Debug Commands

```bash
# Check generated files
ls -la docker/cn/ docker/global/ dev/

# Validate Docker Compose syntax
docker-compose -f dev/docker-compose.yml config

# Test service connectivity
docker-compose -f dev/docker-compose.yml ps
```

## Contributing

When modifying deployment configurations:

1. Test changes locally with development files first
2. Ensure both China and Global regions are properly configured
3. Update documentation for any new services or configurations
4. Validate generated Docker Compose files before committing

## License

This deployment configuration follows the same license as the FastGPT project.