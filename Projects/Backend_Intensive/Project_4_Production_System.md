# Project 4: Production System
*Backend Intensive Plan - Weeks 7-8*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 7-8)  
**Tech Stack**: Multi-language system (Rails + Python + Node.js) + AWS + Docker + Kubernetes  
**Focus**: Production deployment, multi-service orchestration, and business operations  

### Learning Objectives
- Master production deployment across multiple languages
- Implement comprehensive monitoring and alerting
- Build scalable and resilient multi-service systems
- Handle production challenges and troubleshooting
- Implement CI/CD pipelines and DevOps practices

---

## 🏗️ Multi-Service Production Architecture

### System Overview
```
┌─────────────────────────────────────────────────────────────────┐
│                        Load Balancer (AWS ALB)                 │
└─────────────────────┬───────────────────────────────────────────┘
                      │
┌─────────────────────┼───────────────────────────────────────────┐
│                     │                                           │
│  ┌─────────────────▼─────────────────┐  ┌─────────────────────▼─────────────────┐
│  │         API Gateway               │  │         Admin Dashboard               │
│  │         (Node.js)                 │  │         (Next.js)                     │
│  └─────────────────┬─────────────────┘  └─────────────────────┬─────────────────┘
│                    │                                           │
└────────────────────┼───────────────────────────────────────────┼─────────────────────┘
                     │                                           │
┌────────────────────┼───────────────────────────────────────────┼─────────────────────┐
│                    │                                           │                     │
│  ┌─────────────────▼─────────────────┐  ┌─────────────────────▼─────────────────┐  │
│  │      User Management              │  │      Product Catalog                  │  │
│  │      (Ruby on Rails)              │  │      (Ruby on Rails)                  │  │
│  └─────────────────┬─────────────────┘  └─────────────────────┬─────────────────┘  │
│                    │                                           │                     │
│  ┌─────────────────▼─────────────────┐  ┌─────────────────────▼─────────────────┐  │
│  │      Data Processing              │  │      Real-time Chat                   │  │
│  │      (Python/FastAPI)             │  │      (Node.js/Express)                │  │
│  └─────────────────┬─────────────────┘  └─────────────────────┬─────────────────┘  │
│                    │                                           │                     │
└────────────────────┼───────────────────────────────────────────┼─────────────────────┘
                     │                                           │
┌────────────────────┼───────────────────────────────────────────┼─────────────────────┐
│                    │                                           │                     │
│  ┌─────────────────▼─────────────────┐  ┌─────────────────────▼─────────────────┐  │
│  │      PostgreSQL (RDS)             │  │      MongoDB (DocumentDB)             │  │
│  │      - Users & Products           │  │      - Messages & Analytics            │  │
│  └───────────────────────────────────┘  └───────────────────────────────────────┘  │
│                                                                                     │
│  ┌─────────────────┬─────────────────┐  ┌─────────────────────┬─────────────────┐  │
│  │      Redis       │      S3         │  │      SQS            │      CloudWatch │  │
│  │      (Cache)     │      (Files)    │  │      (Queues)       │      (Logs)     │  │
│  └─────────────────┴─────────────────┘  └─────────────────────┴─────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────────┘
```

### Service Responsibilities

#### 1. API Gateway (Node.js)
- **Port**: 3000
- **Purpose**: Request routing, authentication, rate limiting, API versioning
- **Features**: Load balancing, circuit breakers, request/response transformation

#### 2. User Management Service (Ruby on Rails)
- **Port**: 3001
- **Purpose**: User authentication, profiles, permissions, account management
- **Database**: PostgreSQL
- **Features**: JWT tokens, role-based access, email verification

#### 3. Product Catalog Service (Ruby on Rails)
- **Port**: 3002
- **Purpose**: Product management, inventory, categories, search
- **Database**: PostgreSQL
- **Features**: Full-text search, inventory tracking, product recommendations

#### 4. Data Processing Service (Python/FastAPI)
- **Port**: 8000
- **Purpose**: Analytics, reporting, ETL pipelines, business intelligence
- **Database**: PostgreSQL + MongoDB
- **Features**: Data processing, ML predictions, scheduled reports

#### 5. Real-time Chat Service (Node.js)
- **Port**: 3003
- **Purpose**: Real-time messaging, notifications, live chat
- **Database**: MongoDB
- **Features**: WebSockets, file sharing, message encryption

---

## 📊 Production Database Schema

### Cross-Service Data Consistency
```sql
-- Shared database for user sessions and API keys
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  service_name VARCHAR(50) NOT NULL,
  key_hash VARCHAR(255) NOT NULL,
  permissions JSONB DEFAULT '{}',
  rate_limit INTEGER DEFAULT 1000,
  expires_at TIMESTAMP,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Service health monitoring
CREATE TABLE service_health (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  service_name VARCHAR(50) NOT NULL,
  status VARCHAR(20) NOT NULL,
  response_time_ms INTEGER,
  error_rate DECIMAL(5,2),
  last_check TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  metadata JSONB DEFAULT '{}'
);

-- Distributed transactions log
CREATE TABLE distributed_transactions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  transaction_id UUID NOT NULL,
  service_name VARCHAR(50) NOT NULL,
  operation VARCHAR(100) NOT NULL,
  status VARCHAR(20) NOT NULL,
  data JSONB DEFAULT '{}',
  error_message TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  completed_at TIMESTAMP
);
```

### Business Intelligence Schema
```sql
-- Daily business metrics
CREATE TABLE daily_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date DATE NOT NULL,
  service_name VARCHAR(50) NOT NULL,
  metric_name VARCHAR(100) NOT NULL,
  metric_value DECIMAL(15,2) NOT NULL,
  dimensions JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(date, service_name, metric_name, dimensions)
);

-- User behavior analytics
CREATE TABLE user_behavior (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL,
  session_id VARCHAR(100) NOT NULL,
  event_type VARCHAR(50) NOT NULL,
  event_data JSONB DEFAULT '{}',
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  service_name VARCHAR(50) NOT NULL
);

-- Performance metrics
CREATE TABLE performance_metrics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  service_name VARCHAR(50) NOT NULL,
  endpoint VARCHAR(200) NOT NULL,
  method VARCHAR(10) NOT NULL,
  response_time_ms INTEGER NOT NULL,
  status_code INTEGER NOT NULL,
  request_size_bytes INTEGER,
  response_size_bytes INTEGER,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🛠️ Production API Specifications

### API Gateway Configuration
```javascript
// gateway/config.js
module.exports = {
  services: {
    userService: {
      url: process.env.USER_SERVICE_URL || 'http://user-service:3001',
      timeout: 5000,
      retries: 3,
      circuitBreaker: {
        threshold: 5,
        timeout: 60000,
        resetTimeout: 30000
      }
    },
    productService: {
      url: process.env.PRODUCT_SERVICE_URL || 'http://product-service:3002',
      timeout: 5000,
      retries: 3,
      circuitBreaker: {
        threshold: 5,
        timeout: 60000,
        resetTimeout: 30000
      }
    },
    dataService: {
      url: process.env.DATA_SERVICE_URL || 'http://data-service:8000',
      timeout: 10000,
      retries: 2,
      circuitBreaker: {
        threshold: 3,
        timeout: 120000,
        resetTimeout: 60000
      }
    },
    chatService: {
      url: process.env.CHAT_SERVICE_URL || 'http://chat-service:3003',
      timeout: 5000,
      retries: 3,
      circuitBreaker: {
        threshold: 5,
        timeout: 60000,
        resetTimeout: 30000
      }
    }
  },
  
  routes: [
    {
      path: '/api/v1/users/*',
      service: 'userService',
      methods: ['GET', 'POST', 'PUT', 'DELETE'],
      auth: true
    },
    {
      path: '/api/v1/products/*',
      service: 'productService',
      methods: ['GET', 'POST', 'PUT', 'DELETE'],
      auth: true
    },
    {
      path: '/api/v1/analytics/*',
      service: 'dataService',
      methods: ['GET', 'POST'],
      auth: true
    },
    {
      path: '/api/v1/chat/*',
      service: 'chatService',
      methods: ['GET', 'POST', 'PUT', 'DELETE'],
      auth: true
    }
  ]
};
```

### Service Health Monitoring
```javascript
// gateway/health.js
const axios = require('axios');
const config = require('./config');

class HealthMonitor {
  constructor() {
    this.services = config.services;
    this.healthStatus = {};
  }

  async checkAllServices() {
    const healthChecks = Object.entries(this.services).map(async ([name, service]) => {
      try {
        const startTime = Date.now();
        const response = await axios.get(`${service.url}/health`, {
          timeout: service.timeout
        });
        const responseTime = Date.now() - startTime;

        this.healthStatus[name] = {
          status: 'healthy',
          responseTime,
          lastCheck: new Date().toISOString(),
          error: null
        };

        // Log to monitoring system
        await this.logHealthMetric(name, 'healthy', responseTime);

      } catch (error) {
        this.healthStatus[name] = {
          status: 'unhealthy',
          responseTime: null,
          lastCheck: new Date().toISOString(),
          error: error.message
        };

        // Log to monitoring system
        await this.logHealthMetric(name, 'unhealthy', null, error.message);
      }
    });

    await Promise.all(healthChecks);
  }

  async logHealthMetric(serviceName, status, responseTime, error = null) {
    // Log to CloudWatch, Prometheus, or other monitoring system
    console.log({
      service: serviceName,
      status,
      responseTime,
      error,
      timestamp: new Date().toISOString()
    });
  }

  getHealthStatus() {
    const overallStatus = Object.values(this.healthStatus).every(
      service => service.status === 'healthy'
    ) ? 'healthy' : 'unhealthy';

    return {
      status: overallStatus,
      services: this.healthStatus,
      timestamp: new Date().toISOString()
    };
  }
}

module.exports = new HealthMonitor();
```

---

## 🐳 Kubernetes Deployment

### API Gateway Deployment
```yaml
# k8s/api-gateway-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-gateway
  namespace: production
  labels:
    app: api-gateway
spec:
  replicas: 3
  selector:
    matchLabels:
      app: api-gateway
  template:
    metadata:
      labels:
        app: api-gateway
    spec:
      containers:
      - name: api-gateway
        image: your-registry/api-gateway:latest
        ports:
        - containerPort: 3000
        env:
        - name: NODE_ENV
          value: "production"
        - name: USER_SERVICE_URL
          value: "http://user-service:3001"
        - name: PRODUCT_SERVICE_URL
          value: "http://product-service:3002"
        - name: DATA_SERVICE_URL
          value: "http://data-service:8000"
        - name: CHAT_SERVICE_URL
          value: "http://chat-service:3003"
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secrets
              key: url
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: api-gateway-service
  namespace: production
spec:
  selector:
    app: api-gateway
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
  type: LoadBalancer
```

### User Service Deployment (Rails)
```yaml
# k8s/user-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: user-service
  namespace: production
  labels:
    app: user-service
spec:
  replicas: 2
  selector:
    matchLabels:
      app: user-service
  template:
    metadata:
      labels:
        app: user-service
    spec:
      containers:
      - name: user-service
        image: your-registry/user-service:latest
        ports:
        - containerPort: 3001
        env:
        - name: RAILS_ENV
          value: "production"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: database-secrets
              key: url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: redis-secrets
              key: url
        - name: SECRET_KEY_BASE
          valueFrom:
            secretKeyRef:
              name: rails-secrets
              key: secret_key_base
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "500m"
        livenessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /health
            port: 3001
          initialDelaySeconds: 30
          periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: user-service
  namespace: production
spec:
  selector:
    app: user-service
  ports:
  - protocol: TCP
    port: 3001
    targetPort: 3001
```

---

## 📊 Monitoring & Observability

### Prometheus Metrics Collection
```javascript
// monitoring/metrics.js
const prometheus = require('prom-client');

// Create a Registry to register the metrics
const register = new prometheus.Registry();

// Add a default label which is added to all metrics
register.setDefaultLabels({
  app: 'api-gateway',
  environment: process.env.NODE_ENV || 'development'
});

// Enable the collection of default metrics
prometheus.collectDefaultMetrics({ register });

// Custom business metrics
const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});

const httpRequestTotal = new prometheus.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

const activeConnections = new prometheus.Gauge({
  name: 'active_connections_total',
  help: 'Number of active connections'
});

const serviceHealthStatus = new prometheus.Gauge({
  name: 'service_health_status',
  help: 'Health status of downstream services',
  labelNames: ['service_name']
});

const businessMetrics = new prometheus.Counter({
  name: 'business_events_total',
  help: 'Total number of business events',
  labelNames: ['event_type', 'service']
});

// Register all metrics
register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);
register.registerMetric(activeConnections);
register.registerMetric(serviceHealthStatus);
register.registerMetric(businessMetrics);

module.exports = {
  register,
  httpRequestDuration,
  httpRequestTotal,
  activeConnections,
  serviceHealthStatus,
  businessMetrics
};
```

### Grafana Dashboard Configuration
```json
{
  "dashboard": {
    "title": "Production System Overview",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{route}}"
          }
        ]
      },
      {
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      },
      {
        "title": "Service Health",
        "type": "singlestat",
        "targets": [
          {
            "expr": "service_health_status",
            "legendFormat": "{{service_name}}"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status_code=~\"5..\"}[5m])",
            "legendFormat": "5xx errors"
          }
        ]
      },
      {
        "title": "Business Metrics",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(business_events_total[5m])",
            "legendFormat": "{{event_type}} from {{service}}"
          }
        ]
      }
    ]
  }
}
```

---

## 🚀 CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/production-deploy.yml
name: Production Deployment

on:
  push:
    branches: [main]
  workflow_dispatch:

env:
  AWS_REGION: us-east-1
  ECR_REGISTRY: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.us-east-1.amazonaws.com

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [api-gateway, user-service, product-service, data-service, chat-service]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Node.js (for Node.js services)
      if: contains(matrix.service, 'api-gateway') || contains(matrix.service, 'chat-service')
      uses: actions/setup-node@v3
      with:
        node-version: '18'
        cache: 'npm'
        cache-dependency-path: ${{ matrix.service }}/package-lock.json
    
    - name: Set up Ruby (for Rails services)
      if: contains(matrix.service, 'user-service') || contains(matrix.service, 'product-service')
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: '3.2'
        bundler-cache: true
        working-directory: ${{ matrix.service }}
    
    - name: Set up Python (for FastAPI service)
      if: matrix.service == 'data-service'
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'
        cache: 'pip'
        cache-dependency-path: ${{ matrix.service }}/requirements.txt
    
    - name: Install dependencies
      run: |
        cd ${{ matrix.service }}
        if [ -f package.json ]; then npm ci; fi
        if [ -f Gemfile ]; then bundle install; fi
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi
    
    - name: Run tests
      run: |
        cd ${{ matrix.service }}
        if [ -f package.json ]; then npm test; fi
        if [ -f Gemfile ]; then bundle exec rspec; fi
        if [ -f requirements.txt ]; then pytest; fi
    
    - name: Run security scan
      run: |
        cd ${{ matrix.service }}
        if [ -f package.json ]; then npm audit --audit-level moderate; fi
        if [ -f Gemfile ]; then bundle audit; fi
        if [ -f requirements.txt ]; then safety check; fi

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [api-gateway, user-service, product-service, data-service, chat-service]
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
    
    - name: Build and push Docker image
      env:
        ECR_REPOSITORY: ${{ matrix.service }}
        IMAGE_TAG: ${{ github.sha }}
      run: |
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG ./${{ matrix.service }}
        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:latest ./${{ matrix.service }}
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    
    - name: Update kubeconfig
      run: aws eks update-kubeconfig --region ${{ env.AWS_REGION }} --name production-cluster
    
    - name: Deploy to Kubernetes
      run: |
        # Update image tags in Kubernetes manifests
        find k8s -name "*.yaml" -exec sed -i "s/:latest/:${{ github.sha }}/g" {} \;
        
        # Apply Kubernetes manifests
        kubectl apply -f k8s/
        
        # Wait for rollout to complete
        kubectl rollout status deployment/api-gateway -n production
        kubectl rollout status deployment/user-service -n production
        kubectl rollout status deployment/product-service -n production
        kubectl rollout status deployment/data-service -n production
        kubectl rollout status deployment/chat-service -n production
    
    - name: Run smoke tests
      run: |
        # Wait for services to be ready
        kubectl wait --for=condition=available --timeout=300s deployment/api-gateway -n production
        
        # Run basic health checks
        API_URL=$(kubectl get service api-gateway-service -n production -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
        curl -f http://$API_URL/health || exit 1
```

---

## 📚 Learning Resources

### Essential Reading
- [Kubernetes Documentation](https://kubernetes.io/docs/) - Container orchestration
- [AWS Documentation](https://docs.aws.amazon.com/) - Cloud services
- [Prometheus Documentation](https://prometheus.io/docs/) - Monitoring
- [Docker Production Guide](https://docs.docker.com/config/production/) - Production deployment

### Udemy Courses (Focus Sections)
- **Kubernetes Complete Course** - Sections 5-9 (Production deployment, Monitoring)
- **AWS Fundamentals** - Sections 4-8 (RDS, ECS, S3, CloudWatch)
- **Docker Production** - Sections 6-10 (Multi-container apps, Security)

### Practice Projects
- Deploy a multi-service application to AWS EKS
- Set up monitoring with Prometheus and Grafana
- Implement CI/CD pipeline with GitHub Actions

---

## ✅ Success Criteria

### Week 7 Goals
- [ ] Set up multi-service architecture on AWS
- [ ] Implement service communication and API Gateway
- [ ] Deploy services with Docker and Kubernetes
- [ ] Set up monitoring and logging
- [ ] Implement comprehensive security measures

### Week 8 Goals
- [ ] Create business intelligence dashboard
- [ ] Implement CI/CD pipeline
- [ ] Add comprehensive alerting
- [ ] Perform load testing and optimization
- [ ] Document production procedures
- [ ] Achieve 99.9% uptime target

### Final Deliverables
- [ ] Production-ready multi-service system
- [ ] Comprehensive monitoring and alerting
- [ ] Automated CI/CD pipeline
- [ ] Business intelligence dashboard
- [ ] Production documentation and runbooks
- [ ] Load testing results and performance metrics
- [ ] Security audit and compliance documentation

---

## 🔗 Related Projects
- **Previous Project**: [Project 3: Real-time Application](../Backend_Intensive/Project_3_Real_Time_Application/)
- **Training Plan**: [Backend Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)
- **Main README**: [Repository Overview](../../README.html)
