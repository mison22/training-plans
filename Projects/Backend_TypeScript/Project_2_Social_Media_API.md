# Project 2: Social Media Backend API
*Backend TypeScript Intensive Plan - Weeks 3-4*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive/)** | 🏠 **[Back to README](../../README)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 3-4)  
**Tech Stack**: NestJS + GraphQL + MongoDB + Redis + Socket.io  
**Focus**: Real-time features, GraphQL APIs, and social interactions  

### Learning Objectives
- Master GraphQL schema design and resolvers
- Implement real-time features with WebSockets
- Build complex social media functionality
- Handle file uploads and media processing
- Implement advanced search and recommendation systems

---

## 📊 Database Schema Design (MongoDB)

### Users Collection
```javascript
{
  _id: ObjectId,
  email: String (unique, required),
  username: String (unique, required, lowercase),
  passwordHash: String (required),
  profile: {
    firstName: String (required),
    lastName: String (required),
    bio: String (max: 500),
    avatar: String, // URL to avatar image
    coverPhoto: String, // URL to cover photo
    location: String,
    website: String,
    birthDate: Date,
    isPublic: Boolean (default: true)
  },
  preferences: {
    emailNotifications: Boolean (default: true),
    pushNotifications: Boolean (default: true),
    profileVisibility: String (enum: ['public', 'followers', 'private']),
    allowMessages: String (enum: ['everyone', 'followers', 'none'])
  },
  stats: {
    followersCount: Number (default: 0),
    followingCount: Number (default: 0),
    postsCount: Number (default: 0),
    likesReceived: Number (default: 0)
  },
  isVerified: Boolean (default: false),
  isActive: Boolean (default: true),
  lastActive: Date,
  createdAt: Date,
  updatedAt: Date
}
```

### Posts Collection
```javascript
{
  _id: ObjectId,
  authorId: ObjectId (ref: 'User', required),
  content: {
    text: String (max: 2000),
    type: String (enum: ['text', 'image', 'video', 'poll']),
    media: [{
      url: String (required),
      type: String (enum: ['image', 'video']),
      altText: String,
      width: Number,
      height: Number,
      duration: Number // for videos
    }]
  },
  poll: {
    question: String,
    options: [{
      text: String,
      votes: Number (default: 0)
    }],
    expiresAt: Date,
    allowMultipleVotes: Boolean (default: false)
  },
  metadata: {
    likesCount: Number (default: 0),
    commentsCount: Number (default: 0),
    sharesCount: Number (default: 0),
    viewsCount: Number (default: 0)
  },
  visibility: String (enum: ['public', 'followers', 'private']),
  tags: [String], // Hashtags
  mentions: [ObjectId], // User IDs mentioned
  location: {
    name: String,
    coordinates: {
      type: 'Point',
      coordinates: [Number, Number] // [longitude, latitude]
    }
  },
  isEdited: Boolean (default: false),
  editedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

### Comments Collection
```javascript
{
  _id: ObjectId,
  postId: ObjectId (ref: 'Post', required),
  authorId: ObjectId (ref: 'User', required),
  parentCommentId: ObjectId (ref: 'Comment'), // for replies
  content: String (max: 1000, required),
  metadata: {
    likesCount: Number (default: 0),
    repliesCount: Number (default: 0)
  },
  mentions: [ObjectId], // User IDs mentioned
  createdAt: Date,
  updatedAt: Date
}
```

### Follows Collection
```javascript
{
  _id: ObjectId,
  followerId: ObjectId (ref: 'User', required),
  followingId: ObjectId (ref: 'User', required),
  createdAt: Date,
  // Compound index on (followerId, followingId) for uniqueness
}
```

### Likes Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required),
  targetType: String (enum: ['post', 'comment']),
  targetId: ObjectId (required),
  createdAt: Date,
  // Compound index on (userId, targetType, targetId) for uniqueness
}
```

### Notifications Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required),
  type: String (enum: ['like', 'comment', 'follow', 'mention', 'post_shared']),
  fromUserId: ObjectId (ref: 'User'),
  targetType: String (enum: ['post', 'comment', 'user']),
  targetId: ObjectId,
  content: {
    title: String (required),
    message: String (required),
    actionUrl: String
  },
  isRead: Boolean (default: false),
  createdAt: Date
}
```

---

## 🛠️ GraphQL Schema Design

### Type Definitions
```graphql
type User {
  id: ID!
  email: String!
  username: String!
  profile: UserProfile!
  preferences: UserPreferences!
  stats: UserStats!
  isVerified: Boolean!
  isActive: Boolean!
  lastActive: DateTime
  createdAt: DateTime!
  updatedAt: DateTime!
  
  # Relationships
  posts(limit: Int, offset: Int): [Post!]!
  followers(limit: Int, offset: Int): [User!]!
  following(limit: Int, offset: Int): [User!]!
  notifications(limit: Int, offset: Int, unreadOnly: Boolean): [Notification!]!
}

type UserProfile {
  firstName: String!
  lastName: String!
  bio: String
  avatar: String
  coverPhoto: String
  location: String
  website: String
  birthDate: DateTime
  isPublic: Boolean!
}

type UserStats {
  followersCount: Int!
  followingCount: Int!
  postsCount: Int!
  likesReceived: Int!
}

type Post {
  id: ID!
  author: User!
  content: PostContent!
  poll: Poll
  metadata: PostMetadata!
  visibility: PostVisibility!
  tags: [String!]!
  mentions: [User!]!
  location: Location
  isEdited: Boolean!
  editedAt: DateTime
  createdAt: DateTime!
  updatedAt: DateTime!
  
  # Relationships
  comments(limit: Int, offset: Int): [Comment!]!
  likes(limit: Int, offset: Int): [User!]!
  isLikedByCurrentUser: Boolean!
}

type PostContent {
  text: String
  type: PostContentType!
  media: [Media!]!
}

type Poll {
  question: String!
  options: [PollOption!]!
  expiresAt: DateTime
  allowMultipleVotes: Boolean!
  userVotes: [Int!]! # Current user's votes
}

type PollOption {
  text: String!
  votes: Int!
}

type Media {
  url: String!
  type: MediaType!
  altText: String
  width: Int
  height: Int
  duration: Int
}

type PostMetadata {
  likesCount: Int!
  commentsCount: Int!
  sharesCount: Int!
  viewsCount: Int!
}

type Comment {
  id: ID!
  post: Post!
  author: User!
  parentComment: Comment
  content: String!
  metadata: CommentMetadata!
  mentions: [User!]!
  createdAt: DateTime!
  updatedAt: DateTime!
  
  # Relationships
  replies(limit: Int, offset: Int): [Comment!]!
  likes(limit: Int, offset: Int): [User!]!
  isLikedByCurrentUser: Boolean!
}

type CommentMetadata {
  likesCount: Int!
  repliesCount: Int!
}

type Notification {
  id: ID!
  type: NotificationType!
  fromUser: User
  targetType: String
  targetId: ID
  content: NotificationContent!
  isRead: Boolean!
  createdAt: DateTime!
}

type NotificationContent {
  title: String!
  message: String!
  actionUrl: String
}

type Location {
  name: String!
  coordinates: Coordinates!
}

type Coordinates {
  latitude: Float!
  longitude: Float!
}

# Enums
enum PostContentType {
  TEXT
  IMAGE
  VIDEO
  POLL
}

enum MediaType {
  IMAGE
  VIDEO
}

enum PostVisibility {
  PUBLIC
  FOLLOWERS
  PRIVATE
}

enum NotificationType {
  LIKE
  COMMENT
  FOLLOW
  MENTION
  POST_SHARED
}

# Input Types
input CreatePostInput {
  content: PostContentInput!
  poll: PollInput
  visibility: PostVisibility = PUBLIC
  tags: [String!] = []
  location: LocationInput
}

input PostContentInput {
  text: String
  type: PostContentType!
  media: [MediaInput!] = []
}

input MediaInput {
  url: String!
  type: MediaType!
  altText: String
  width: Int
  height: Int
  duration: Int
}

input PollInput {
  question: String!
  options: [String!]!
  expiresAt: DateTime
  allowMultipleVotes: Boolean = false
}

input LocationInput {
  name: String!
  latitude: Float!
  longitude: Float!
}

input UpdateUserProfileInput {
  firstName: String
  lastName: String
  bio: String
  location: String
  website: String
  birthDate: DateTime
  isPublic: Boolean
}

# Queries
type Query {
  # User queries
  me: User
  user(id: ID, username: String): User
  users(search: String, limit: Int, offset: Int): [User!]!
  
  # Post queries
  posts(limit: Int, offset: Int, type: PostContentType, userId: ID): [Post!]!
  post(id: ID!): Post
  feed(limit: Int, offset: Int): [Post!]! # Personalized feed
  
  # Comment queries
  comments(postId: ID!, limit: Int, offset: Int): [Comment!]!
  
  # Search
  searchUsers(query: String!, limit: Int): [User!]!
  searchPosts(query: String!, limit: Int): [Post!]!
  searchHashtags(query: String!, limit: Int): [String!]!
  
  # Notifications
  notifications(limit: Int, offset: Int, unreadOnly: Boolean): [Notification!]!
  unreadNotificationsCount: Int!
}

# Mutations
type Mutation {
  # User mutations
  updateProfile(input: UpdateUserProfileInput!): User!
  followUser(userId: ID!): Boolean!
  unfollowUser(userId: ID!): Boolean!
  
  # Post mutations
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: CreatePostInput!): Post!
  deletePost(id: ID!): Boolean!
  likePost(postId: ID!): Boolean!
  unlikePost(postId: ID!): Boolean!
  
  # Comment mutations
  createComment(postId: ID!, content: String!, parentCommentId: ID): Comment!
  updateComment(id: ID!, content: String!): Comment!
  deleteComment(id: ID!): Boolean!
  likeComment(commentId: ID!): Boolean!
  unlikeComment(commentId: ID!): Boolean!
  
  # Poll mutations
  votePoll(postId: ID!, optionIndices: [Int!]!): Boolean!
  
  # Notification mutations
  markNotificationAsRead(id: ID!): Boolean!
  markAllNotificationsAsRead: Boolean!
}

# Subscriptions
type Subscription {
  # Real-time updates
  postAdded(userId: ID): Post!
  commentAdded(postId: ID!): Comment!
  likeAdded(postId: ID!): LikeEvent!
  followAdded(userId: ID!): FollowEvent!
  notificationAdded(userId: ID!): Notification!
}

type LikeEvent {
  postId: ID!
  userId: ID!
  user: User!
  action: String! # 'like' or 'unlike'
}

type FollowEvent {
  followerId: ID!
  followingId: ID!
  follower: User!
  action: String! # 'follow' or 'unfollow'
}
```

---

## 🛠️ REST API Endpoints (Additional)

### File Upload Endpoints

#### POST /upload/avatar
**Request**: Multipart form data with image file  
**Response (200):**
```json
{
  "success": true,
  "data": {
    "url": "https://cdn.example.com/avatars/user123.jpg",
    "altText": "User avatar",
    "width": 400,
    "height": 400
  }
}
```

#### POST /upload/media
**Request**: Multipart form data with image/video file  
**Response (200):**
```json
{
  "success": true,
  "data": {
    "url": "https://cdn.example.com/media/post456.jpg",
    "type": "image",
    "altText": "Post image",
    "width": 1200,
    "height": 800
  }
}
```

### Search Endpoints

#### GET /search/users
**Query Parameters:**
- `q` (string): Search query
- `limit` (number): Maximum results (default: 20)

**Response (200):**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "id": "uuid",
        "username": "johndoe",
        "profile": {
          "firstName": "John",
          "lastName": "Doe",
          "avatar": "https://cdn.example.com/avatar.jpg"
        },
        "stats": {
          "followersCount": 150,
          "followingCount": 200
        }
      }
    ],
    "total": 45
  }
}
```

#### GET /search/posts
**Query Parameters:**
- `q` (string): Search query
- `type` (string): Content type filter
- `limit` (number): Maximum results (default: 20)

---

## 🧪 Testing Requirements

### Unit Tests (80%+ coverage required)
- **User Service**: Profile management, follow/unfollow logic
- **Post Service**: CRUD operations, content validation
- **Comment Service**: Threading, mention parsing
- **Notification Service**: Real-time notification creation
- **Search Service**: User and content search algorithms

### Integration Tests
- **GraphQL Resolvers**: All queries and mutations
- **Real-time Features**: WebSocket connections and events
- **File Upload**: Image/video processing and storage
- **Search Functionality**: Elasticsearch integration

### API Tests
- **GraphQL Schema**: Schema validation and introspection
- **WebSocket Connections**: Real-time event broadcasting
- **File Upload**: Media processing and validation
- **Rate Limiting**: API abuse prevention

---

## 🚀 Deployment Requirements

### Environment Setup
```bash
# MongoDB
MONGODB_URI=mongodb://localhost:27017/socialmedia
MONGODB_DB_NAME=socialmedia

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=secure_redis_password

# File Storage (AWS S3 or similar)
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-east-1
AWS_S3_BUCKET=socialmedia-media

# JWT
JWT_SECRET=super_secure_jwt_secret_key
JWT_EXPIRES_IN=1h
JWT_REFRESH_SECRET=super_secure_refresh_secret
JWT_REFRESH_EXPIRES_IN=7d

# App
NODE_ENV=development
PORT=3000
API_PREFIX=api/v1
GRAPHQL_ENDPOINT=/graphql
SUBSCRIPTION_ENDPOINT=/subscriptions
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
- [GraphQL Documentation](https://graphql.org/learn/) - GraphQL fundamentals
- [Apollo Server](https://www.apollographql.com/docs/apollo-server/) - GraphQL server setup
- [MongoDB Documentation](https://docs.mongodb.com/) - NoSQL database
- [Socket.io Documentation](https://socket.io/docs/) - Real-time communication

### Udemy Courses (Focus Sections)
- **GraphQL Complete Course** - Sections 2-6 (Schema design, Resolvers, Subscriptions)
- **MongoDB Complete Course** - Sections 3-5 (Schema design, Aggregation)
- **Real-time Applications** - Sections 4-7 (WebSockets, Socket.io)

### Practice Projects
- Build a simple chat application with Socket.io
- Create a basic GraphQL API with subscriptions
- Implement a file upload system with image processing

---

## ✅ Success Criteria

### Week 3 Goals
- [ ] Set up GraphQL server with Apollo
- [ ] Implement user management with MongoDB
- [ ] Create post and comment system
- [ ] Build basic real-time features
- [ ] Set up file upload functionality

### Week 4 Goals
- [ ] Implement advanced search features
- [ ] Add notification system
- [ ] Create personalized feed algorithm
- [ ] Implement WebSocket subscriptions
- [ ] Add comprehensive testing
- [ ] Deploy with real-time capabilities

### Final Deliverables
- [ ] Fully functional GraphQL API
- [ ] Real-time features with WebSockets
- [ ] File upload and media processing
- [ ] Advanced search functionality
- [ ] 80%+ test coverage
- [ ] Deployed application with live demo
- [ ] GraphQL playground documentation

---

## 🔗 Related Projects
- **Previous Project**: [Project 1: E-commerce API Platform](../Backend_TypeScript/Project_1_E_Commerce_API/)
- **Next Project**: [Project 3: Data Processing Microservices](../Backend_TypeScript/Project_3_Data_Processing_Microservices/)
- **Training Plan**: [Backend TypeScript Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_TypeScript_Intensive/)
- **Main README**: [Repository Overview](../../README)
