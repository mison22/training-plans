# Project 4: Rails Production System
*Complete DevOps pipeline with Docker, Kubernetes, CI/CD, and monitoring*

## Project Overview

Deploy a production-ready Rails application with comprehensive DevOps practices, including containerization, orchestration, CI/CD pipelines, monitoring, logging, and security. This project demonstrates expertise in deploying and operating Rails applications at scale.

## Technology Stack

- **Framework**: Rails 7.1+
- **Language**: Ruby 3.2+
- **Database**: PostgreSQL 15+ (with replication)
- **Cache**: Redis 7+ (with clustering)
- **Containerization**: Docker
- **Orchestration**: Kubernetes (AWS EKS)
- **CI/CD**: GitHub Actions
- **Load Balancer**: AWS ALB
- **CDN**: CloudFront
- **Storage**: AWS S3
- **Monitoring**: Prometheus + Grafana
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **APM**: New Relic or Datadog
- **Error Tracking**: Sentry or Rollbar
- **Security**: AWS Secrets Manager, SSL/TLS

## Core Features

### 1. Infrastructure as Code
- Terraform for AWS infrastructure
- Kubernetes manifests for deployment
- Docker Compose for local development
- Automated provisioning

### 2. CI/CD Pipeline
- Automated testing
- Code quality checks
- Security scanning
- Automated deployments
- Blue-green deployments
- Rollback capabilities

### 3. Monitoring & Observability
- Application metrics
- Infrastructure metrics
- Custom business metrics
- Distributed tracing
- Real-time alerting

### 4. Logging & Debugging
- Centralized logging
- Log aggregation
- Search and analysis
- Error tracking

### 5. Security & Compliance
- Secret management
- SSL/TLS encryption
- DDoS protection
- Security scanning
- Compliance monitoring

## Implementation Guide

### Week 7-8: DevOps & Production Deployment

#### Day 1-2: Docker Configuration
```dockerfile
# Dockerfile
FROM ruby:3.2-alpine AS builder

# Install build dependencies
RUN apk add --no-cache \
    build-base \
    postgresql-dev \
    nodejs \
    yarn \
    git

WORKDIR /app

# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle config set --local deployment 'true' && \
    bundle config set --local without 'development test' && \
    bundle install -j4

# Install node modules
COPY package.json yarn.lock ./
RUN yarn install --frozen-lockfile --production

# Copy application code
COPY . .

# Precompile assets
RUN RAILS_ENV=production SECRET_KEY_BASE=dummy bundle exec rails assets:precompile

# Final stage
FROM ruby:3.2-alpine

# Install runtime dependencies
RUN apk add --no-cache \
    postgresql-client \
    nodejs \
    tzdata

WORKDIR /app

# Copy built application from builder
COPY --from=builder /usr/local/bundle /usr/local/bundle
COPY --from=builder /app /app

# Create non-root user
RUN adduser -D -u 1000 rails && \
    chown -R rails:rails /app

USER rails

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["bundle", "exec", "puma", "-C", "config/puma.rb"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5

  web:
    build: .
    command: bundle exec puma -C config/puma.rb
    volumes:
      - .:/app
      - bundle_cache:/usr/local/bundle
    ports:
      - "3000:3000"
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    environment:
      DATABASE_URL: postgresql://postgres:password@db:5432/app_production
      REDIS_URL: redis://redis:6379/0
      RAILS_ENV: production
      SECRET_KEY_BASE: ${SECRET_KEY_BASE}
    healthcheck:
      test: ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
      interval: 30s
      timeout: 10s
      retries: 3

  sidekiq:
    build: .
    command: bundle exec sidekiq -C config/sidekiq.yml
    volumes:
      - .:/app
    depends_on:
      - db
      - redis
    environment:
      DATABASE_URL: postgresql://postgres:password@db:5432/app_production
      REDIS_URL: redis://redis:6379/0
      RAILS_ENV: production

volumes:
  postgres_data:
  redis_data:
  bundle_cache:
```

#### Day 3-5: Kubernetes Deployment
```yaml
# kubernetes/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production

---
# kubernetes/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: rails-app-config
  namespace: production
data:
  RAILS_ENV: "production"
  RAILS_LOG_TO_STDOUT: "true"
  RAILS_SERVE_STATIC_FILES: "true"

---
# kubernetes/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: rails-app-secrets
  namespace: production
type: Opaque
stringData:
  database-url: postgresql://user:password@postgres:5432/app_production
  redis-url: redis://redis:6379/0
  secret-key-base: your-secret-key-base-here

---
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rails-app
  namespace: production
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: rails-app
  template:
    metadata:
      labels:
        app: rails-app
        version: v1
    spec:
      containers:
      - name: rails-app
        image: rails-app:latest
        ports:
        - containerPort: 3000
          name: http
        envFrom:
        - configMapRef:
            name: rails-app-config
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: rails-app-secrets
              key: database-url
        - name: REDIS_URL
          valueFrom:
            secretKeyRef:
              name: rails-app-secrets
              key: redis-url
        - name: SECRET_KEY_BASE
          valueFrom:
            secretKeyRef:
              name: rails-app-secrets
              key: secret-key-base
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
          timeoutSeconds: 5
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
          timeoutSeconds: 3

---
# kubernetes/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: rails-app
  namespace: production
spec:
  type: ClusterIP
  selector:
    app: rails-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000

---
# kubernetes/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rails-app-ingress
  namespace: production
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - app.example.com
    secretName: rails-app-tls
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rails-app
            port:
              number: 80

---
# kubernetes/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: rails-app-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: rails-app
  minReplicas: 3
  maxReplicas: 10
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

#### Day 6-8: CI/CD Pipeline
```.github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  RUBY_VERSION: 3.2
  POSTGRES_VERSION: 15
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7-alpine
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Ruby
      uses: ruby/setup-ruby@v1
      with:
        ruby-version: ${{ env.RUBY_VERSION }}
        bundler-cache: true
    
    - name: Install dependencies
      run: |
        bundle install --jobs 4 --retry 3
        yarn install --frozen-lockfile
    
    - name: Setup database
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
        RAILS_ENV: test
      run: |
        bundle exec rails db:create
        bundle exec rails db:schema:load
    
    - name: Run tests
      env:
        DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
        REDIS_URL: redis://localhost:6379/0
        RAILS_ENV: test
      run: |
        bundle exec rspec
    
    - name: Run RuboCop
      run: bundle exec rubocop
    
    - name: Run Brakeman security scan
      run: bundle exec brakeman -z
    
    - name: Run Bundler Audit
      run: bundle exec bundle-audit check --update
    
    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        files: ./coverage/coverage.xml
        fail_ci_if_error: true

  build:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Log in to Container Registry
      uses: docker/login-action@v2
      with:
        registry: ${{ env.REGISTRY }}
        username: ${{ github.actor }}
        password: ${{ secrets.GITHUB_TOKEN }}
    
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1
    
    - name: Update kubeconfig
      run: |
        aws eks update-kubeconfig --name production-cluster --region us-east-1
    
    - name: Deploy to Kubernetes
      run: |
        kubectl set image deployment/rails-app \
          rails-app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:main-${{ github.sha }} \
          -n production
        
        kubectl rollout status deployment/rails-app -n production
    
    - name: Run database migrations
      run: |
        kubectl exec -n production \
          $(kubectl get pod -n production -l app=rails-app -o jsonpath='{.items[0].metadata.name}') \
          -- bundle exec rails db:migrate
    
    - name: Notify deployment
      uses: 8398a7/action-slack@v3
      with:
        status: ${{ job.status }}
        text: 'Deployment to production completed'
        webhook_url: ${{ secrets.SLACK_WEBHOOK }}
      if: always()
```

#### Day 9-11: Monitoring & Logging
```ruby
# config/initializers/prometheus.rb
require 'prometheus_exporter/middleware'
require 'prometheus_exporter/instrumentation'

# This reports stats per request like HTTP status and timings
Rails.application.middleware.unshift PrometheusExporter::Middleware

# Custom metrics
module PrometheusMetrics
  class << self
    def setup
      # Business metrics
      @orders_total = PrometheusExporter::Metric::Counter.new(
        "orders_total",
        "Total number of orders"
      )
      
      @revenue_total = PrometheusExporter::Metric::Counter.new(
        "revenue_total",
        "Total revenue"
      )
      
      @active_users = PrometheusExporter::Metric::Gauge.new(
        "active_users",
        "Number of active users"
      )
    end
    
    def record_order(amount)
      @orders_total.observe(1)
      @revenue_total.observe(amount)
    end
    
    def update_active_users(count)
      @active_users.observe(count)
    end
  end
end

PrometheusMetrics.setup

# config/initializers/lograge.rb
Rails.application.configure do
  config.lograge.enabled = true
  config.lograge.formatter = Lograge::Formatters::Json.new
  
  config.lograge.custom_options = lambda do |event|
    {
      time: event.time,
      remote_ip: event.payload[:remote_ip],
      user_id: event.payload[:user_id],
      params: event.payload[:params].except('controller', 'action')
    }
  end
end

# app/services/logging_service.rb
class LoggingService
  def self.logger
    @logger ||= Logger.new(STDOUT).tap do |log|
      log.formatter = proc do |severity, datetime, progname, msg|
        {
          timestamp: datetime.iso8601,
          severity: severity,
          message: msg,
          service: ENV['SERVICE_NAME'],
          environment: Rails.env
        }.to_json + "\n"
      end
    end
  end
  
  def self.info(message, metadata = {})
    logger.info({ message: message }.merge(metadata).to_json)
  end
  
  def self.error(message, error = nil, metadata = {})
    payload = {
      message: message,
      error_class: error&.class&.name,
      error_message: error&.message,
      backtrace: error&.backtrace&.first(10)
    }.merge(metadata)
    
    logger.error(payload.to_json)
  end
end
```

```yaml
# kubernetes/monitoring/prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    scrape_configs:
    - job_name: 'rails-app'
      kubernetes_sd_configs:
      - role: pod
        namespaces:
          names:
          - production
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_label_app]
        action: keep
        regex: rails-app
      - source_labels: [__meta_kubernetes_pod_ip]
        action: replace
        target_label: __address__
        replacement: ${1}:9394
    
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
    
    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod

---
# kubernetes/monitoring/grafana-dashboard.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboards
  namespace: monitoring
data:
  rails-dashboard.json: |
    {
      "dashboard": {
        "title": "Rails Application Dashboard",
        "panels": [
          {
            "title": "Request Rate",
            "targets": [
              {
                "expr": "rate(http_requests_total[5m])"
              }
            ]
          },
          {
            "title": "Response Time",
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
              }
            ]
          },
          {
            "title": "Error Rate",
            "targets": [
              {
                "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
              }
            ]
          },
          {
            "title": "Active Users",
            "targets": [
              {
                "expr": "active_users"
              }
            ]
          }
        ]
      }
    }
```

#### Day 12-14: Security & Performance
```ruby
# config/initializers/rack_attack.rb
class Rack::Attack
  # Throttle all requests by IP
  throttle('req/ip', limit: 300, period: 5.minutes) do |req|
    req.ip
  end
  
  # Throttle login attempts
  throttle('logins/ip', limit: 5, period: 20.seconds) do |req|
    if req.path == '/users/sign_in' && req.post?
      req.ip
    end
  end
  
  # Throttle API requests
  throttle('api/ip', limit: 1000, period: 1.hour) do |req|
    if req.path.start_with?('/api/')
      req.ip
    end
  end
  
  # Block suspicious requests
  blocklist('bad actors') do |req|
    Rack::Attack::Allow2Ban.filter(req.ip, maxretry: 5, findtime: 1.minute, bantime: 1.hour) do
      req.path == '/users/sign_in' && req.post?
    end
  end
end

# config/initializers/content_security_policy.rb
Rails.application.config.content_security_policy do |policy|
  policy.default_src :self, :https
  policy.font_src    :self, :https, :data
  policy.img_src     :self, :https, :data
  policy.object_src  :none
  policy.script_src  :self, :https
  policy.style_src   :self, :https
end

# app/models/concerns/encryptable.rb
module Encryptable
  extend ActiveSupport::Concern
  
  class_methods do
    def attr_encrypted(*attributes)
      attributes.each do |attribute|
        define_method("#{attribute}=") do |value|
          encrypted = encrypt(value)
          super(encrypted)
        end
        
        define_method(attribute) do
          encrypted_value = super()
          decrypt(encrypted_value) if encrypted_value
        end
      end
    end
  end
  
  private
  
  def encrypt(value)
    crypt = ActiveSupport::MessageEncryptor.new(Rails.application.credentials.secret_key_base[0..31])
    crypt.encrypt_and_sign(value)
  end
  
  def decrypt(value)
    crypt = ActiveSupport::MessageEncryptor.new(Rails.application.credentials.secret_key_base[0..31])
    crypt.decrypt_and_verify(value)
  end
end
```

## Terraform Infrastructure

```hcl
# terraform/main.tf
provider "aws" {
  region = "us-east-1"
}

# VPC
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  
  name = "production-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["us-east-1a", "us-east-1b", "us-east-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  enable_dns_hostnames = true
  
  tags = {
    Environment = "production"
  }
}

# EKS Cluster
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  
  cluster_name    = "production-cluster"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  
  eks_managed_node_groups = {
    main = {
      min_size     = 3
      max_size     = 10
      desired_size = 3
      
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
    }
  }
}

# RDS PostgreSQL
resource "aws_db_instance" "postgres" {
  identifier = "production-postgres"
  
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = "db.t3.medium"
  
  allocated_storage     = 100
  max_allocated_storage = 1000
  storage_encrypted     = true
  
  db_name  = "production"
  username = "postgres"
  password = var.db_password
  
  vpc_security_group_ids = [aws_security_group.rds.id]
  db_subnet_group_name   = aws_db_subnet_group.main.name
  
  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "Mon:04:00-Mon:05:00"
  
  multi_az               = true
  deletion_protection    = true
  skip_final_snapshot    = false
  final_snapshot_identifier = "production-postgres-final-snapshot"
}

# ElastiCache Redis
resource "aws_elasticache_replication_group" "redis" {
  replication_group_id       = "production-redis"
  replication_group_description = "Production Redis cluster"
  
  engine         = "redis"
  engine_version = "7.0"
  node_type      = "cache.t3.medium"
  
  number_cache_clusters = 3
  automatic_failover_enabled = true
  
  subnet_group_name = aws_elasticache_subnet_group.main.name
  security_group_ids = [aws_security_group.redis.id]
  
  at_rest_encryption_enabled = true
  transit_encryption_enabled = true
}
```

## Success Criteria

- [ ] Complete Docker containerization
- [ ] Kubernetes deployment successful
- [ ] CI/CD pipeline fully automated
- [ ] Monitoring and alerting configured
- [ ] Centralized logging implemented
- [ ] Security best practices applied
- [ ] Auto-scaling working correctly
- [ ] 99.9% uptime achieved
- [ ] Comprehensive documentation

## Performance Benchmarks

- **Response Time**: < 200ms (95th percentile)
- **Throughput**: 1000+ requests/second
- **Availability**: 99.9% uptime
- **Database Queries**: < 50ms average
- **Cache Hit Rate**: > 90%

---

📋 **[Back to Projects Overview](../Projects/Ruby_Rails_Backend/)** | 🏠 **[Back to Training Plan](../Intensive_Plans/Training_Plan_Ruby_Rails_Backend_Intensive.html)** | 🏠 **[Back to README](../README.html)**

### Related Projects
- **[Project 1: E-commerce Platform](./Project_1_Rails_E_Commerce.html)** - Full Rails application with Stripe
- **[Project 2: Social Media API](./Project_2_Social_Media_API.html)** - Rails API with real-time features
- **[Project 3: Microservices Platform](./Project_3_Microservices_Platform.html)** - Rails microservices architecture

