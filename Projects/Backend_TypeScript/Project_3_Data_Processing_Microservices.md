# Project 3: Data Processing Microservices
*Backend TypeScript Intensive Plan - Weeks 5-6*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive.html/)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 5-6)  
**Tech Stack**: NestJS + Python/FastAPI + PostgreSQL + MongoDB + Redis + RabbitMQ  
**Focus**: Microservices architecture, event-driven design, and data processing  

### Learning Objectives
- Master microservices architecture patterns
- Implement event-driven communication between services
- Build data processing pipelines with Python
- Handle distributed system challenges
- Implement monitoring and logging across services

---

## 🏗️ Microservices Architecture

### Service Overview
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   API Gateway   │    │  User Service   │    │ Product Service │
│   (NestJS)      │◄──►│   (NestJS)      │    │   (NestJS)      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Event Bus      │    │  Data Service   │    │ Analytics Service│
│  (RabbitMQ)     │◄──►│   (FastAPI)     │    │   (FastAPI)     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Database      │    │   File Storage  │    │   Monitoring    │
│  (PostgreSQL)   │    │     (S3)        │    │   (Prometheus)  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Service Responsibilities

#### 1. API Gateway (NestJS)
- **Port**: 3000
- **Purpose**: Single entry point, routing, authentication, rate limiting
- **Database**: None (stateless)

#### 2. User Service (NestJS)
- **Port**: 3001
- **Purpose**: User management, authentication, profiles
- **Database**: PostgreSQL
- **Events**: `user.created`, `user.updated`, `user.deleted`

#### 3. Product Service (NestJS)
- **Port**: 3002
- **Purpose**: Product catalog, inventory management
- **Database**: PostgreSQL
- **Events**: `product.created`, `product.updated`, `product.low_stock`

#### 4. Data Service (FastAPI)
- **Port**: 8000
- **Purpose**: Data ingestion, processing, ETL pipelines
- **Database**: MongoDB (raw data), PostgreSQL (processed data)
- **Events**: `data.processed`, `data.validated`, `data.error`

#### 5. Analytics Service (FastAPI)
- **Port**: 8001
- **Purpose**: Analytics, reporting, ML predictions
- **Database**: PostgreSQL (aggregated data)
- **Events**: `analytics.generated`, `prediction.completed`

---

## 📊 Database Schema Design

### User Service (PostgreSQL)
```sql
-- Users table (same as Project 1)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  role user_role NOT NULL DEFAULT 'customer',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- User activity tracking
CREATE TABLE user_activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  activity_type VARCHAR(50) NOT NULL,
  activity_data JSONB,
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Product Service (PostgreSQL)
```sql
-- Products table (same as Project 1)
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(200) NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  stock_quantity INTEGER DEFAULT 0,
  category_id UUID,
  vendor_id UUID,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Product analytics
CREATE TABLE product_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  date DATE NOT NULL,
  views INTEGER DEFAULT 0,
  purchases INTEGER DEFAULT 0,
  revenue DECIMAL(10,2) DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(product_id, date)
);
```

### Data Service (MongoDB)
```javascript
// Raw data collection
{
  _id: ObjectId,
  source: String, // 'csv', 'api', 'webhook'
  dataType: String, // 'sales', 'users', 'products'
  rawData: Object, // Original data structure
  processedAt: Date,
  status: String, // 'pending', 'processing', 'completed', 'error'
  errorMessage: String,
  metadata: {
    fileSize: Number,
    recordCount: Number,
    checksum: String
  },
  createdAt: Date
}

// Processed data collection
{
  _id: ObjectId,
  originalId: ObjectId,
  dataType: String,
  processedData: Object, // Cleaned and validated data
  validationResults: {
    isValid: Boolean,
    errors: [String],
    warnings: [String]
  },
  processedAt: Date,
  createdAt: Date
}
```

### Analytics Service (PostgreSQL)
```sql
-- Daily analytics aggregations
CREATE TABLE daily_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  date DATE NOT NULL,
  metric_type VARCHAR(50) NOT NULL,
  metric_value DECIMAL(15,2) NOT NULL,
  dimensions JSONB, -- Additional grouping dimensions
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(date, metric_type, dimensions)
);

-- ML predictions
CREATE TABLE predictions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  model_name VARCHAR(100) NOT NULL,
  prediction_type VARCHAR(50) NOT NULL,
  input_data JSONB NOT NULL,
  prediction_result JSONB NOT NULL,
  confidence_score DECIMAL(3,2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🛠️ API Specifications

### API Gateway Endpoints

#### Health Check
```http
GET /health
Response: 200 OK
{
  "status": "healthy",
  "services": {
    "user": "healthy",
    "product": "healthy",
    "data": "healthy",
    "analytics": "healthy"
  },
  "timestamp": "2024-01-01T00:00:00Z"
}
```

#### Service Discovery
```http
GET /services
Response: 200 OK
{
  "services": [
    {
      "name": "user-service",
      "url": "http://user-service:3001",
      "status": "healthy",
      "version": "1.0.0"
    }
  ]
}
```

### Data Service Endpoints (FastAPI)

#### POST /data/ingest
**Request Body:**
```json
{
  "source": "csv",
  "dataType": "sales",
  "data": [
    {
      "date": "2024-01-01",
      "product_id": "uuid",
      "quantity": 10,
      "price": 99.99
    }
  ]
}
```

**Response (202):**
```json
{
  "jobId": "uuid",
  "status": "processing",
  "estimatedCompletionTime": "2024-01-01T00:05:00Z"
}
```

#### GET /data/status/{jobId}
**Response (200):**
```json
{
  "jobId": "uuid",
  "status": "completed",
  "progress": 100,
  "processedRecords": 1000,
  "errors": [],
  "result": {
    "validRecords": 950,
    "invalidRecords": 50,
    "duplicateRecords": 10
  }
}
```

### Analytics Service Endpoints (FastAPI)

#### POST /analytics/generate
**Request Body:**
```json
{
  "reportType": "sales_summary",
  "dateRange": {
    "start": "2024-01-01",
    "end": "2024-01-31"
  },
  "groupBy": ["category", "region"],
  "metrics": ["revenue", "units_sold", "avg_price"]
}
```

**Response (202):**
```json
{
  "reportId": "uuid",
  "status": "processing",
  "estimatedCompletionTime": "2024-01-01T00:10:00Z"
}
```

#### GET /analytics/reports/{reportId}
**Response (200):**
```json
{
  "reportId": "uuid",
  "status": "completed",
  "data": {
    "summary": {
      "totalRevenue": 125000.00,
      "totalUnitsSold": 2500,
      "averagePrice": 50.00
    },
    "breakdown": [
      {
        "category": "Electronics",
        "region": "North",
        "revenue": 45000.00,
        "unitsSold": 900,
        "averagePrice": 50.00
      }
    ]
  }
}
```

---

## 🔄 Event-Driven Architecture

### Event Bus Configuration (RabbitMQ)
```javascript
// Event schemas
const events = {
  'user.created': {
    schema: {
      userId: 'string',
      email: 'string',
      role: 'string',
      timestamp: 'string'
    }
  },
  'product.low_stock': {
    schema: {
      productId: 'string',
      currentStock: 'number',
      threshold: 'number',
      vendorId: 'string',
      timestamp: 'string'
    }
  },
  'data.processed': {
    schema: {
      jobId: 'string',
      dataType: 'string',
      recordCount: 'number',
      status: 'string',
      timestamp: 'string'
    }
  }
};
```

### Event Handlers

#### User Service Event Handlers
```typescript
@EventPattern('user.created')
async handleUserCreated(data: UserCreatedEvent) {
  // Send welcome email
  await this.emailService.sendWelcomeEmail(data.email);
  
  // Create analytics profile
  await this.analyticsService.createUserProfile(data.userId);
  
  // Log user creation
  await this.loggingService.log('USER_CREATED', data);
}
```

#### Data Service Event Handlers
```typescript
@EventPattern('data.processed')
async handleDataProcessed(data: DataProcessedEvent) {
  // Update analytics with new data
  await this.analyticsService.updateMetrics(data);
  
  // Trigger ML model retraining if needed
  if (data.recordCount > 10000) {
    await this.mlService.scheduleModelRetraining(data.dataType);
  }
}
```

---

## 🧪 Testing Requirements

### Unit Tests (80%+ coverage required)
- **Service Logic**: Business logic for each microservice
- **Event Handlers**: Event processing and side effects
- **Data Processing**: ETL pipeline validation
- **API Controllers**: Request/response handling

### Integration Tests
- **Service Communication**: Inter-service API calls
- **Event Flow**: End-to-end event processing
- **Database Operations**: Cross-service data consistency
- **Message Queue**: RabbitMQ message handling

### End-to-End Tests
- **Data Pipeline**: Complete data ingestion to analytics
- **User Journey**: Registration to analytics generation
- **Error Handling**: Service failure scenarios

---

## 🚀 Deployment Requirements

### Docker Compose Configuration
```yaml
version: '3.8'
services:
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - rabbitmq
      - user-service
      - product-service

  user-service:
    build: ./user-service
    ports:
      - "3001:3001"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/users
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - postgres
      - rabbitmq

  data-service:
    build: ./data-service
    ports:
      - "8000:8000"
    environment:
      - MONGODB_URL=mongodb://mongodb:27017/data
      - POSTGRES_URL=postgresql://user:pass@postgres:5432/analytics
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - mongodb
      - postgres
      - rabbitmq

  analytics-service:
    build: ./analytics-service
    ports:
      - "8001:8001"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/analytics
      - RABBITMQ_URL=amqp://rabbitmq:5672
    depends_on:
      - postgres
      - rabbitmq

  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=main
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  mongodb:
    image: mongo:6
    volumes:
      - mongodb_data:/data/db

  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
  mongodb_data:
  rabbitmq_data:
```

### Environment Variables
```bash
# API Gateway
API_GATEWAY_PORT=3000
RABBITMQ_URL=amqp://localhost:5672
JWT_SECRET=your_jwt_secret

# User Service
USER_SERVICE_PORT=3001
DATABASE_URL=postgresql://user:pass@localhost:5432/users
RABBITMQ_URL=amqp://localhost:5672

# Data Service
DATA_SERVICE_PORT=8000
MONGODB_URL=mongodb://localhost:27017/data
POSTGRES_URL=postgresql://user:pass@localhost:5432/analytics
RABBITMQ_URL=amqp://localhost:5672

# Analytics Service
ANALYTICS_SERVICE_PORT=8001
DATABASE_URL=postgresql://user:pass@localhost:5432/analytics
RABBITMQ_URL=amqp://localhost:5672
```

---

## 📚 Learning Resources

### Essential Reading
- [Microservices Patterns](https://microservices.io/) - Architecture patterns
- [RabbitMQ Documentation](https://www.rabbitmq.com/documentation/) - Message queuing
- [FastAPI Documentation](https://fastapi.tiangolo.com/) - Python API framework
- [Docker Documentation](https://docs.docker.com/) - Containerization

### Udemy Courses (Focus Sections)
- **Microservices Architecture** - Sections 2-6 (Design patterns, Communication)
- **FastAPI Complete Course** - Sections 3-7 (Advanced features, Background tasks)
- **Docker Complete Course** - Sections 4-8 (Multi-container apps, Docker Compose)

### Practice Projects
- Build a simple microservices chat application
- Create a basic event-driven e-commerce system
- Implement a data processing pipeline with Python

---

## ✅ Success Criteria

### Week 5 Goals
- [ ] Set up microservices architecture
- [ ] Implement API Gateway with routing
- [ ] Create User and Product services
- [ ] Set up RabbitMQ message bus
- [ ] Implement basic event handling

### Week 6 Goals
- [ ] Build Data Service with ETL pipelines
- [ ] Create Analytics Service with reporting
- [ ] Implement comprehensive monitoring
- [ ] Add error handling and resilience
- [ ] Deploy with Docker Compose
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional microservices architecture
- [ ] Event-driven communication between services
- [ ] Data processing and analytics capabilities
- [ ] Comprehensive monitoring and logging
- [ ] 80%+ test coverage across all services
- [ ] Docker Compose deployment configuration
- [ ] API documentation for all services

---

## 🔗 Related Projects
- **Previous Project**: [Project 2: Social Media Backend API](../Backend_TypeScript/Project_2_Social_Media_API/)
- **Next Project**: [Project 4: Production System](../Backend_TypeScript/Project_4_Production_System/)
- **Training Plan**: [Backend TypeScript Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive.html/)
- **Main README**: [Repository Overview](../../README.html)
