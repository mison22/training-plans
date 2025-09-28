# Project 1: Personal Portfolio API
*3 weeks | C# fundamentals & ASP.NET Core*

## Project Overview
Build a personal portfolio API using C# and ASP.NET Core. This project focuses on C# language fundamentals, ASP.NET Core framework, Entity Framework Core, and creating a RESTful API that showcases your skills and projects.

## Learning Objectives
- Master C# language syntax and features
- Learn ASP.NET Core framework for web development
- Implement basic CRUD operations
- Understand C# project structure and packages
- Practice error handling and logging
- Learn about C# testing patterns

## Technical Requirements

### **Core Technologies**
- **Language**: C# 12
- **Framework**: ASP.NET Core 8
- **Database**: SQL Server 2022
- **ORM**: Entity Framework Core 8
- **Authentication**: JWT
- **Documentation**: Swagger/OpenAPI

## Week 1: C# Fundamentals & Project Setup

### **Day 1-2: C# Language Basics**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: C# syntax and features
- **8:00 PM - 9:00 PM**: Set up project structure
- **9:00 PM - 9:30 PM**: Review and practice

**Learning Content:**
- C# syntax and data types
- Object-oriented programming
- LINQ and collections
- Async/await patterns
- Error handling
- Package management with NuGet

**Implementation:**
- Set up .NET 8 project
- Create basic project structure
- Implement utility functions
- Write basic tests

### **Day 3-4: ASP.NET Core Introduction**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: ASP.NET Core concepts
- **8:00 PM - 9:00 PM**: Build basic API endpoints
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- ASP.NET Core overview
- Dependency injection
- Middleware pipeline
- Controllers and actions
- Model binding and validation
- Configuration management

**Implementation:**
- Set up ASP.NET Core server
- Create basic routes
- Implement request/response models
- Add automatic documentation

### **Day 5: Database Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Database concepts and Entity Framework
- **8:00 PM - 9:00 PM**: Set up database connection
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- SQL Server setup
- Entity Framework Core basics
- Database models and relationships
- Connection management
- Basic queries

**Implementation:**
- Set up SQL Server
- Configure Entity Framework
- Create database models
- Implement basic CRUD operations

## Week 2: API Development & Authentication

### **Day 1-2: User Management System**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: User authentication concepts
- **8:00 PM - 9:00 PM**: Implement user registration/login
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- JWT authentication
- Password hashing with BCrypt
- User registration and login
- Session management
- Security best practices

**Implementation:**
- Create user model
- Implement registration endpoint
- Add login functionality
- Set up JWT middleware

### **Day 3-4: Portfolio Management**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Portfolio data modeling
- **8:00 PM - 9:00 PM**: Implement portfolio CRUD operations
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Data modeling for portfolio
- CRUD operations with Entity Framework
- Data validation with Data Annotations
- Error handling
- API response formatting

**Implementation:**
- Create portfolio models
- Implement CRUD endpoints
- Add data validation
- Create API responses

### **Day 5: Project Management**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Project data modeling
- **8:00 PM - 9:00 PM**: Implement project management
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Project data structure
- File upload handling
- Image processing
- Data relationships
- Query optimization

**Implementation:**
- Create project models
- Implement project endpoints
- Add file upload functionality
- Create project-portfolio relationships

## Week 3: Testing, Documentation & Deployment

### **Day 1-2: Testing Implementation**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: C# testing concepts
- **8:00 PM - 9:00 PM**: Write comprehensive tests
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- xUnit testing framework
- Unit testing patterns
- Integration testing
- Mocking strategies
- Test coverage

**Implementation:**
- Write unit tests for services
- Create integration tests for APIs
- Implement test database setup
- Add test coverage reporting

### **Day 3-4: API Documentation**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: API documentation concepts
- **8:00 PM - 9:00 PM**: Implement comprehensive documentation
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- OpenAPI/Swagger specification
- API documentation best practices
- Interactive documentation
- Code generation
- Documentation maintenance

**Implementation:**
- Set up automatic documentation
- Add API descriptions
- Create example responses
- Generate client SDKs

### **Day 5: Deployment & Production**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Deployment concepts
- **8:00 PM - 9:00 PM**: Set up production deployment
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Docker containerization
- Production configuration
- Environment management
- Database migrations
- Monitoring and logging

**Implementation:**
- Create Dockerfile
- Set up docker-compose
- Configure production settings
- Deploy to Azure

## API Endpoints

### **Authentication**
```
POST   /api/auth/register    - User registration
POST   /api/auth/login       - User login
POST   /api/auth/refresh     - Refresh token
POST   /api/auth/logout      - User logout
```

### **User Management**
```
GET    /api/users/me         - Get current user profile
PUT    /api/users/me         - Update current user profile
DELETE /api/users/me         - Delete current user account
```

### **Portfolio Management**
```
GET    /api/portfolios       - Get all portfolios
POST   /api/portfolios       - Create portfolio
GET    /api/portfolios/{id}  - Get portfolio by ID
PUT    /api/portfolios/{id}  - Update portfolio
DELETE /api/portfolios/{id}  - Delete portfolio
```

### **Project Management**
```
GET    /api/projects         - Get all projects
POST   /api/projects         - Create project
GET    /api/projects/{id}    - Get project by ID
PUT    /api/projects/{id}    - Update project
DELETE /api/projects/{id}    - Delete project
POST   /api/projects/{id}/upload - Upload project files
```

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

Ready to build your personal portfolio API? Let's start coding! 🚀
