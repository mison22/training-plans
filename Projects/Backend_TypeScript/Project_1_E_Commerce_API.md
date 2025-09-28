# Project 1: E-commerce API Platform
*Backend TypeScript Intensive Plan - Weeks 1-2*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive/)** | 🏠 **[Back to README](../../README)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 1-2)  
**Tech Stack**: NestJS + TypeScript + PostgreSQL + Redis  
**Focus**: RESTful API development, authentication, and database design  

### Learning Objectives
- Master NestJS framework architecture and dependency injection
- Implement JWT authentication with refresh tokens
- Design and implement complex database relationships
- Build comprehensive RESTful APIs with proper error handling
- Implement caching strategies with Redis

---

## 📊 Database Schema Design

### Users Table
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  role user_role NOT NULL DEFAULT 'customer',
  avatar_url VARCHAR(500),
  email_verified BOOLEAN DEFAULT false,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  last_login TIMESTAMP
);

CREATE TYPE user_role AS ENUM ('admin', 'vendor', 'customer');
```

### Categories Table
```sql
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  description TEXT,
  parent_id UUID REFERENCES categories(id) ON DELETE SET NULL,
  image_url VARCHAR(500),
  is_active BOOLEAN DEFAULT true,
  sort_order INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Products Table
```sql
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(200) NOT NULL,
  slug VARCHAR(200) UNIQUE NOT NULL,
  description TEXT,
  short_description VARCHAR(500),
  sku VARCHAR(100) UNIQUE NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  compare_price DECIMAL(10,2),
  cost_price DECIMAL(10,2),
  stock_quantity INTEGER DEFAULT 0,
  low_stock_threshold INTEGER DEFAULT 5,
  weight DECIMAL(8,2),
  dimensions JSONB, -- {length, width, height}
  category_id UUID REFERENCES categories(id),
  vendor_id UUID REFERENCES users(id),
  is_active BOOLEAN DEFAULT true,
  is_featured BOOLEAN DEFAULT false,
  meta_title VARCHAR(200),
  meta_description VARCHAR(500),
  tags TEXT[], -- Array of tags
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Product Images Table
```sql
CREATE TABLE product_images (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  image_url VARCHAR(500) NOT NULL,
  alt_text VARCHAR(200),
  sort_order INTEGER DEFAULT 0,
  is_primary BOOLEAN DEFAULT false,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Shopping Cart Table
```sql
CREATE TABLE cart_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id) ON DELETE CASCADE,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(user_id, product_id)
);
```

### Orders Table
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  user_id UUID REFERENCES users(id),
  status order_status DEFAULT 'pending',
  subtotal DECIMAL(10,2) NOT NULL,
  tax_amount DECIMAL(10,2) DEFAULT 0,
  shipping_amount DECIMAL(10,2) DEFAULT 0,
  discount_amount DECIMAL(10,2) DEFAULT 0,
  total_amount DECIMAL(10,2) NOT NULL,
  currency VARCHAR(3) DEFAULT 'USD',
  shipping_address JSONB NOT NULL,
  billing_address JSONB NOT NULL,
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TYPE order_status AS ENUM (
  'pending', 'processing', 'shipped', 'delivered', 'cancelled', 'refunded'
);
```

### Order Items Table
```sql
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID REFERENCES products(id),
  product_snapshot JSONB NOT NULL, -- Store product details at time of order
  quantity INTEGER NOT NULL,
  unit_price DECIMAL(10,2) NOT NULL,
  total_price DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🛠️ API Endpoints Specification

### Authentication Endpoints

#### POST /auth/register
**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "firstName": "John",
  "lastName": "Doe",
  "role": "customer" // optional, defaults to customer
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "customer",
      "emailVerified": false,
      "createdAt": "2024-01-01T00:00:00Z"
    },
    "message": "Registration successful. Please verify your email."
  }
}
```

#### POST /auth/login
**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "role": "customer"
    },
    "accessToken": "jwt_access_token",
    "refreshToken": "jwt_refresh_token",
    "expiresIn": 3600
  }
}
```

### Product Endpoints

#### GET /products
**Query Parameters:**
- `page` (number): Page number (default: 1)
- `limit` (number): Items per page (default: 20, max: 100)
- `category` (string): Category slug
- `search` (string): Search term
- `minPrice` (number): Minimum price filter
- `maxPrice` (number): Maximum price filter
- `sortBy` (string): Sort field (price, name, createdAt)
- `sortOrder` (string): Sort direction (asc, desc)

**Response (200):**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "id": "uuid",
        "name": "Premium Wireless Headphones",
        "slug": "premium-wireless-headphones",
        "description": "High-quality wireless headphones...",
        "price": 299.99,
        "comparePrice": 399.99,
        "stockQuantity": 50,
        "category": {
          "id": "uuid",
          "name": "Electronics",
          "slug": "electronics"
        },
        "images": [
          {
            "url": "https://example.com/image1.jpg",
            "altText": "Headphones front view",
            "isPrimary": true
          }
        ],
        "tags": ["wireless", "premium", "audio"],
        "createdAt": "2024-01-01T00:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150,
      "totalPages": 8,
      "hasNext": true,
      "hasPrev": false
    }
  }
}
```

#### POST /products (Admin/Vendor only)
**Request Body:**
```json
{
  "name": "New Product",
  "description": "Product description",
  "price": 99.99,
  "comparePrice": 129.99,
  "stockQuantity": 100,
  "categoryId": "uuid",
  "sku": "PROD-001",
  "weight": 1.5,
  "dimensions": {
    "length": 10,
    "width": 5,
    "height": 3
  },
  "tags": ["new", "featured"],
  "images": [
    {
      "url": "https://example.com/image.jpg",
      "altText": "Product image",
      "isPrimary": true
    }
  ]
}
```

### Cart Endpoints

#### GET /cart
**Response (200):**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": "uuid",
        "product": {
          "id": "uuid",
          "name": "Product Name",
          "price": 99.99,
          "images": [
            {
              "url": "https://example.com/image.jpg",
              "isPrimary": true
            }
          ]
        },
        "quantity": 2,
        "subtotal": 199.98,
        "addedAt": "2024-01-01T00:00:00Z"
      }
    ],
    "summary": {
      "itemCount": 2,
      "subtotal": 199.98,
      "tax": 19.99,
      "total": 219.97
    }
  }
}
```

#### POST /cart/items
**Request Body:**
```json
{
  "productId": "uuid",
  "quantity": 2
}
```

### Order Endpoints

#### POST /orders
**Request Body:**
```json
{
  "shippingAddress": {
    "firstName": "John",
    "lastName": "Doe",
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "country": "US"
  },
  "billingAddress": {
    "firstName": "John",
    "lastName": "Doe",
    "street": "123 Main St",
    "city": "New York",
    "state": "NY",
    "zipCode": "10001",
    "country": "US"
  },
  "notes": "Please leave package at front door"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "order": {
      "id": "uuid",
      "orderNumber": "ORD-2024-001",
      "status": "pending",
      "subtotal": 199.98,
      "taxAmount": 19.99,
      "shippingAmount": 9.99,
      "totalAmount": 229.96,
      "items": [
        {
          "productName": "Product Name",
          "quantity": 2,
          "unitPrice": 99.99,
          "totalPrice": 199.98
        }
      ],
      "createdAt": "2024-01-01T00:00:00Z"
    }
  }
}
```

---

## 🧪 Testing Requirements

### Unit Tests (80%+ coverage required)
- **Auth Service**: Registration, login, token validation
- **User Service**: CRUD operations, role management
- **Product Service**: CRUD operations, search, filtering
- **Cart Service**: Add/remove items, calculations
- **Order Service**: Order creation, status updates

### Integration Tests
- **Authentication Flow**: Complete registration to login flow
- **Product Management**: Create, update, delete products
- **Shopping Cart**: Add items, calculate totals, persist cart
- **Order Processing**: Create order from cart, update status

### API Tests
- **Endpoint Validation**: All endpoints return correct status codes
- **Request Validation**: Proper error handling for invalid data
- **Authentication**: Protected endpoints require valid tokens
- **Rate Limiting**: API rate limits are enforced

---

## 🚀 Deployment Requirements

### Environment Setup
```bash
# Database
POSTGRES_DB=ecommerce_db
POSTGRES_USER=postgres
POSTGRES_PASSWORD=secure_password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=secure_redis_password

# JWT
JWT_SECRET=super_secure_jwt_secret_key
JWT_EXPIRES_IN=1h
JWT_REFRESH_SECRET=super_secure_refresh_secret
JWT_REFRESH_EXPIRES_IN=7d

# App
NODE_ENV=development
PORT=3000
API_PREFIX=api/v1
```

### Docker Configuration
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

EXPOSE 3000

CMD ["npm", "run", "start:prod"]
```

---

## 📚 Learning Resources

### Essential Reading
- [NestJS Documentation](https://docs.nestjs.com/) - Framework fundamentals
- [TypeORM Documentation](https://typeorm.io/) - Database ORM
- [JWT.io](https://jwt.io/) - JWT token understanding
- [PostgreSQL Documentation](https://www.postgresql.org/docs/) - Database features

### Udemy Courses (Focus Sections)
- **NestJS Complete Course** - Sections 3-8 (Modules, Controllers, Services)
- **TypeScript Fundamentals** - Sections 4-6 (Advanced types, Decorators)
- **PostgreSQL Complete Course** - Sections 2-4 (Schema design, Relationships)

### Practice Projects
- Build a simple blog API with authentication
- Create a todo app with user management
- Implement a basic e-commerce API without payment

---

## ✅ Success Criteria

### Week 1 Goals
- [ ] Set up NestJS project with TypeORM
- [ ] Implement user authentication system
- [ ] Create product and category management
- [ ] Build basic CRUD operations
- [ ] Write unit tests for core services

### Week 2 Goals
- [ ] Implement shopping cart functionality
- [ ] Create order management system
- [ ] Add comprehensive API documentation
- [ ] Implement caching with Redis
- [ ] Deploy to AWS/Heroku
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional RESTful API
- [ ] Comprehensive API documentation
- [ ] 80%+ test coverage
- [ ] Deployed application with live demo
- [ ] GitHub repository with clean commit history
- [ ] README with setup instructions

---

## 🔗 Related Projects
- **Next Project**: [Project 2: Social Media Backend API](../Backend_TypeScript/Project_2_Social_Media_API/)
- **Training Plan**: [Backend TypeScript Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive/)
- **Main README**: [Repository Overview](../../README)
