# Project 4: E-commerce Microservices
*3-week project for TypeScript Backend Evening (Weeks 10-12)*

## Project Overview
Create a comprehensive e-commerce microservices platform with separate services for users, products, orders, and payments. This project focuses on microservices architecture, cloud deployment, and production-ready systems while building a scalable e-commerce solution.

## Learning Objectives
- Master microservices architecture patterns
- Implement service-to-service communication
- Deploy to cloud platforms
- Set up monitoring and observability
- Build production-ready systems

## Technology Stack
- **Language**: TypeScript 5.0+
- **Framework**: NestJS
- **Database**: PostgreSQL
- **ORM**: TypeORM
- **Message Queue**: Redis
- **API Gateway**: NestJS
- **Containerization**: Docker
- **Cloud**: AWS/GCP
- **Monitoring**: Prometheus, Grafana

## Project Timeline (3 Weeks)

### **Week 10: Microservices Architecture**
**Goal**: Design and implement microservices architecture

#### **Day 1-2: Service Design**
- Microservices architecture planning
- Service boundaries and responsibilities
- API Gateway setup
- Service discovery and registration
- Inter-service communication

#### **Day 3-4: Core Services Implementation**
- User Service (authentication, profiles)
- Product Service (catalog, inventory)
- Order Service (order management)
- Payment Service (payment processing)

#### **Day 5: Service Communication**
- HTTP communication between services
- Event-driven architecture with Redis
- Error handling and retry logic
- Circuit breaker pattern
- Service health checks

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Microservices theory and design
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 11: Cloud Deployment & DevOps**
**Goal**: Deploy microservices to cloud platforms

#### **Day 1-2: Containerization**
- Docker containerization for each service
- Docker Compose for local development
- Multi-stage builds for production
- Container optimization
- Health checks in containers

#### **Day 3-4: Cloud Deployment**
- AWS ECS/EKS deployment
- Load balancer configuration
- Database setup (RDS)
- Redis cluster setup (ElastiCache)
- Environment configuration

#### **Day 5: CI/CD Pipeline**
- GitHub Actions setup
- Automated testing
- Docker image building
- Deployment automation
- Rollback strategies

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Cloud deployment and DevOps
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 12: Monitoring & Production**
**Goal**: Set up monitoring and production readiness

#### **Day 1-2: Monitoring Setup**
- Prometheus metrics collection
- Grafana dashboards
- Application performance monitoring
- Error tracking and alerting
- Log aggregation

#### **Day 3-4: Production Optimization**
- Performance optimization
- Security hardening
- Database optimization
- Caching strategies
- Load testing

#### **Day 5: Documentation & Final Review**
- Complete documentation
- API documentation
- Deployment guide
- Monitoring guide
- Final testing and review

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Monitoring and production optimization
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and documentation

## Detailed Requirements

### **Microservices Architecture**

#### **Service Overview**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │    │   User Service  │    │ Product Service │
│                 │    │                 │    │                 │
│ - Authentication│    │ - User Management│    │ - Product Catalog│
│ - Rate Limiting │    │ - Profiles      │    │ - Inventory     │
│ - Load Balancing│    │ - Authentication│    │ - Categories    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         └───────────────────────┼───────────────────────┘
                                 │
         ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
         │  Order Service  │    │ Payment Service │    │ Notification    │
         │                 │    │                 │    │ Service         │
         │ - Order Mgmt    │    │ - Payment Proc  │    │ - Email         │
         │ - Order History │    │ - Refunds       │    │ - SMS           │
         │ - Order Status  │    │ - Webhooks      │    │ - Push Notif    │
         └─────────────────┘    └─────────────────┘    └─────────────────┘
```

#### **API Gateway Configuration**
```typescript
import { Module } from '@nestjs/common';
import { HttpModule } from '@nestjs/axios';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot(),
    HttpModule.register({
      timeout: 5000,
      maxRedirects: 5,
    }),
  ],
  controllers: [ApiGatewayController],
  providers: [ApiGatewayService, CircuitBreakerService],
})
export class ApiGatewayModule {}
```

#### **Service Communication**
```typescript
import { Injectable, HttpException, HttpStatus } from '@nestjs/common';
import { HttpService } from '@nestjs/axios';
import { firstValueFrom } from 'rxjs';
import { CircuitBreakerService } from './circuit-breaker.service';

@Injectable()
export class ServiceCommunicationService {
  constructor(
    private httpService: HttpService,
    private circuitBreaker: CircuitBreakerService,
  ) {}

  async callUserService(endpoint: string, data: any): Promise<any> {
    return this.circuitBreaker.execute(async () => {
      const response = await firstValueFrom(
        this.httpService.post(`${process.env.USER_SERVICE_URL}${endpoint}`, data)
      );
      return response.data;
    });
  }

  async callProductService(endpoint: string, data: any): Promise<any> {
    return this.circuitBreaker.execute(async () => {
      const response = await firstValueFrom(
        this.httpService.post(`${process.env.PRODUCT_SERVICE_URL}${endpoint}`, data)
      );
      return response.data;
    });
  }

  async callOrderService(endpoint: string, data: any): Promise<any> {
    return this.circuitBreaker.execute(async () => {
      const response = await firstValueFrom(
        this.httpService.post(`${process.env.ORDER_SERVICE_URL}${endpoint}`, data)
      );
      return response.data;
    });
  }

  async callPaymentService(endpoint: string, data: any): Promise<any> {
    return this.circuitBreaker.execute(async () => {
      const response = await firstValueFrom(
        this.httpService.post(`${process.env.PAYMENT_SERVICE_URL}${endpoint}`, data)
      );
      return response.data;
    });
  }
}
```

### **Database Schema**

#### **User Service Database**
```sql
-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- User addresses table
CREATE TABLE user_addresses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    type VARCHAR(20) NOT NULL, -- 'billing', 'shipping'
    street VARCHAR(255) NOT NULL,
    city VARCHAR(100) NOT NULL,
    state VARCHAR(100) NOT NULL,
    zip_code VARCHAR(20) NOT NULL,
    country VARCHAR(100) NOT NULL,
    is_default BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Product Service Database**
```sql
-- Categories table
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Products table
CREATE TABLE products (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    name VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    short_description TEXT,
    sku VARCHAR(100) UNIQUE NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    compare_price DECIMAL(10,2),
    cost_price DECIMAL(10,2),
    stock_quantity INTEGER DEFAULT 0,
    min_stock_quantity INTEGER DEFAULT 0,
    weight DECIMAL(8,2),
    dimensions JSONB,
    images JSONB,
    is_active BOOLEAN DEFAULT TRUE,
    is_featured BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Product variants table
CREATE TABLE product_variants (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    product_id UUID REFERENCES products(id) ON DELETE CASCADE,
    sku VARCHAR(100) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    stock_quantity INTEGER DEFAULT 0,
    attributes JSONB,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Order Service Database**
```sql
-- Orders table
CREATE TABLE orders (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    order_number VARCHAR(50) UNIQUE NOT NULL,
    status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'confirmed', 'shipped', 'delivered', 'cancelled'
    subtotal DECIMAL(10,2) NOT NULL,
    tax_amount DECIMAL(10,2) DEFAULT 0,
    shipping_amount DECIMAL(10,2) DEFAULT 0,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    total_amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    shipping_address JSONB NOT NULL,
    billing_address JSONB NOT NULL,
    notes TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Order items table
CREATE TABLE order_items (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
    product_id UUID NOT NULL,
    variant_id UUID,
    quantity INTEGER NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_price DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### **Payment Service Database**
```sql
-- Payments table
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    order_id UUID NOT NULL,
    amount DECIMAL(10,2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'completed', 'failed', 'refunded'
    payment_method VARCHAR(50) NOT NULL,
    payment_provider VARCHAR(50) NOT NULL,
    provider_transaction_id VARCHAR(255),
    provider_response JSONB,
    processed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Refunds table
CREATE TABLE refunds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    payment_id UUID REFERENCES payments(id) ON DELETE CASCADE,
    amount DECIMAL(10,2) NOT NULL,
    reason TEXT,
    status VARCHAR(20) DEFAULT 'pending', -- 'pending', 'completed', 'failed'
    provider_refund_id VARCHAR(255),
    processed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### **Event-Driven Architecture**

#### **Event Types**
```typescript
export enum EventType {
  USER_CREATED = 'user.created',
  USER_UPDATED = 'user.updated',
  PRODUCT_CREATED = 'product.created',
  PRODUCT_UPDATED = 'product.updated',
  ORDER_CREATED = 'order.created',
  ORDER_UPDATED = 'order.updated',
  PAYMENT_PROCESSED = 'payment.processed',
  PAYMENT_FAILED = 'payment.failed',
}

export interface BaseEvent {
  id: string;
  type: EventType;
  timestamp: Date;
  source: string;
  data: any;
}

export interface UserCreatedEvent extends BaseEvent {
  type: EventType.USER_CREATED;
  data: {
    userId: string;
    email: string;
    firstName: string;
    lastName: string;
  };
}

export interface OrderCreatedEvent extends BaseEvent {
  type: EventType.ORDER_CREATED;
  data: {
    orderId: string;
    userId: string;
    totalAmount: number;
    items: Array<{
      productId: string;
      quantity: number;
      unitPrice: number;
    }>;
  };
}
```

#### **Event Publisher Service**
```typescript
import { Injectable } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';
import { v4 as uuid } from 'uuid';
import { BaseEvent, EventType } from './event.types';

@Injectable()
export class EventPublisherService {
  constructor(private redisService: RedisService) {}

  async publishEvent<T extends BaseEvent>(event: Omit<T, 'id' | 'timestamp'>): Promise<void> {
    const fullEvent: T = {
      ...event,
      id: uuid(),
      timestamp: new Date(),
    } as T;

    await this.redisService.publish('events', JSON.stringify(fullEvent));
  }

  async publishUserCreated(data: any): Promise<void> {
    await this.publishEvent({
      type: EventType.USER_CREATED,
      source: 'user-service',
      data,
    });
  }

  async publishOrderCreated(data: any): Promise<void> {
    await this.publishEvent({
      type: EventType.ORDER_CREATED,
      source: 'order-service',
      data,
    });
  }

  async publishPaymentProcessed(data: any): Promise<void> {
    await this.publishEvent({
      type: EventType.PAYMENT_PROCESSED,
      source: 'payment-service',
      data,
    });
  }
}
```

#### **Event Subscriber Service**
```typescript
import { Injectable, OnModuleInit } from '@nestjs/common';
import { RedisService } from '@nestjs-modules/ioredis';
import { BaseEvent, EventType } from './event.types';

@Injectable()
export class EventSubscriberService implements OnModuleInit {
  constructor(private redisService: RedisService) {}

  onModuleInit() {
    this.redisService.subscribe('events');
    this.redisService.on('message', (channel, message) => {
      if (channel === 'events') {
        this.handleEvent(JSON.parse(message));
      }
    });
  }

  private async handleEvent(event: BaseEvent): Promise<void> {
    switch (event.type) {
      case EventType.USER_CREATED:
        await this.handleUserCreated(event);
        break;
      case EventType.ORDER_CREATED:
        await this.handleOrderCreated(event);
        break;
      case EventType.PAYMENT_PROCESSED:
        await this.handlePaymentProcessed(event);
        break;
      default:
        console.log(`Unhandled event type: ${event.type}`);
    }
  }

  private async handleUserCreated(event: any): Promise<void> {
    // Send welcome email
    // Create user profile
    // Set up default preferences
    console.log('Handling user created event:', event.data);
  }

  private async handleOrderCreated(event: any): Promise<void> {
    // Send order confirmation email
    // Update inventory
    // Create shipping label
    console.log('Handling order created event:', event.data);
  }

  private async handlePaymentProcessed(event: any): Promise<void> {
    // Send payment confirmation
    // Update order status
    // Trigger fulfillment
    console.log('Handling payment processed event:', event.data);
  }
}
```

### **Docker Configuration**

#### **Dockerfile for Services**
```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:18-alpine AS runtime

WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

RUN npm run build

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "dist/main.js"]
```

#### **Docker Compose for Local Development**
```yaml
version: '3.8'

services:
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - USER_SERVICE_URL=http://user-service:3001
      - PRODUCT_SERVICE_URL=http://product-service:3002
      - ORDER_SERVICE_URL=http://order-service:3003
      - PAYMENT_SERVICE_URL=http://payment-service:3004
    depends_on:
      - user-service
      - product-service
      - order-service
      - payment-service

  user-service:
    build: ./user-service
    ports:
      - "3001:3001"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/user_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  product-service:
    build: ./product-service
    ports:
      - "3002:3002"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/product_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  order-service:
    build: ./order-service
    ports:
      - "3003:3003"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/order_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  payment-service:
    build: ./payment-service
    ports:
      - "3004:3004"
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:password@postgres:5432/payment_db
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=user_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### **Monitoring Setup**

#### **Prometheus Configuration**
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'api-gateway'
    static_configs:
      - targets: ['api-gateway:3000']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'user-service'
    static_configs:
      - targets: ['user-service:3001']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'product-service'
    static_configs:
      - targets: ['product-service:3002']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'order-service'
    static_configs:
      - targets: ['order-service:3003']
    metrics_path: '/metrics'
    scrape_interval: 5s

  - job_name: 'payment-service'
    static_configs:
      - targets: ['payment-service:3004']
    metrics_path: '/metrics'
    scrape_interval: 5s
```

#### **Grafana Dashboard Configuration**
```json
{
  "dashboard": {
    "title": "E-commerce Microservices",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{service}}"
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
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "{{service}}"
          }
        ]
      }
    ]
  }
}
```

## Deliverables

### **Code Repository**
- Complete microservices architecture
- Docker containerization
- Event-driven communication
- Cloud deployment configuration
- Monitoring setup

### **Documentation**
- Microservices architecture guide
- API documentation for all services
- Deployment guide
- Monitoring and observability guide
- Troubleshooting guide

### **Demo Application**
- Fully functional e-commerce platform
- Microservices communication
- Real-time event processing
- Cloud deployment
- Monitoring dashboards

## Success Criteria

### **Functional Requirements**
- [ ] Complete microservices architecture
- [ ] Service-to-service communication
- [ ] Event-driven architecture
- [ ] Cloud deployment
- [ ] Monitoring and observability

### **Technical Requirements**
- [ ] TypeScript best practices
- [ ] Microservices patterns
- [ ] Docker containerization
- [ ] Cloud deployment
- [ ] Production readiness

### **Quality Requirements**
- [ ] Clean architecture
- [ ] Error handling
- [ ] Security best practices
- [ ] Performance optimization
- [ ] Documentation completeness

## Next Steps

1. **Design microservices architecture**
2. **Implement core services**
3. **Set up service communication**
4. **Deploy to cloud**
5. **Set up monitoring**
6. **Production optimization**

Ready to build your e-commerce microservices platform? Let's start coding! 🚀
