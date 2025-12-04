# Docker Deployment Guide

## Introduction

This guide provides detailed instructions for deploying FastGPT using Docker and Docker Compose. Docker deployment offers a containerized approach that ensures consistency across environments and simplifies dependency management.

## Prerequisites

### System Requirements
- Docker 20.10+
- Docker Compose 2.0+
- At least 4GB RAM
- At least 10GB disk space
- Linux/macOS/Windows with WSL2

### Optional Components
- Docker Desktop (for GUI management)
- Portainer (for web-based container management)
- Nginx (for reverse proxy)

## Quick Start

### 1. Clone and Prepare

```bash
# Clone the repository
git clone https://github.com/labring/FastGPT.git
cd FastGPT

# Copy environment file
cp projects/app/.env.example projects/app/.env.local

# Edit the environment file
nano projects/app/.env.local
```

### 2. Basic Docker Compose Deployment

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  # Main FastGPT Application
  fastgpt:
    build:
      context: .
      dockerfile: projects/app/Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/fastgpt
      - REDIS_URL=redis://redis:6379
    depends_on:
      - mongodb
      - redis
    volumes:
      - ./data/uploads:/app/uploads
      - ./data/logs:/app/logs
    restart: unless-stopped
    networks:
      - fastgpt-network

  # MongoDB Database
  mongodb:
    image: mongo:6.0
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=fastgpt
    volumes:
      - mongodb_data:/data/db
      - ./deploy/mongo/init.js:/docker-entrypoint-initdb.d/init.js:ro
    restart: unless-stopped
    networks:
      - fastgpt-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_data:/data
    restart: unless-stopped
    networks:
      - fastgpt-network

  # PostgreSQL with pgvector (optional)
  postgres:
    image: pgvector/pgvector:pg15
    ports:
      - "5432:5432"
    environment:
      - POSTGRES_DB=fastgpt
      - POSTGRES_USER=fastgpt
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./deploy/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql:ro
    restart: unless-stopped
    networks:
      - fastgpt-network

  # Milvus Vector Database (optional)
  etcd:
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000
    volumes:
      - etcd_data:/etcd
    command: etcd -advertise-client-urls=http://127.0.0.1:2379 -listen-client-urls http://0.0.0.0:2379 --data-dir /etcd
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 20s
      retries: 3
    networks:
      - fastgpt-network

  minio:
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: ${MINIO_ACCESS_KEY}
      MINIO_SECRET_KEY: ${MINIO_SECRET_KEY}
    ports:
      - "9001:9001"
      - "9000:9000"
    volumes:
      - minio_data:/data
    command: minio server /data --console-address ":9001"
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
      interval: 30s
      timeout: 20s
      retries: 3
    networks:
      - fastgpt-network

  milvus:
    image: milvusdb/milvus:v2.3.0
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    volumes:
      - milvus_data:/var/lib/milvus
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9091/healthz"]
      interval: 30s
      start_period: 90s
      timeout: 20s
      retries: 3
    ports:
      - "19530:19530"
      - "9091:9091"
    depends_on:
      - "etcd"
      - "minio"
    networks:
      - fastgpt-network

volumes:
  mongodb_data:
  redis_data:
  postgres_data:
  etcd_data:
  minio_data:
  milvus_data:

networks:
  fastgpt-network:
    driver: bridge
```

### 3. Environment Variables

Create `.env` file:

```env
# Database Passwords
MONGO_ROOT_PASSWORD=your_mongo_password
REDIS_PASSWORD=your_redis_password
POSTGRES_PASSWORD=your_postgres_password

# MinIO
MINIO_ACCESS_KEY=your_minio_key
MINIO_SECRET_KEY=your_minio_secret

# FastGPT Configuration
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
SECRET_KEY=your_secret_key

# Vector Database (choose one)
VECTOR_DB=pg  # or milvus
```

### 4. Start Services

```bash
# Start all services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f fastgpt
```

## Dockerfile Configuration

### Multi-stage Dockerfile for FastGPT

```dockerfile
# projects/app/Dockerfile
FROM node:18-alpine AS base

# Install dependencies only when needed
FROM base AS deps
WORKDIR /app

# Copy package files
COPY package*.json pnpm-lock.yaml ./
COPY packages/global/package*.json ./packages/global/
COPY packages/service/package*.json ./packages/service/
COPY packages/web/package*.json ./packages/web/
COPY projects/app/package*.json ./projects/app/

# Install pnpm
RUN npm install -g pnpm@8

# Install dependencies
RUN pnpm install --frozen-lockfile

# Build application
FROM base AS builder
WORKDIR /app

# Copy source code
COPY --from=deps /app/node_modules ./node_modules
COPY --from=deps /app/packages ./packages
COPY --from=deps /app/projects ./projects
COPY tsconfig.json ./
COPY pnpm-workspace.yaml ./

# Build packages
RUN pnpm build

# Production image
FROM node:18-alpine AS runner
WORKDIR /app

# Create non-root user
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# Copy built application
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/dist ./dist
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/package.json ./package.json
COPY --from=builder --chown=nextjs:nodejs /app/projects/app/node_modules ./node_modules

# Create necessary directories
RUN mkdir -p /app/uploads /app/logs && chown -R nextjs:nodejs /app

# Set environment
ENV NODE_ENV=production
ENV PORT=3000

# Switch to non-root user
USER nextjs

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "dist/server.js"]
```

### Health Check Script

```javascript
// projects/app/healthcheck.js
const http = require('http');

const options = {
  hostname: 'localhost',
  port: process.env.PORT || 3000,
  path: '/api/health',
  method: 'GET',
  timeout: 2000
};

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

req.on('error', () => {
  process.exit(1);
});

req.on('timeout', () => {
  req.destroy();
  process.exit(1);
});

req.end();
```

## Production Deployment

### 1. Production Docker Compose

Create `docker-compose.prod.yml`:

```yaml
version: '3.8'

services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./deploy/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./deploy/nginx/ssl:/etc/nginx/ssl:ro
      - ./data/uploads:/var/www/uploads:ro
    depends_on:
      - fastgpt
    restart: unless-stopped
    networks:
      - fastgpt-network

  # FastGPT Application (multiple instances)
  fastgpt:
    image: fastgpt/app:latest
    environment:
      - NODE_ENV=production
      - MONGODB_URI=mongodb://mongodb:27017/fastgpt
      - REDIS_URL=redis://redis:6379
      - PORT=3000
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
    depends_on:
      - mongodb
      - redis
    volumes:
      - ./data/uploads:/app/uploads
      - ./data/logs:/app/logs
    restart: unless-stopped
    networks:
      - fastgpt-network
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/api/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # MongoDB with replication
  mongodb:
    image: mongo:6.0
    command: mongod --replSet rs0 --bind_ip_all
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=${MONGO_ROOT_PASSWORD}
      - MONGO_INITDB_DATABASE=fastgpt
    volumes:
      - mongodb_data:/data/db
      - ./deploy/mongo/rs-init.js:/docker-entrypoint-initdb.d/rs-init.js:ro
    deploy:
      replicas: 3
    restart: unless-stopped
    networks:
      - fastgpt-network

  # Redis Cluster
  redis-master:
    image: redis:7-alpine
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    volumes:
      - redis_master_data:/data
    restart: unless-stopped
    networks:
      - fastgpt-network

  redis-slave:
    image: redis:7-alpine
    command: redis-server --slaveof redis-master 6379 --requirepass ${REDIS_PASSWORD}
    depends_on:
      - redis-master
    volumes:
      - redis_slave_data:/data
    restart: unless-stopped
    networks:
      - fastgpt-network

  # Monitoring
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./deploy/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
      - '--storage.tsdb.retention.time=200h'
      - '--web.enable-lifecycle'
    restart: unless-stopped
    networks:
      - fastgpt-network

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
    volumes:
      - grafana_data:/var/lib/grafana
      - ./deploy/grafana/dashboards:/etc/grafana/provisioning/dashboards:ro
      - ./deploy/grafana/datasources:/etc/grafana/provisioning/datasources:ro
    depends_on:
      - prometheus
    restart: unless-stopped
    networks:
      - fastgpt-network

volumes:
  mongodb_data:
  redis_master_data:
  redis_slave_data:
  prometheus_data:
  grafana_data:

networks:
  fastgpt-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
```

### 2. Nginx Configuration

Create `deploy/nginx/nginx.conf`:

```nginx
events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # Logging
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log warn;

    # Performance
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 100M;

    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript
               application/javascript application/xml+rss
               application/json application/xml;

    # Rate limiting
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=upload:10m rate=1r/s;

    # Upstream servers
    upstream fastgpt_backend {
        least_conn;
        server fastgpt_1:3000 max_fails=3 fail_timeout=30s;
        server fastgpt_2:3000 max_fails=3 fail_timeout=30s;
        server fastgpt_3:3000 max_fails=3 fail_timeout=30s;
    }

    # HTTPS server
    server {
        listen 443 ssl http2;
        listen [::]:443 ssl http2;
        server_name your-domain.com;

        # SSL configuration
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;

        # Modern SSL configuration
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;

        # HSTS
        add_header Strict-Transport-Security "max-age=63072000" always;

        # API routes
        location /api/ {
            limit_req zone=api burst=20 nodelay;
            proxy_pass http://fastgpt_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
            proxy_read_timeout 86400;
        }

        # File upload routes
        location /api/upload {
            limit_req zone=upload burst=5 nodelay;
            proxy_pass http://fastgpt_backend;
            proxy_http_version 1.1;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            client_max_body_size 100M;
            proxy_read_timeout 300s;
        }

        # Static files
        location /_next/static/ {
            proxy_pass http://fastgpt_backend;
            add_header Cache-Control "public, max-age=31536000, immutable";
        }

        # Uploads
        location /uploads/ {
            alias /var/www/uploads/;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # Main application
        location / {
            proxy_pass http://fastgpt_backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection 'upgrade';
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_cache_bypass $http_upgrade;
        }
    }

    # HTTP to HTTPS redirect
    server {
        listen 80;
        listen [::]:80;
        server_name your-domain.com;
        return 301 https://$server_name$request_uri;
    }
}
```

## Docker Optimization

### 1. Image Optimization

#### Multi-stage Build
```dockerfile
# Optimize layer caching
FROM node:18-alpine AS deps
WORKDIR /app

# Copy package files first
COPY package.json pnpm-lock.yaml ./

# Install pnpm
RUN npm install -g pnpm@8

# Install dependencies
RUN pnpm install --frozen-lockfile --no-optional
```

#### Base Image Selection
```dockerfile
# Use Alpine for smaller size
FROM node:18-alpine AS base

# Or use distroless for security
FROM gcr.io/distroless/nodejs18-debian11 AS runner
```

### 2. Docker Compose Optimization

#### Resource Limits
```yaml
services:
  fastgpt:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
```

#### Restart Policies
```yaml
services:
  fastgpt:
    restart: unless-stopped
    # or: on-failure:5
    # or: always
```

### 3. Volume Management

#### Named Volumes
```yaml
volumes:
  mongodb_data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: /data/mongodb
```

#### Bind Mounts
```yaml
services:
  fastgpt:
    volumes:
      - type: bind
        source: ./data
        target: /app/data
```

## Container Orchestration

### 1. Docker Swarm

#### Initialize Swarm
```bash
# Initialize swarm
docker swarm init

# Join worker nodes
docker swarm join --token <TOKEN> <MANAGER_IP>:2377

# Deploy stack
docker stack deploy -c docker-compose.yml fastgpt
```

#### Docker Compose for Swarm
```yaml
version: '3.8'

services:
  fastgpt:
    image: fastgpt/app:latest
    deploy:
      mode: replicated
      replicas: 3
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

### 2. Portainer Integration

```yaml
services:
  portainer:
    image: portainer/portainer-ce:latest
    ports:
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    restart: always
```

## Security Configuration

### 1. Container Security

#### Non-root User
```dockerfile
FROM node:18-alpine AS base
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs
```

#### Read-only Filesystem
```yaml
services:
  fastgpt:
    read_only: true
    tmpfs:
      - /tmp
      - /app/logs
```

### 2. Network Security

#### Custom Network
```yaml
networks:
  fastgpt-internal:
    driver: bridge
    internal: true
  fastgpt-external:
    driver: bridge
```

#### Firewall Rules
```bash
# Docker iptables rules
iptables -I DOCKER-USER -i eth0 -p tcp --dport 27017 -j DROP
iptables -I DOCKER-USER -i eth0 -p tcp --dport 6379 -j DROP
```

## Monitoring and Logging

### 1. Container Metrics

#### cAdvisor
```yaml
services:
  cadvisor:
    image: gcr.io/cadvisor/cadvisor:latest
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:ro
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
```

### 2. Log Aggregation

#### ELK Stack
```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    volumes:
      - ./deploy/logstash/pipeline:/usr/share/logstash/pipeline:ro
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
```

## Backup and Recovery

### 1. Data Backup Script

```bash
#!/bin/bash
# backup.sh

BACKUP_DIR="/backup/$(date +%Y%m%d)"
CONTAINER_NAME="fastgpt_mongodb_1"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup MongoDB
docker exec $CONTAINER_NAME mongodump --out /tmp/backup
docker cp $CONTAINER_NAME:/tmp/backup $BACKUP_DIR/mongodb

# Backup uploads
docker cp fastgpt:/app/uploads $BACKUP_DIR/uploads

# Compress backup
tar -czf "$BACKUP_DIR.tar.gz" $BACKUP_DIR
rm -rf $BACKUP_DIR

# Keep only last 7 days
find /backup -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: $BACKUP_DIR.tar.gz"
```

### 2. Automated Backup

```yaml
services:
  backup:
    image: alpine:latest
    volumes:
      - ./backup.sh:/backup.sh:ro
      - ./backups:/backups
      - /var/run/docker.sock:/var/run/docker.sock
    command: >
      sh -c "
        apk add --no-cache docker-cli
        while true; do
          /backup.sh
          sleep 86400
        done
      "
    restart: unless-stopped
```

## Troubleshooting

### 1. Common Issues

#### Container Not Starting
```bash
# Check logs
docker logs fastgpt

# Check container status
docker inspect fastgpt

# Check resource usage
docker stats
```

#### Database Connection Issues
```bash
# Test database connection
docker exec -it mongodb mongo --eval "db.adminCommand('ismaster')"

# Check network
docker network ls
docker network inspect fastgpt_fastgpt-network
```

#### Performance Issues
```bash
# Check container resources
docker stats fastgpt

# Check disk usage
docker system df

# Clean up unused resources
docker system prune -a
```

### 2. Debug Mode

#### Debug Docker Compose
```bash
# Run with debug
docker-compose --debug up

# Check configuration
docker-compose config

# Run specific service
docker-compose run --rm fastgpt sh
```

This Docker deployment guide provides comprehensive instructions for deploying FastGPT in containerized environments, from development to production.