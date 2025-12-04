# Security Architecture Overview

## Introduction

FastGPT implements a comprehensive security architecture designed to protect data, ensure privacy, and maintain system integrity. This document provides an overview of the security model, threat landscape, and protective measures implemented throughout the platform.

## Security Principles

### 1. Defense in Depth
FastGPT employs multiple layers of security controls:
- **Network Security**: Firewalls, VPNs, and network segmentation
- **Application Security**: Authentication, authorization, and input validation
- **Data Security**: Encryption at rest and in transit
- **Infrastructure Security**: Secure configurations and monitoring

### 2. Zero Trust Architecture
- Never trust, always verify
- Principle of least privilege
- Micro-segmentation
- Continuous monitoring and validation

### 3. Privacy by Design
- Data minimization
- Purpose limitation
- End-to-end encryption
- GDPR compliance

## Threat Model

### 1. External Threats

#### Malicious Actors
- Unauthorized API access
- Data exfiltration
- Service disruption
- Injection attacks

#### Automated Attacks
- Brute force attempts
- DDoS attacks
- Vulnerability scanning
- Bot attacks

### 2. Internal Threats

#### Insider Threats
- Privilege escalation
- Data theft
- Unauthorized access
- Configuration errors

#### Accidental Threats
- Misconfigurations
- Human error
- Data leaks
- Credential exposure

### 3. Supply Chain Threats

#### Third-party Dependencies
- Vulnerable dependencies
- Compromised packages
- Supply chain attacks
- License violations

## Security Controls

### 1. Authentication and Authorization

#### Multi-Factor Authentication (MFA)
```typescript
interface MFAConfig {
  methods: ('totp' | 'sms' | 'email')[];
  required: boolean;
  gracePeriod: number;
  backupCodes: string[];
}

// MFA Implementation
class MFAService {
  async enableMFA(userId: string, method: string): Promise<void> {
    // Generate TOTP secret
    const secret = this.generateTOTPSecret();

    // Store encrypted secret
    await this.storeEncryptedSecret(userId, secret);

    // Send QR code to user
    await this.sendQRCode(userId, secret);
  }
}
```

#### Role-Based Access Control (RBAC)
```typescript
// Role definitions
const roles = {
  admin: {
    permissions: ['*'],
    scope: 'all'
  },
  developer: {
    permissions: ['app:*', 'workflow:*', 'dataset:read'],
    scope: 'team'
  },
  user: {
    permissions: ['app:read', 'workflow:execute'],
    scope: 'own'
  }
};

// Permission checking
const checkPermission = (user: User, resource: string, action: string): boolean => {
  const userPermissions = user.role.permissions;
  return userPermissions.includes('*') ||
         userPermissions.includes(`${resource}:*`) ||
         userPermissions.includes(`${resource}:${action}`);
};
```

### 2. Data Protection

#### Encryption at Rest
```typescript
// Database encryption
const encryptionConfig = {
  mongodb: {
    encryptionAtRest: true,
    keyManagement: 'customer-managed',
    rotationInterval: 90 // days
  },
  postgresql: {
    transparentDataEncryption: true,
    columnLevelEncryption: ['ssn', 'email', 'phone']
  },
  fileStorage: {
    serverSideEncryption: 'AES-256',
    clientSideEncryption: 'optional'
  }
};

// Application-level encryption
class EncryptionService {
  private key: Buffer;

  constructor(key: string) {
    this.key = Buffer.from(key, 'base64');
  }

  encrypt(data: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipher('aes-256-gcm', this.key);
    cipher.setAAD(Buffer.from('fastgpt'));

    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');

    const authTag = cipher.getAuthTag();
    return iv.toString('hex') + ':' + authTag.toString('hex') + ':' + encrypted;
  }
}
```

#### Encryption in Transit
```yaml
# HTTPS configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-tls-config
data:
  nginx.conf: |
    server {
      listen 443 ssl http2;
      ssl_certificate /etc/ssl/certs/tls.crt;
      ssl_certificate_key /etc/ssl/private/tls.key;

      # Modern SSL configuration
      ssl_protocols TLSv1.2 TLSv1.3;
      ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
      ssl_prefer_server_ciphers off;

      # HSTS
      add_header Strict-Transport-Security "max-age=63072000" always;

      # Certificate pinning
      add_header Public-Key-Pins 'pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000';
    }
```

### 3. Input Validation and Sanitization

#### Request Validation
```typescript
// Input validation schema
const createAppSchema = Joi.object({
  name: Joi.string().min(1).max(100).required(),
  description: Joi.string().max(1000).optional(),
  config: Joi.object({
    model: Joi.string().valid('gpt-3.5-turbo', 'gpt-4').required(),
    temperature: Joi.number().min(0).max(2).required(),
    maxTokens: Joi.number().min(1).max(4000).required()
  }).required()
});

// Validation middleware
const validateInput = (schema: Joi.Schema) => {
  return (req: Request, res: Response, next: NextFunction) => {
    const { error, value } = schema.validate(req.body);

    if (error) {
      return res.status(400).json({
        error: 'Validation Error',
        details: error.details.map(d => ({
          field: d.path.join('.'),
          message: d.message
        }))
      });
    }

    req.body = value;
    next();
  };
};
```

#### XSS Prevention
```typescript
// Sanitization library
import DOMPurify from 'dompurify';
import { JSDOM } from 'jsdom';

// Create DOMPurify instance
const window = new JSDOM('').window;
const purify = DOMPurify(window);

// Sanitize user input
const sanitizeInput = (input: string): string => {
  return purify.sanitize(input, {
    ALLOWED_TAGS: ['p', 'strong', 'em', 'u'],
    ALLOWED_ATTR: [],
    KEEP_CONTENT: true
  });
};

// Content Security Policy
const cspHeader = "default-src 'self'; " +
                 "script-src 'self' 'unsafe-inline'; " +
                 "style-src 'self' 'unsafe-inline'; " +
                 "img-src 'self' data: https:; " +
                 "font-src 'self' data:; " +
                 "connect-src 'self' https://api.openai.com; " +
                 "frame-ancestors 'none';";
```

### 4. API Security

#### Rate Limiting
```typescript
// Redis-based rate limiting
import Redis from 'ioredis';

class RateLimiter {
  private redis: Redis;

  constructor(redis: Redis) {
    this.redis = redis;
  }

  async isAllowed(
    key: string,
    limit: number,
    window: number
  ): Promise<{ allowed: boolean; remaining: number; resetTime: number }> {
    const now = Date.now();
    const windowStart = now - window;
    const redisKey = `rate_limit:${key}`;

    // Remove old entries
    await this.redis.zremrangebyscore(redisKey, 0, windowStart);

    // Count current requests
    const current = await this.redis.zcard(redisKey);

    if (current >= limit) {
      const oldestRequest = await this.redis.zrange(redisKey, 0, 0, 'WITHSCORES');
      const resetTime = parseInt(oldestRequest[0][1]) + window;

      return {
        allowed: false,
        remaining: 0,
        resetTime
      };
    }

    // Add current request
    await this.redis.zadd(redisKey, now, now.toString());
    await this.redis.expire(redisKey, Math.ceil(window / 1000));

    return {
      allowed: true,
      remaining: limit - current - 1,
      resetTime: now + window
    };
  }
}
```

#### API Key Security
```typescript
// API key generation and management
class APIKeyService {
  private readonly keyPrefix = 'fastgpt_';
  private readonly keyLength = 32;

  generateAPIKey(type: 'live' | 'test'): string {
    const keyBytes = crypto.randomBytes(this.keyLength);
    const key = keyBytes.toString('base64')
      .replace(/[+/=]/g, '')
      .substring(0, this.keyLength);

    return `${this.keyPrefix}${type}_${key}`;
  }

  async hashAPIKey(key: string): Promise<string> {
    const salt = crypto.randomBytes(16);
    const hash = crypto.pbkdf2Sync(key, salt, 100000, 64, 'sha512');
    return salt.toString('hex') + ':' + hash.toString('hex');
  }

  async verifyAPIKey(key: string, hashedKey: string): Promise<boolean> {
    const [salt, hash] = hashedKey.split(':');
    const verifyHash = crypto.pbkdf2Sync(key, Buffer.from(salt, 'hex'), 100000, 64, 'sha512');
    return hash === verifyHash.toString('hex');
  }
}
```

### 5. Infrastructure Security

#### Network Security
```yaml
# Network policies for Kubernetes
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: fastgpt-network-policy
spec:
  podSelector:
    matchLabels:
      app: fastgpt
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: nginx
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: mongodb
    ports:
    - protocol: TCP
      port: 27017
  - to: []
    ports:
    - protocol: TCP
      port: 443
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

#### Container Security
```dockerfile
# Secure base image
FROM node:18-alpine AS base

# Create non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 nextjs

# Security scanner integration
RUN apk add --no-cache ca-certificates && \
    update-ca-certificates

# Remove unnecessary packages
RUN apk del --purge \
    && rm -rf /var/cache/apk/* \
    && rm -rf /tmp/*

# Run as non-root user
USER nextjs

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js
```

### 6. Monitoring and Logging

#### Security Monitoring
```typescript
// Security event monitoring
class SecurityMonitor {
  private eventLogger: Logger;
  private alertService: AlertService;

  async trackSecurityEvent(event: SecurityEvent): Promise<void> {
    // Log event
    await this.eventLogger.info('Security Event', {
      type: event.type,
      userId: event.userId,
      ip: event.ip,
      userAgent: event.userAgent,
      timestamp: new Date(),
      severity: event.severity
    });

    // Check for alert conditions
    if (await this.shouldAlert(event)) {
      await this.alertService.send({
        level: 'HIGH',
        message: `Security event detected: ${event.type}`,
        details: event
      });
    }
  }

  private async shouldAlert(event: SecurityEvent): Promise<boolean> {
    // Multiple failed logtries
    if (event.type === 'AUTH_FAILED' && event.count > 5) {
      return true;
    }

    // Suspicious IP activity
    if (event.type === 'SUSPICIOUS_IP') {
      return true;
    }

    // Privilege escalation attempt
    if (event.type === 'PRIVILEGE_ESCALATION') {
      return true;
    }

    return false;
  }
}
```

#### Audit Logging
```typescript
// Audit logger
class AuditLogger {
  async logAuditEvent(
    userId: string,
    action: string,
    resource: string,
    result: 'SUCCESS' | 'FAILURE',
    details?: Record<string, any>
  ): Promise<void> {
    const auditEvent = {
      userId,
      action,
      resource,
      result,
      details,
      timestamp: new Date().toISOString(),
      requestId: this.getRequestId()
    };

    // Write to immutable audit log
    await this.writeAuditLog(auditEvent);

    // Send to SIEM
    await this.sendToSIEM(auditEvent);
  }
}
```

## Compliance and Regulations

### 1. GDPR Compliance

#### Data Subject Rights
```typescript
// GDPR implementation
class GDPRService {
  async exportUserData(userId: string): Promise<UserDataExport> {
    // Collect all user data
    const userData = await Promise.all([
      this.getUserProfile(userId),
      this.getUserApps(userId),
      this.getUserWorkflows(userId),
      this.getUserChatHistory(userId),
      this.getUserActivityLogs(userId)
    ]);

    // Format for export
    return {
      personalData: userData[0],
      applications: userData[1],
      workflows: userData[2],
      conversations: userData[3],
      activity: userData[4],
      exportDate: new Date()
    };
  }

  async deleteUserData(userId: string): Promise<void> {
    // Anonymize instead of delete for audit purposes
    await this.anonymizeUserData(userId);

    // Delete from live systems
    await this.deleteFromProduction(userId);

    // Log deletion
    await this.auditLogger.log('DATA_DELETION', { userId });
  }
}
```

### 2. SOC 2 Compliance

#### Security Controls
```typescript
// SOC 2 Type II controls
class SOC2Controls {
  // Access Control
  async implementAccessControl(): Promise<void> {
    // Regular access reviews
    await this.scheduleAccessReviews();

    // Automated provisioning/deprovisioning
    await this.setupProvisioningAutomation();

    // Privileged access management
    await this.configurePAM();
  }

  // Change Management
  async implementChangeManagement(): Promise<void> {
    // Change approval workflow
    await this.setupChangeApproval();

    // Rollback procedures
    await this.documentRollbackProcedures();

    // Change documentation
    await this.implementChangeDocumentation();
  }

  // Incident Response
  async implementIncidentResponse(): Promise<void> {
    // Incident response plan
    await this.createIncidentResponsePlan();

    // Incident detection
    await this.setupIncidentDetection();

    // Communication procedures
    await this.defineCommunicationProcedures();
  }
}
```

## Security Testing

### 1. Vulnerability Scanning

```yaml
# CI/CD security scanning
name: Security Scan

on: [push, pull_request]

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Dependency scanning
      - name: Run Snyk
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      # Container scanning
      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'fastgpt:latest'
          format: 'sarif'
          output: 'trivy-results.sarif'

      # Static analysis
      - name: Run CodeQL
        uses: github/codeql-action/init@v2
        with:
          languages: typescript
```

### 2. Penetration Testing

```typescript
// Penetration test automation
class PenTestRunner {
  async runAuthenticationTests(): Promise<TestResult[]> {
    const tests = [
      this.testWeakPasswords(),
      this.testSessionFixation(),
      this.testCSRF(),
      this.testJWTSecurity(),
      this.testRateLimitBypass()
    ];

    return Promise.allSettled(tests);
  }

  async runAPITests(): Promise<TestResult[]> {
    const tests = [
      this.testSQLInjection(),
      this.testNoSQLInjection(),
      this.testXSS(),
      this.testAuthorizationBypass(),
      this.testParameterPollution()
    ];

    return Promise.allSettled(tests);
  }
}
```

## Incident Response

### 1. Incident Classification

| Severity | Description | Response Time |
|----------|-------------|---------------|
| Critical | System compromise, data breach | 15 minutes |
| High | Service disruption, security control failure | 1 hour |
| Medium | Suspicious activity, policy violation | 4 hours |
| Low | Configuration issue, minor security event | 24 hours |

### 2. Incident Response Process

```typescript
// Incident response automation
class IncidentResponse {
  async handleSecurityIncident(incident: SecurityIncident): Promise<void> {
    // 1. Containment
    await this.containIncident(incident);

    // 2. Investigation
    const investigation = await this.investigateIncident(incident);

    // 3. Eradication
    await this.eradicateThreat(investigation);

    // 4. Recovery
    await this.recoverFromIncident(incident);

    // 5. Lessons learned
    await this.documentLessonsLearned(incident);
  }

  private async containIncident(incident: SecurityIncident): Promise<void> {
    // Block malicious IPs
    await this.firewallService.blockIP(incident.sourceIP);

    // Disable compromised accounts
    await this.authService.disableAccount(incident.userId);

    // Isolate affected systems
    await this.infrastructureService.isolate(incident.affectedSystems);

    // Preserve evidence
    await this.preserveForensicData(incident);
  }
}
```

## Security Best Practices

### 1. Development
- Implement secure coding guidelines
- Use security linters and static analysis
- Regular security code reviews
- Dependency vulnerability scanning

### 2. Deployment
- Use secrets management
- Implement infrastructure as code
- Regular configuration audits
- Immutable deployments

### 3. Operations
- Continuous security monitoring
- Regular penetration testing
- Security awareness training
- Incident response drills

## Security Roadmap

### Short Term (3 months)
- Implement Web Application Firewall (WAF)
- Add advanced threat detection
- Enhance logging and monitoring
- Conduct third-party security audit

### Medium Term (6 months)
- Implement zero trust architecture
- Add behavioral analysis
- Deploy deception technology
- Establish bug bounty program

### Long Term (12 months)
- AI-powered security automation
- Advanced threat hunting
- Supply chain security hardening
- Continuous compliance monitoring

This security overview demonstrates FastGPT's commitment to maintaining a robust security posture and protecting user data throughout the platform.