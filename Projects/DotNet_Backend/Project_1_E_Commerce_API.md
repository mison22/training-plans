# Project 1: E-commerce API
*2 weeks | C# fundamentals & ASP.NET Core*

## Project Overview
Build a comprehensive e-commerce API using C# and ASP.NET Core. This project focuses on C# language fundamentals, ASP.NET Core framework, Entity Framework Core, and creating a production-ready RESTful API for an online store.

## Learning Objectives
- Master C# language features and best practices
- Learn ASP.NET Core framework for web development
- Implement Entity Framework Core for database operations
- Practice RESTful API design and implementation
- Understand authentication and authorization
- Learn about API documentation and testing

## Technical Requirements

### **Core Technologies**
- **Language**: C# 12
- **Framework**: ASP.NET Core 8
- **Database**: SQL Server 2022
- **ORM**: Entity Framework Core 8
- **Authentication**: JWT
- **Documentation**: Swagger/OpenAPI

### **Project Structure**
```
ECommerceAPI/
├── src/
│   ├── ECommerceAPI/
│   │   ├── Controllers/
│   │   ├── Models/
│   │   ├── Services/
│   │   ├── Data/
│   │   ├── DTOs/
│   │   ├── Middleware/
│   │   └── Program.cs
│   └── ECommerceAPI.Tests/
│       ├── Controllers/
│       ├── Services/
│       └── Integration/
├── tests/
├── docs/
├── docker-compose.yml
├── Dockerfile
├── ECommerceAPI.sln
└── README.md
```

## Week 1: C# Fundamentals & Project Setup

### **Day 1-2: C# Language Mastery**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: C# language features
- **10:00 AM - 11:00 AM**: Object-oriented programming
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Project setup and implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- C# syntax and modern features
- Object-oriented programming principles
- LINQ and collections
- Async/await patterns
- Exception handling
- Generics and delegates

**Implementation:**
- Set up .NET 8 project
- Create domain models
- Implement business logic
- Add data validation

**Resources:**
- [C# Documentation](https://docs.microsoft.com/en-us/dotnet/csharp/) - Language reference
- [C# Programming Guide](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/) - Programming guide
- **Udemy**: [Complete C# Developer Course](https://www.udemy.com/course/complete-csharp-developer-course/) (Sections 1-3)

### **Day 3-4: ASP.NET Core Framework**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: ASP.NET Core concepts
- **10:00 AM - 11:00 AM**: Web API development
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: API implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- ASP.NET Core architecture
- Dependency injection
- Middleware pipeline
- Controllers and actions
- Model binding and validation
- Configuration management

**Implementation:**
- Create API controllers
- Implement CRUD operations
- Add request validation
- Set up dependency injection

**Resources:**
- [ASP.NET Core Documentation](https://docs.microsoft.com/en-us/aspnet/core/) - Framework reference
- [Web API Tutorial](https://docs.microsoft.com/en-us/aspnet/core/tutorials/first-web-api) - API tutorial
- **Udemy**: [ASP.NET Core Web API](https://www.udemy.com/course/aspnet-core-web-api/) (Sections 1-3)

### **Day 5: Entity Framework Core**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Entity Framework concepts
- **10:00 AM - 11:00 AM**: Database modeling
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Database implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- Entity Framework Core basics
- Code-first approach
- Database context and models
- Migrations and seeding
- Query optimization
- Repository pattern

**Implementation:**
- Set up Entity Framework
- Create database models
- Implement repository pattern
- Add database migrations

**Resources:**
- [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/) - ORM reference
- [EF Core Tutorial](https://docs.microsoft.com/en-us/ef/core/get-started/overview/first-app) - Getting started
- **Udemy**: [Entity Framework Core](https://www.udemy.com/course/entity-framework-core/) (Sections 1-3)

## Week 2: Advanced Features & Production

### **Day 1-2: Authentication & Security**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Authentication concepts
- **10:00 AM - 11:00 AM**: JWT implementation
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Security implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- JWT authentication
- ASP.NET Core Identity
- Role-based authorization
- Password hashing
- Security middleware
- API security best practices

**Implementation:**
- Implement JWT authentication
- Add user registration/login
- Set up authorization policies
- Add security middleware

**Resources:**
- [ASP.NET Core Security](https://docs.microsoft.com/en-us/aspnet/core/security/) - Security reference
- [JWT Documentation](https://jwt.io/introduction/) - Token authentication
- **Udemy**: [ASP.NET Core Security](https://www.udemy.com/course/aspnet-core-security/) (Sections 1-3)

### **Day 3-4: API Design & Documentation**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: API design principles
- **10:00 AM - 11:00 AM**: Swagger integration
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: API documentation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- RESTful API design
- HTTP methods and status codes
- API versioning
- Swagger/OpenAPI integration
- Error handling patterns
- Response formatting

**Implementation:**
- Design RESTful endpoints
- Add Swagger documentation
- Implement error handling
- Add API versioning

**Resources:**
- [OpenAPI Specification](https://swagger.io/specification/) - API documentation
- [ASP.NET Core Web API](https://docs.microsoft.com/en-us/aspnet/core/web-api/) - Web API guide
- **Udemy**: [REST API Design](https://www.udemy.com/course/rest-api-design/) (Sections 1-3)

### **Day 5: Testing & Deployment**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Testing concepts
- **10:00 AM - 11:00 AM**: Unit testing
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Testing and deployment
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- xUnit testing framework
- Unit testing patterns
- Integration testing
- Test-driven development
- Docker containerization
- Azure deployment

**Implementation:**
- Write comprehensive tests
- Set up test database
- Create Docker containers
- Deploy to Azure

**Resources:**
- [xUnit Documentation](https://xunit.net/) - Testing framework
- [Docker Documentation](https://docs.docker.com/) - Containerization
- **Udemy**: [Unit Testing in C#](https://www.udemy.com/course/unit-testing-csharp/) (Sections 1-3)

## API Endpoints

### **Authentication**
```
POST   /api/auth/register    - User registration
POST   /api/auth/login       - User login
POST   /api/auth/refresh     - Refresh token
POST   /api/auth/logout      - User logout
```

### **Products**
```
GET    /api/products         - Get all products
POST   /api/products         - Create product
GET    /api/products/{id}    - Get product by ID
PUT    /api/products/{id}    - Update product
DELETE /api/products/{id}    - Delete product
GET    /api/products/search  - Search products
```

### **Categories**
```
GET    /api/categories       - Get all categories
POST   /api/categories       - Create category
GET    /api/categories/{id}  - Get category by ID
PUT    /api/categories/{id}  - Update category
DELETE /api/categories/{id}  - Delete category
```

### **Orders**
```
GET    /api/orders           - Get user orders
POST   /api/orders           - Create order
GET    /api/orders/{id}      - Get order by ID
PUT    /api/orders/{id}      - Update order
DELETE /api/orders/{id}      - Cancel order
```

### **Cart**
```
GET    /api/cart             - Get cart items
POST   /api/cart/add         - Add item to cart
PUT    /api/cart/{id}        - Update cart item
DELETE /api/cart/{id}        - Remove item from cart
POST   /api/cart/clear       - Clear cart
```

## Database Schema

### **Users Table**
```sql
CREATE TABLE Users (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Email NVARCHAR(255) UNIQUE NOT NULL,
    PasswordHash NVARCHAR(255) NOT NULL,
    FirstName NVARCHAR(100) NOT NULL,
    LastName NVARCHAR(100) NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE()
);
```

### **Products Table**
```sql
CREATE TABLE Products (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Name NVARCHAR(200) NOT NULL,
    Description NVARCHAR(MAX),
    Price DECIMAL(10,2) NOT NULL,
    CategoryId UNIQUEIDENTIFIER REFERENCES Categories(Id),
    StockQuantity INT DEFAULT 0,
    IsActive BIT DEFAULT 1,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE()
);
```

### **Categories Table**
```sql
CREATE TABLE Categories (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Name NVARCHAR(100) UNIQUE NOT NULL,
    Description NVARCHAR(500),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE()
);
```

### **Orders Table**
```sql
CREATE TABLE Orders (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    UserId UNIQUEIDENTIFIER REFERENCES Users(Id),
    TotalAmount DECIMAL(10,2) NOT NULL,
    Status NVARCHAR(20) DEFAULT 'Pending',
    ShippingAddress NVARCHAR(MAX),
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE()
);
```

## Testing Strategy

### **Unit Tests**
- Test all service methods
- Mock external dependencies
- Test business logic
- Validate data processing

### **Integration Tests**
- Test API endpoints
- Test database operations
- Test authentication flow
- Test complete workflows

### **Test Coverage Goals**
- **Minimum**: 80% code coverage
- **Target**: 90% code coverage
- **Focus**: Critical business logic

## Deployment Checklist

### **Pre-deployment**
- [ ] All tests passing
- [ ] Code review completed
- [ ] Documentation updated
- [ ] Environment variables configured
- [ ] Database migrations ready

### **Production Setup**
- [ ] Docker containers built
- [ ] Database configured
- [ ] SSL certificates installed
- [ ] Monitoring configured
- [ ] Backup strategy implemented

### **Post-deployment**
- [ ] Health checks passing
- [ ] Performance monitoring active
- [ ] Error tracking configured
- [ ] Documentation accessible
- [ ] Team notified

## Success Criteria

### **Technical Requirements**
- [ ] Clean, maintainable C# code
- [ ] Comprehensive test coverage (90%+)
- [ ] Proper error handling and logging
- [ ] Security best practices implemented
- [ ] Performance optimization applied

### **API Requirements**
- [ ] RESTful API design
- [ ] Proper HTTP status codes
- [ ] Input validation and sanitization
- [ ] Rate limiting implemented
- [ ] CORS configured

### **Documentation Requirements**
- [ ] API documentation complete
- [ ] README with setup instructions
- [ ] Code comments and documentation
- [ ] Deployment guide
- [ ] Troubleshooting guide

## Next Steps

1. **Complete Project 1** with all requirements
2. **Deploy to production** and test thoroughly
3. **Document lessons learned** and challenges
4. **Prepare for Project 2** (Social Media API)
5. **Continue learning** C# advanced features

Ready to build your e-commerce API? Let's start coding! 🚀
