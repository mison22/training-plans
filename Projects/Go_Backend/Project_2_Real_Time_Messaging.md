# Project 2: Real-time Messaging System
*Build a scalable real-time messaging platform using Go, WebSockets, and gRPC*

## Project Overview
Create a production-ready real-time messaging system with WebSocket connections, gRPC microservices, and event-driven architecture. This project focuses on concurrent programming, real-time communication, and distributed systems.

## Learning Objectives
- Master Go concurrency with goroutines and channels
- Implement WebSocket connections and real-time communication
- Build gRPC microservices for inter-service communication
- Implement event-driven architecture with message queues
- Deploy to Kubernetes with proper scaling

## Technology Stack
- **Language**: Go 1.21+
- **WebSockets**: Gorilla WebSocket
- **gRPC**: Protocol Buffers
- **Message Queue**: NATS
- **Database**: MongoDB
- **Containerization**: Docker
- **Orchestration**: Kubernetes

## Architecture Overview

### **Microservices**
1. **WebSocket Gateway** - WebSocket connection management
2. **User Service** - User management and authentication
3. **Chat Service** - Message handling and storage
4. **Notification Service** - Push notifications
5. **Presence Service** - User online/offline status
6. **Message Queue Service** - Event processing

### **Real-time Flow**
```
WebSocket Client → Gateway → Chat Service → MongoDB
                        ↓
                   Message Queue → Notification Service
```

## Detailed Requirements

### **Phase 1: WebSocket Gateway (Week 1, Days 1-3)**

#### **Core Features**
- WebSocket connection management
- Connection pooling and load balancing
- Message routing and broadcasting
- Connection authentication
- Heartbeat and ping/pong handling

#### **WebSocket Events**
```go
type Message struct {
    Type      string      `json:"type"`
    Channel   string      `json:"channel"`
    Data      interface{} `json:"data"`
    Timestamp int64       `json:"timestamp"`
    UserID    string      `json:"user_id"`
}

// Event Types
const (
    EventJoinChannel    = "join_channel"
    EventLeaveChannel   = "leave_channel"
    EventSendMessage    = "send_message"
    EventUserTyping     = "user_typing"
    EventUserOnline     = "user_online"
    EventUserOffline    = "user_offline"
)
```

#### **Implementation Requirements**
- Concurrent connection handling
- Message broadcasting to channels
- Connection state management
- Error handling and reconnection
- Rate limiting per connection

### **Phase 2: User Service (Week 1, Days 4-5)**

#### **Core Features**
- User registration and authentication
- JWT token management
- User profile management
- Friend system and contacts
- User search functionality

#### **gRPC Service Definition**
```protobuf
service UserService {
    rpc Register(RegisterRequest) returns (RegisterResponse);
    rpc Login(LoginRequest) returns (LoginResponse);
    rpc GetProfile(GetProfileRequest) returns (GetProfileResponse);
    rpc UpdateProfile(UpdateProfileRequest) returns (UpdateProfileResponse);
    rpc SearchUsers(SearchUsersRequest) returns (SearchUsersResponse);
    rpc AddFriend(AddFriendRequest) returns (AddFriendResponse);
}
```

#### **Database Schema (MongoDB)**
```go
type User struct {
    ID          primitive.ObjectID `bson:"_id,omitempty" json:"id"`
    Username    string             `bson:"username" json:"username"`
    Email       string             `bson:"email" json:"email"`
    Password    string             `bson:"password" json:"-"`
    DisplayName string             `bson:"display_name" json:"display_name"`
    Avatar      string             `bson:"avatar" json:"avatar"`
    Status      string             `bson:"status" json:"status"` // online, offline, away
    LastSeen    time.Time          `bson:"last_seen" json:"last_seen"`
    CreatedAt   time.Time          `bson:"created_at" json:"created_at"`
    UpdatedAt   time.Time          `bson:"updated_at" json:"updated_at"`
}
```

### **Phase 3: Chat Service (Week 1, Days 6-7)**

#### **Core Features**
- Message storage and retrieval
- Channel management (private, group, public)
- Message history and pagination
- Message search functionality
- File and media sharing

#### **gRPC Service Definition**
```protobuf
service ChatService {
    rpc SendMessage(SendMessageRequest) returns (SendMessageResponse);
    rpc GetMessages(GetMessagesRequest) returns (GetMessagesResponse);
    rpc CreateChannel(CreateChannelRequest) returns (CreateChannelResponse);
    rpc JoinChannel(JoinChannelRequest) returns (JoinChannelResponse);
    rpc LeaveChannel(LeaveChannelRequest) returns (LeaveChannelResponse);
    rpc SearchMessages(SearchMessagesRequest) returns (SearchMessagesResponse);
}
```

#### **Database Schema (MongoDB)**
```go
type Channel struct {
    ID          primitive.ObjectID `bson:"_id,omitempty" json:"id"`
    Name        string             `bson:"name" json:"name"`
    Type        string             `bson:"type" json:"type"` // private, group, public
    Members     []string           `bson:"members" json:"members"`
    CreatedBy   string             `bson:"created_by" json:"created_by"`
    CreatedAt   time.Time          `bson:"created_at" json:"created_at"`
}

type Message struct {
    ID        primitive.ObjectID `bson:"_id,omitempty" json:"id"`
    ChannelID string             `bson:"channel_id" json:"channel_id"`
    UserID    string             `bson:"user_id" json:"user_id"`
    Content   string             `bson:"content" json:"content"`
    Type      string             `bson:"type" json:"type"` // text, image, file
    Metadata  map[string]interface{} `bson:"metadata" json:"metadata"`
    CreatedAt time.Time          `bson:"created_at" json:"created_at"`
}
```

### **Phase 4: Notification Service (Week 2, Days 1-3)**

#### **Core Features**
- Push notification delivery
- Email notifications
- SMS notifications (optional)
- Notification preferences
- Notification history

#### **gRPC Service Definition**
```protobuf
service NotificationService {
    rpc SendPushNotification(SendPushNotificationRequest) returns (SendPushNotificationResponse);
    rpc SendEmail(SendEmailRequest) returns (SendEmailResponse);
    rpc GetNotificationHistory(GetNotificationHistoryRequest) returns (GetNotificationHistoryResponse);
    rpc UpdateNotificationPreferences(UpdateNotificationPreferencesRequest) returns (UpdateNotificationPreferencesResponse);
}
```

#### **Implementation Requirements**
- Integration with Firebase Cloud Messaging
- Email service integration (SendGrid/AWS SES)
- Notification queuing and retry logic
- User preference management
- Delivery status tracking

### **Phase 5: Presence Service (Week 2, Days 4-5)**

#### **Core Features**
- User online/offline status tracking
- Last seen timestamps
- User activity monitoring
- Status updates (online, away, busy)
- Presence broadcasting

#### **gRPC Service Definition**
```protobuf
service PresenceService {
    rpc UpdateStatus(UpdateStatusRequest) returns (UpdateStatusResponse);
    rpc GetUserStatus(GetUserStatusRequest) returns (GetUserStatusResponse);
    rpc GetOnlineUsers(GetOnlineUsersRequest) returns (GetOnlineUsersResponse);
    rpc SubscribeToPresence(SubscribeToPresenceRequest) returns (stream PresenceUpdate);
}
```

#### **Implementation Requirements**
- Redis for real-time status storage
- WebSocket connection tracking
- Heartbeat mechanism
- Status change broadcasting
- Cleanup of stale connections

### **Phase 6: Message Queue Integration (Week 2, Days 6-7)**

#### **Core Features**
- Event publishing and subscription
- Message queuing and processing
- Event sourcing for audit trails
- Dead letter queue handling
- Message persistence

#### **Event Types**
```go
type Event struct {
    ID        string                 `json:"id"`
    Type      string                 `json:"type"`
    Source    string                 `json:"source"`
    Data      map[string]interface{} `json:"data"`
    Timestamp time.Time              `json:"timestamp"`
    Version   int                    `json:"version"`
}

// Event Types
const (
    EventUserJoined     = "user.joined"
    EventUserLeft       = "user.left"
    EventMessageSent    = "message.sent"
    EventChannelCreated = "channel.created"
    EventUserTyping     = "user.typing"
)
```

## Technical Requirements

### **Concurrency & Performance**
- Efficient goroutine management
- Channel-based communication
- Connection pooling
- Memory optimization
- CPU profiling and optimization

### **Real-time Features**
- WebSocket connection management
- Message broadcasting
- Presence tracking
- Typing indicators
- Message delivery status

### **Scalability**
- Horizontal scaling with Kubernetes
- Load balancing across instances
- Database sharding strategies
- Caching with Redis
- Message queue partitioning

### **Testing**
- Unit tests for all services
- Integration tests for gRPC calls
- WebSocket connection testing
- Load testing with concurrent users
- End-to-end testing scenarios

## Deployment Requirements

### **Kubernetes Deployment**
- Deployment manifests for each service
- Service discovery and load balancing
- ConfigMaps and Secrets management
- Horizontal Pod Autoscaling
- Resource limits and requests

### **Monitoring & Observability**
- Prometheus metrics collection
- Grafana dashboards
- Distributed tracing with Jaeger
- Log aggregation with ELK stack
- Health check endpoints

### **Security**
- TLS encryption for all connections
- JWT token validation
- Rate limiting and DDoS protection
- Input validation and sanitization
- Secure WebSocket connections

## Deliverables

### **Code Repository**
- Microservices with gRPC definitions
- WebSocket gateway implementation
- Message queue integration
- Kubernetes deployment manifests
- Comprehensive test suites

### **Documentation**
- API documentation (gRPC and WebSocket)
- Architecture diagrams
- Deployment guide
- Performance benchmarks
- Security audit report

### **Demo Application**
- Real-time chat application
- Mobile app integration
- Admin dashboard
- Performance monitoring
- Load testing results

## Success Criteria

### **Functional Requirements**
- [ ] Real-time messaging with WebSockets
- [ ] User authentication and management
- [ ] Channel and group management
- [ ] Push notifications
- [ ] Presence tracking
- [ ] Message history and search

### **Technical Requirements**
- [ ] gRPC microservices architecture
- [ ] Event-driven design
- [ ] Kubernetes deployment
- [ ] Message queue integration
- [ ] Comprehensive testing
- [ ] Performance optimization

### **Quality Requirements**
- [ ] 90%+ test coverage
- [ ] Security best practices
- [ ] Scalability testing
- [ ] Error handling
- [ ] Documentation completeness

## Timeline

### **Week 1**
- **Days 1-3**: WebSocket Gateway
- **Days 4-5**: User Service
- **Days 6-7**: Chat Service

### **Week 2**
- **Days 1-3**: Notification Service
- **Days 4-5**: Presence Service
- **Days 6-7**: Message Queue Integration

### **Milestones**
- **Day 3**: WebSocket connections working
- **Day 5**: User authentication complete
- **Day 7**: Basic messaging functional
- **Day 10**: Notifications working
- **Day 12**: Presence tracking complete
- **Day 14**: Full platform deployed

## Next Steps

1. **Set up development environment**
2. **Implement WebSocket gateway**
3. **Build User Service with gRPC**
4. **Create Chat Service with MongoDB**
5. **Add Notification Service**
6. **Implement Presence Service**
7. **Deploy to Kubernetes and test**

Ready to build your real-time messaging system? Let's start coding! 🚀
