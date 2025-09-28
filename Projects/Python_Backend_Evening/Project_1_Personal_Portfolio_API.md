# Project 1: Personal Portfolio API
*3 weeks | Python fundamentals & FastAPI framework*

## Project Overview
Build a personal portfolio API using Python and the FastAPI framework. This project focuses on Python language fundamentals, modern web development, and creating a RESTful API that showcases your skills and projects.

## Learning Objectives
- Master Python language syntax and features
- Learn FastAPI framework for web development
- Implement basic CRUD operations
- Understand Python project structure and packaging
- Practice error handling and logging
- Learn about Python testing patterns

## Technical Requirements

### **Core Technologies**
- **Language**: Python 3.9+
- **Framework**: FastAPI
- **Database**: PostgreSQL
- **ORM**: SQLAlchemy
- **Authentication**: JWT
- **Documentation**: Swagger/OpenAPI

### **Project Structure**
```
portfolio-api/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── database.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── portfolio.py
│   │   └── project.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── portfolio.py
│   │   └── project.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── auth.py
│   │       ├── users.py
│   │       ├── portfolios.py
│   │       └── projects.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── security.py
│   │   └── config.py
│   └── crud/
│       ├── __init__.py
│       ├── user.py
│       ├── portfolio.py
│       └── project.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_auth.py
│   ├── test_users.py
│   ├── test_portfolios.py
│   └── test_projects.py
├── alembic/
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
├── requirements.txt
├── pyproject.toml
├── docker-compose.yml
├── Dockerfile
└── README.md
```

## Week 1: Python Fundamentals & Project Setup

### **Day 1-2: Python Language Basics**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Python syntax and features
- **8:00 PM - 9:00 PM**: Set up project structure
- **9:00 PM - 9:30 PM**: Review and practice

**Learning Content:**
- Python syntax and data types
- Functions and classes
- Decorators and context managers
- Error handling with try/except
- Package management with pip
- Virtual environments

**Implementation:**
- Set up Python virtual environment
- Create basic project structure
- Implement utility functions
- Write basic tests

**Resources:**
- [Python Tutorial](https://docs.python.org/3/tutorial/) - Official tutorial
- [PEP 8](https://pep8.org/) - Style guide
- [Python Best Practices](https://docs.python-guide.org/) - Best practices

### **Day 3-4: FastAPI Framework Introduction**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: FastAPI framework concepts
- **8:00 PM - 9:00 PM**: Build basic API endpoints
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- FastAPI framework overview
- Request/response models
- Dependency injection
- Automatic API documentation
- Type hints and validation

**Implementation:**
- Set up FastAPI server
- Create basic routes
- Implement request/response models
- Add automatic documentation

**Resources:**
- [FastAPI Documentation](https://fastapi.tiangolo.com/) - Official docs
- [FastAPI Examples](https://github.com/tiangolo/fastapi/tree/master/docs_src) - Code examples
- [FastAPI Best Practices](https://fastapi.tiangolo.com/tutorial/) - Best practices

### **Day 5: Database Integration**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Database concepts and SQLAlchemy
- **8:00 PM - 9:00 PM**: Set up database connection
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- PostgreSQL setup
- SQLAlchemy ORM basics
- Database models and relationships
- Connection management
- Basic queries

**Implementation:**
- Set up PostgreSQL
- Configure SQLAlchemy
- Create database models
- Implement basic CRUD operations

**Resources:**
- [SQLAlchemy Documentation](https://docs.sqlalchemy.org/) - ORM reference
- [PostgreSQL Documentation](https://www.postgresql.org/docs/) - Database reference
- [Alembic](https://alembic.sqlalchemy.org/) - Database migrations

## Week 2: API Development & Authentication

### **Day 1-2: User Management System**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: User authentication concepts
- **8:00 PM - 9:00 PM**: Implement user registration/login
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- JWT authentication
- Password hashing with bcrypt
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
- [bcrypt](https://pypi.org/project/bcrypt/) - Password hashing
- [python-jose](https://python-jose.readthedocs.io/) - JWT implementation

### **Day 3-4: Portfolio Management**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Portfolio data modeling
- **8:00 PM - 9:00 PM**: Implement portfolio CRUD operations
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Data modeling for portfolio
- CRUD operations with SQLAlchemy
- Data validation with Pydantic
- Error handling
- API response formatting

**Implementation:**
- Create portfolio models
- Implement CRUD endpoints
- Add data validation
- Create API responses

**Resources:**
- [Pydantic](https://pydantic-docs.helpmanual.io/) - Data validation
- [SQLAlchemy CRUD](https://docs.sqlalchemy.org/en/14/orm/tutorial/) - Database operations
- [FastAPI Dependencies](https://fastapi.tiangolo.com/tutorial/dependencies/) - Dependency injection

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
- [FastAPI File Upload](https://fastapi.tiangolo.com/tutorial/request-files/) - File handling
- [SQLAlchemy Relationships](https://docs.sqlalchemy.org/en/14/orm/basic_relationships/) - Data relationships
- [Pillow](https://pillow.readthedocs.io/) - Image processing

## Week 3: Testing, Documentation & Deployment

### **Day 1-2: Testing Implementation**
**Evening Schedule (2.5 hours):**
- **7:00 PM - 8:00 PM**: Python testing concepts
- **8:00 PM - 9:00 PM**: Write comprehensive tests
- **9:00 PM - 9:30 PM**: Review and testing

**Learning Content:**
- Python testing framework (pytest)
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
- [pytest Documentation](https://docs.pytest.org/) - Testing framework
- [pytest-asyncio](https://pytest-asyncio.readthedocs.io/) - Async testing
- [pytest-cov](https://pytest-cov.readthedocs.io/) - Coverage reporting

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

**Resources:**
- [FastAPI Documentation](https://fastapi.tiangolo.com/tutorial/metadata/) - API docs
- [OpenAPI Specification](https://swagger.io/specification/) - API docs
- [Swagger UI](https://swagger.io/tools/swagger-ui/) - Interactive docs

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
- [Python Docker](https://docs.docker.com/language/python/) - Python containerization

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
GET    /api/v1/users/me         - Get current user profile
PUT    /api/v1/users/me         - Update current user profile
DELETE /api/v1/users/me         - Delete current user account
```

### **Portfolio Management**
```
GET    /api/v1/portfolios       - Get all portfolios
POST   /api/v1/portfolios       - Create portfolio
GET    /api/v1/portfolios/{id}  - Get portfolio by ID
PUT    /api/v1/portfolios/{id}  - Update portfolio
DELETE /api/v1/portfolios/{id}  - Delete portfolio
```

### **Project Management**
```
GET    /api/v1/projects         - Get all projects
POST   /api/v1/projects         - Create project
GET    /api/v1/projects/{id}    - Get project by ID
PUT    /api/v1/projects/{id}    - Update project
DELETE /api/v1/projects/{id}    - Delete project
POST   /api/v1/projects/{id}/upload - Upload project files
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
- [ ] Clean, maintainable Python code
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
5. **Continue learning** Python advanced features

Ready to build your personal portfolio API? Let's start coding! 🚀
