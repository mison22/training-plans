# Project 1: Personal Portfolio API
*3 weeks | Go fundamentals & Gin framework*

## Project Overview
Build a personal portfolio API using Go and the Gin framework. This project focuses on Go language fundamentals, basic web development, and creating a RESTful API that showcases your skills and projects.

## Learning Objectives
- Master Go language syntax and features
- Learn Gin framework for web development
- Implement basic CRUD operations
- Understand Go project structure and modules
- Practice error handling and logging
- Learn about Go testing patterns

## Technical Requirements

### **Core Technologies**
- **Language**: Go 1.21+
- **Framework**: Gin
- **Database**: PostgreSQL
- **ORM**: GORM
- **Authentication**: JWT
- **Documentation**: Swagger

### **Project Structure**
```
portfolio-api/
├── cmd/
│   └── server/
│       └── main.go
├── internal/
│   ├── config/
│   ├── handlers/
│   ├── middleware/
│   ├── models/
│   ├── repository/
│   └── services/
├── pkg/
│   ├── database/
│   ├── jwt/
│   └── utils/
├── api/
│   └── docs/
├── tests/
├── docker-compose.yml
├── Dockerfile
├── go.mod
├── go.sum
└── README.md
```

## Week 1: Go Fundamentals & Project Setup

### **Day 1-2: Go Language Basics**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Go syntax and features
- **8:00 PM - 9:00 PM**: Set up project structure
- **9:00 PM - 9:30 PM**: Review and practice

**Learning Content:**
- Go syntax and data types
- Functions and methods
- Structs and interfaces
- Pointers and memory management
- Error handling patterns
- Package system and modules

**Implementation:**
- Set up Go module
- Create basic project structure
- Implement utility functions
- Write basic tests

**Resources:**
- [Go Tour](https://tour.golang.org/) - Interactive tutorial
- [Go by Example](https://gobyexample.com/) - Code examples
- [Effective Go](https://golang.org/doc/effective_go.html) - Best practices

### **Day 3-4: Gin Framework Introduction**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Gin framework concepts
- **8:00 PM - 9:00 PM**: Build basic API endpoints
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Gin framework overview
- Routing and middleware
- Request/response handling
- JSON binding and validation
- Error handling in Gin

**Implementation:**
- Set up Gin server
- Create basic routes
- Implement middleware
- Add request validation

**Resources:**
- [Gin Documentation](https://gin-gonic.com/docs/) - Official docs
- [Gin Examples](https://github.com/gin-gonic/examples) - Code examples
- [Gin Best Practices](https://gin-gonic.com/docs/best-practices/) - Best practices

### **Day 5: Database Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Database concepts and GORM
- **8:00 PM - 9:00 PM**: Set up database connection
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- PostgreSQL setup
- GORM ORM basics
- Database models and migrations
- Connection management
- Basic queries

**Implementation:**
- Set up PostgreSQL
- Configure GORM
- Create database models
- Implement basic CRUD operations

**Resources:**
- [GORM Documentation](https://gorm.io/docs/) - ORM reference
- [PostgreSQL Documentation](https://www.postgresql.org/docs/) - Database reference
- [GORM Examples](https://gorm.io/examples/) - Code examples

## Week 2: API Development & Authentication

### **Day 1-2: User Management System**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: User authentication concepts
- **8:00 PM - 9:00 PM**: Implement user registration/login
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- JWT authentication
- Password hashing
- User registration and login
- Session management
- Security best practices

**Implementation:**
- Create user model
- Implement registration endpoint
- Add login functionality
- Set up JWT middleware

**Resources:**
- [JWT Documentation](https://jwt.io/introduction/) - Token authentication
- [bcrypt Package](https://pkg.go.dev/golang.org/x/crypto/bcrypt) - Password hashing
- [JWT-Go Package](https://github.com/golang-jwt/jwt) - JWT implementation

### **Day 3-4: Portfolio Management**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Portfolio data modeling
- **8:00 PM - 9:00 PM**: Implement portfolio CRUD operations
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Data modeling for portfolio
- CRUD operations
- Data validation
- Error handling
- API response formatting

**Implementation:**
- Create portfolio models
- Implement CRUD endpoints
- Add data validation
- Create API responses

**Resources:**
- [Go Validator](https://github.com/go-playground/validator) - Data validation
- [GORM CRUD](https://gorm.io/docs/create.html) - Database operations
- [Gin Binding](https://gin-gonic.com/docs/binding/) - Request binding

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

**Resources:**
- [Gin File Upload](https://gin-gonic.com/docs/examples/upload-file/) - File handling
- [GORM Associations](https://gorm.io/docs/associations.html) - Data relationships
- [Go Image Package](https://pkg.go.dev/image) - Image processing

## Week 3: Testing, Documentation & Deployment

### **Day 1-2: Testing Implementation**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Go testing concepts
- **8:00 PM - 9:00 PM**: Write comprehensive tests
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Go testing framework
- Unit testing patterns
- Integration testing
- Mocking strategies
- Test coverage

**Implementation:**
- Write unit tests for services
- Create integration tests for APIs
- Implement test database setup
- Add test coverage reporting

**Resources:**
- [Go Testing](https://golang.org/pkg/testing/) - Testing framework
- [Testify Package](https://github.com/stretchr/testify) - Testing utilities
- [GoMock](https://github.com/golang/mock) - Mocking framework

### **Day 3-4: API Documentation**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: API documentation concepts
- **8:00 PM - 9:00 PM**: Implement Swagger documentation
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- OpenAPI/Swagger specification
- API documentation best practices
- Interactive documentation
- Code generation
- Documentation maintenance

**Implementation:**
- Set up Swagger documentation
- Add API annotations
- Generate documentation
- Create interactive API explorer

**Resources:**
- [Swaggo](https://github.com/swaggo/swag) - Swagger generation
- [OpenAPI Specification](https://swagger.io/specification/) - API docs
- [Gin Swagger](https://github.com/swaggo/gin-swagger) - Gin integration

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
- Deploy to cloud platform

**Resources:**
- [Docker Documentation](https://docs.docker.com/) - Containerization
- [Docker Compose](https://docs.docker.com/compose/) - Multi-container apps
- [Go Docker](https://docs.docker.com/language/golang/) - Go containerization

## API Endpoints

### **Authentication**
```
POST   /api/v1/auth/register    - User registration
POST   /api/v1/auth/login       - User login
POST   /api/v1/auth/refresh     - Refresh token
POST   /api/v1/auth/logout      - User logout
```

### **User Management**
```
GET    /api/v1/users/profile    - Get user profile
PUT    /api/v1/users/profile    - Update user profile
DELETE /api/v1/users/profile    - Delete user account
```

### **Portfolio Management**
```
GET    /api/v1/portfolios       - Get all portfolios
POST   /api/v1/portfolios       - Create portfolio
GET    /api/v1/portfolios/:id   - Get portfolio by ID
PUT    /api/v1/portfolios/:id   - Update portfolio
DELETE /api/v1/portfolios/:id   - Delete portfolio
```

### **Project Management**
```
GET    /api/v1/projects         - Get all projects
POST   /api/v1/projects         - Create project
GET    /api/v1/projects/:id     - Get project by ID
PUT    /api/v1/projects/:id     - Update project
DELETE /api/v1/projects/:id     - Delete project
POST   /api/v1/projects/:id/upload - Upload project files
```

## Database Schema

### **Users Table**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    bio TEXT,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Portfolios Table**
```sql
CREATE TABLE portfolios (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    website_url VARCHAR(500),
    github_url VARCHAR(500),
    is_public BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### **Projects Table**
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    portfolio_id UUID REFERENCES portfolios(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    description TEXT,
    technologies TEXT[],
    github_url VARCHAR(500),
    demo_url VARCHAR(500),
    image_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## Testing Strategy

### **Unit Tests**
- Test all service functions
- Mock external dependencies
- Test error handling
- Validate business logic

### **Integration Tests**
- Test API endpoints
- Test database operations
- Test authentication flow
- Test file upload functionality

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
- [ ] Clean, maintainable Go code
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
4. **Prepare for Project 2** (Blog Management System)
5. **Continue learning** Go advanced features

Ready to build your personal portfolio API? Let's start coding! 🚀
