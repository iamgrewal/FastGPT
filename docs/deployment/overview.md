# Deployment Overview

## Introduction

FastGPT offers multiple deployment options to suit different needs, from development and testing to production environments. This guide provides an overview of all available deployment strategies and helps you choose the right one for your use case.

## Deployment Options

### 1. Development Deployment
- **Purpose**: Local development and testing
- **Complexity**: Low
- **Resources**: Minimal
- **When to use**: During development, feature testing, quick prototyping

### 2. Docker Deployment
- **Purpose**: Containerized deployment for staging/production
- **Complexity**: Medium
- **Resources**: Moderate
- **When to use**: Staging environments, small production deployments

### 3. Kubernetes Deployment
- **Purpose**: Scalable, production-ready deployment
- **Complexity**: High
- **Resources**: Significant
- **When to use**: Large production deployments, multi-tenant SaaS

### 4. Cloud Platform Deployment
- **Purpose**: Managed cloud deployment
- **Complexity**: Low to Medium
- **Resources**: Varies by provider
- **When to use**: When using cloud-specific services

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Load Balancer                            │
│                    (Nginx/CloudFlare)                       │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                    Application Layer                        │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │   Web App   │  │   API GW    │  │   Plugin Manager    │ │
│  │  (NextJS)   │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                   Service Layer                             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ Workflow    │  │ AI Services │  │   Code Sandbox      │ │
│  │ Engine      │  │             │  │                     │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────┬───────────────────────────────────────────┘
                  │
┌─────────────────┴───────────────────────────────────────────┐
│                    Data Layer                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │  MongoDB    │  │ PostgreSQL  │  │    Vector DB        │ │
│  │             │  │ + pgvector  │  │  (Milvus/PG)        │ │
│  └─────────────┘  └─────────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
```

## Choosing the Right Deployment

### Development Environment
Choose **Local Development** if you:
- Are developing features or fixes
- Need hot reload and debugging
- Want to test quickly
- Have limited resources

### Staging Environment
Choose **Docker Compose** if you:
- Need to test before production
- Want production-like environment
- Need isolated testing
- Have moderate resources

### Production Environment
Choose based on your scale:

#### Small Production (< 1000 users)
- **Single Docker Instance**
- Managed Database (MongoDB Atlas, Supabase)
- CDN for static assets
- Basic monitoring

#### Medium Production (1000-10000 users)
- **Docker with Load Balancer**
- Self-hosted databases with replication
- Redis for caching
- Comprehensive monitoring

#### Large Production (> 10000 users)
- **Kubernetes Cluster**
- Multi-AZ deployment
- Database sharding
- Advanced monitoring and autoscaling

## Infrastructure Requirements

### Minimum Requirements

#### Development
- CPU: 2 cores
- RAM: 4GB
- Storage: 20GB SSD
- Network: Basic broadband

#### Production (Small)
- CPU: 4 cores
- RAM: 8GB
- Storage: 100GB SSD
- Network: 100Mbps
- Database: MongoDB 5.0+

#### Production (Large)
- CPU: 16+ cores
- RAM: 32GB+
- Storage: 500GB+ SSD
- Network: 1Gbps+
- Database: Clustered MongoDB
- Cache: Redis Cluster

### Recommended Cloud Providers

#### Public Cloud
- **AWS**: Full-featured, excellent reliability
- **Google Cloud**: Strong AI/ML integration
- **Azure**: Enterprise features
- **DigitalOcean**: Developer-friendly, cost-effective

#### Managed Services
- **MongoDB Atlas**: Managed MongoDB
- **Supabase**: PostgreSQL with vector support
- **Zilliz**: Managed Milvus
- **Redis Cloud**: Managed Redis

## Deployment Checklist

### Pre-Deployment

#### Security
- [ ] Change all default passwords
- [ ] Set up SSL/TLS certificates
- [ ] Configure firewall rules
- [ ] Set up API rate limiting
- [ ] Configure CORS properly
- [ ] Enable security headers

#### Performance
- [ ] Configure database indexes
- [ ] Set up Redis caching
- [ ] Enable gzip compression
- [ ] Configure CDN for assets
- [ ] Optimize images and static files

#### Monitoring
- [ ] Set up application monitoring
- [ ] Configure error tracking
- [ ] Set up log aggregation
- [ ] Configure health checks
- [ ] Set up alerts

#### Backups
- [ ] Configure database backups
- [ ] Set up file storage backup
- [ ] Document recovery process
- [ ] Test restore procedures

### Post-Deployment

#### Verification
- [ ] Test all user flows
- [ ] Verify API endpoints
- [ ] Check authentication flow
- [ ] Test file uploads
- [ ] Verify AI model integration

#### Monitoring Setup
- [ ] Monitor response times
- [ ] Track error rates
- [ ] Monitor resource usage
- [ ] Set up performance alerts
- [ ] Create dashboards

## Deployment Strategies

### 1. Blue-Green Deployment

```mermaid
graph LR
    A[Load Balancer] --> B[Blue Environment]
    A --> C[Green Environment]
    D[New Version] --> C
    C --> E[Health Check]
    E --> F[Switch Traffic]
```

**Pros**: Zero downtime, instant rollback
**Cons**: Double infrastructure cost
**Best for**: Critical production systems

### 2. Rolling Deployment

```mermaid
graph LR
    A[Load Balancer] --> B[Instance 1]
    A --> C[Instance 2]
    A --> D[Instance 3]

    subgraph "Update Process"
        E[Update Instance 1] --> F[Update Instance 2]
        F --> G[Update Instance 3]
    end
```

**Pros**: Resource efficient, gradual rollout
**Cons**: Slower deployment, version mixing
**Best for**: Stateless applications

### 3. Canary Deployment

```mermaid
graph LR
    A[Load Balancer] --> B[90% Old Version]
    A --> C[10% New Version]

    D[Monitor] --> E{Healthy?}
    E -->|Yes| F[Gradually Increase]
    E -->|No| G[Rollback]
```

**Pros**: Risk mitigation, gradual testing
**Cons**: Complex infrastructure
**Best for**: Risky releases, large user bases

## Environment Configuration

### Development (`.env.development`)
```env
# Database
MONGODB_URI=mongodb://localhost:27017/fastgpt-dev
REDIS_URL=redis://localhost:6379

# AI Services
OPENAI_API_KEY=your_dev_key
ANTHROPIC_API_KEY=your_dev_key

# Debug
LOG_LEVEL=debug
DEBUG=true
HOT_RELOAD=true
```

### Staging (`.env.staging`)
```env
# Database
MONGODB_URI=mongodb://staging-db:27017/fastgpt-staging
REDIS_URL=redis://staging-redis:6379

# AI Services
OPENAI_API_KEY=your_staging_key
ANTHROPIC_API_KEY=your_staging_key

# Security
SECRET_KEY=your_staging_secret
JWT_SECRET=your_jwt_secret

# Monitoring
LOG_LEVEL=info
METRICS_ENABLED=true
```

### Production (`.env.production`)
```env
# Database
MONGODB_URI=mongodb://prod-replica:27017/fastgpt-prod
REDIS_URL=redis://prod-redis-cluster:6379

# AI Services
OPENAI_API_KEY=your_prod_key
ANTHROPIC_API_KEY=your_prod_key

# Security
SECRET_KEY=your_prod_secret
JWT_SECRET=your_jwt_secret
CORS_ORIGIN=https://yourdomain.com

# Performance
LOG_LEVEL=warn
CACHE_TTL=3600
RATE_LIMIT=100

# Monitoring
METRICS_ENABLED=true
ERROR_TRACKING_ENABLED=true
```

## Scaling Considerations

### Horizontal Scaling

#### Application Layer
```yaml
# Kubernetes Horizontal Pod Autoscaler
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: fastgpt-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: fastgpt-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

#### Database Scaling
- **Read Replicas**: Distribute read queries
- **Sharding**: Split data across servers
- **Connection Pooling**: Efficient connection use

### Vertical Scaling

#### Resource Optimization
- **Node Optimization**: Optimize Node.js event loop
- **Memory Management**: Monitor and optimize memory usage
- **CPU Profiling**: Identify and fix CPU bottlenecks

## Monitoring and Observability

### Key Metrics

#### Application Metrics
- Response time (p95, p99)
- Request rate
- Error rate
- Active users
- Workflow execution time

#### Infrastructure Metrics
- CPU usage
- Memory usage
- Disk I/O
- Network traffic
- Database performance

#### Business Metrics
- Daily active users
- Chat messages per day
- Workflow runs
- API calls
- User satisfaction

### Monitoring Stack

#### Prometheus + Grafana
```yaml
# Prometheus configuration
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'fastgpt'
    static_configs:
      - targets: ['app:3000']
    metrics_path: '/metrics'
    scrape_interval: 5s
```

#### ELK Stack
```yaml
# Logstash configuration
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][service] == "fastgpt" {
    json {
      source => "message"
    }

    date {
      match => [ "timestamp", "ISO8601" ]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "fastgpt-%{+YYYY.MM.dd}"
  }
}
```

## Deployment Automation

### CI/CD Pipeline

#### GitHub Actions
```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run tests
        run: pnpm test

  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build Docker image
        run: |
          docker build -t fastgpt:${{ github.sha }} .
          docker push ${{ secrets.REGISTRY_URL }}/fastgpt:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Kubernetes
        run: |
          kubectl set image deployment/fastgpt app=${{ secrets.REGISTRY_URL }}/fastgpt:${{ github.sha }}
          kubectl rollout status deployment/fastgpt
```

### GitOps with ArgoCD

#### Application Manifest
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fastgpt
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/fastgpt-k8s
    targetRevision: HEAD
    path: manifests
  destination:
    server: https://kubernetes.default.svc
    namespace: fastgpt
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

## Security Best Practices

### Network Security
- Use VPCs or network segments
- Implement firewalls
- Use private networks for internal services
- Enable DDoS protection

### Application Security
- Validate all inputs
- Use prepared statements
- Implement rate limiting
- Use Web Application Firewall (WAF)

### Data Security
- Encrypt data at rest
- Encrypt data in transit
- Use secret management
- Regular security audits

### Compliance
- GDPR compliance
- SOC 2 compliance
- HIPAA compliance (if applicable)
- Regular penetration testing

## Troubleshooting

### Common Issues

#### Database Connection
```bash
# Check MongoDB connection
mongo --eval "db.adminCommand('ismaster')"

# Check PostgreSQL connection
psql -h localhost -U postgres -c "SELECT version();"
```

#### Redis Issues
```bash
# Check Redis status
redis-cli ping

# Monitor Redis
redis-cli monitor
```

#### Application Logs
```bash
# View application logs
docker logs fastgpt-app

# View recent logs
tail -f /var/log/fastgpt/app.log
```

### Performance Issues

#### Database Optimization
```javascript
// Find slow queries
db.setProfilingLevel(2);
db.system.profile.find().sort({millis: -1}).limit(5);

// Check indexes
db.collection.getIndexes();
```

#### Memory Issues
```bash
# Check Node.js memory
node --inspect index.js

# Profile memory usage
node --prof index.js
node --prof-process isolate-*.log > processed.txt
```

## Disaster Recovery

### Backup Strategy

#### Database Backups
```bash
# MongoDB backup
mongodump --uri="mongodb://user:pass@host:27017/db" --out=/backup/$(date +%Y%m%d)

# Automated backup script
#!/bin/bash
BACKUP_DIR="/backup/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR
mongodump --uri="$MONGODB_URI" --out=$BACKUP_DIR
tar -czf "$BACKUP_DIR.tar.gz" $BACKUP_DIR
rm -rf $BACKUP_DIR
```

#### File Backups
```bash
# Backup user uploads
rsync -av --delete /app/uploads/ /backup/uploads/

# Backup to cloud storage
aws s3 sync /app/uploads/ s3://backup-bucket/uploads/
```

### Recovery Process

#### Database Recovery
```bash
# Restore from backup
mongorestore --uri="mongodb://user:pass@host:27017/db" /backup/db/

# Point-in-time recovery (if using oplog)
mongorestore --uri="$MONGODB_URI" --oplogReplay /backup/
```

#### Application Recovery
```bash
# Deploy previous version
kubectl rollout undo deployment/fastgpt

# Or deploy specific version
kubectl set image deployment/fastgpt app=fastgpt:v1.2.3
```

This deployment overview provides the foundation for understanding FastGPT deployment options. For specific deployment instructions, refer to the detailed guides for each deployment method.