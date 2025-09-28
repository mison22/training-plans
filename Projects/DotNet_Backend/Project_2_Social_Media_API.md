# Project 2: Social Media API
*2 weeks | Advanced features & real-time communication*

## Project Overview
Build a comprehensive social media API using C# and ASP.NET Core. This project focuses on advanced features, real-time communication with SignalR, caching with Redis, and creating a scalable social platform.

## Learning Objectives
- Master advanced ASP.NET Core features
- Implement real-time communication with SignalR
- Learn caching strategies with Redis
- Practice microservices patterns
- Understand event-driven architecture
- Learn about performance optimization

## Technical Requirements

### **Core Technologies**
- **Language**: C# 12
- **Framework**: ASP.NET Core 8
- **Database**: SQL Server 2022
- **ORM**: Entity Framework Core 8
- **Cache**: Redis
- **Real-time**: SignalR
- **Authentication**: JWT

## Week 1: Advanced Features & Real-time

### **Day 1-2: SignalR & Real-time Features**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: SignalR concepts
- **10:00 AM - 11:00 AM**: Real-time implementation
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Real-time features
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- SignalR overview and architecture
- Hub implementation
- Client connections and groups
- Real-time messaging
- Authentication with SignalR
- Scaling considerations

**Implementation:**
- Set up SignalR
- Create chat functionality
- Implement live notifications
- Add real-time updates

### **Day 3-4: Redis Caching & Performance**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Redis concepts
- **10:00 AM - 11:00 AM**: Caching implementation
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Performance optimization
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- Redis data structures
- Caching strategies
- Session management
- Performance optimization
- Memory management
- Load testing

**Implementation:**
- Integrate Redis
- Implement caching layers
- Add session management
- Optimize performance

### **Day 5: Advanced API Features**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Advanced API concepts
- **10:00 AM - 11:00 AM**: File upload and processing
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Advanced features
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- File upload handling
- Image processing
- Background services
- Message queues
- Event sourcing
- CQRS pattern

**Implementation:**
- Add file upload
- Implement background services
- Create event system
- Add message queuing

## Week 2: Microservices & Production

### **Day 1-2: Microservices Architecture**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Microservices concepts
- **10:00 AM - 11:00 AM**: Service design
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Microservices implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- Microservices patterns
- Service communication
- API Gateway
- Service discovery
- Circuit breaker
- Event-driven architecture

**Implementation:**
- Design microservices
- Implement service communication
- Add API Gateway
- Create event system

### **Day 3-4: Cloud Deployment**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Cloud concepts
- **10:00 AM - 11:00 AM**: Azure services
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Cloud deployment
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- Azure App Service
- Azure Functions
- Azure Service Bus
- Azure Storage
- Container orchestration
- CI/CD pipelines

**Implementation:**
- Deploy to Azure
- Set up Azure services
- Configure CI/CD
- Add monitoring

### **Day 5: Testing & Quality Assurance**
**Daily Schedule (8-10 hours):**
- **9:00 AM - 10:00 AM**: Testing strategies
- **10:00 AM - 11:00 AM**: Comprehensive testing
- **11:00 AM - 12:00 PM**: Hands-on practice
- **1:00 PM - 2:00 PM**: LeetCode practice
- **2:00 PM - 3:00 PM**: System design study
- **3:00 PM - 5:00 PM**: Testing implementation
- **5:00 PM - 6:00 PM**: Review and documentation

**Learning Content:**
- Unit testing
- Integration testing
- End-to-end testing
- Performance testing
- Load testing
- Security testing

**Implementation:**
- Write comprehensive tests
- Set up test automation
- Add performance tests
- Implement security tests

## API Endpoints

### **Posts**
```
GET    /api/posts             - Get all posts
POST   /api/posts             - Create post
GET    /api/posts/{id}        - Get post by ID
PUT    /api/posts/{id}        - Update post
DELETE /api/posts/{id}        - Delete post
POST   /api/posts/{id}/like   - Like post
POST   /api/posts/{id}/comment - Add comment
```

### **Users**
```
GET    /api/users             - Get all users
GET    /api/users/{id}        - Get user by ID
PUT    /api/users/{id}        - Update user
POST   /api/users/{id}/follow - Follow user
POST   /api/users/{id}/unfollow - Unfollow user
GET    /api/users/{id}/followers - Get followers
GET    /api/users/{id}/following - Get following
```

### **Real-time Features**
```
WS     /hubs/chat             - Chat hub
WS     /hubs/notifications    - Notifications hub
```

## Success Criteria

### **Technical Requirements**
- [ ] Clean, maintainable C# code
- [ ] Comprehensive test coverage (90%+)
- [ ] Real-time features working
- [ ] Performance optimization applied
- [ ] Security best practices implemented

### **Feature Requirements**
- [ ] User management
- [ ] Post creation and management
- [ ] Real-time messaging
- [ ] Caching implementation
- [ ] Microservices architecture

Ready to build your social media API? Let's start coding! 🚀
