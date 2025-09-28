# Project 3: Real-time Application
*Backend Intensive Plan - Weeks 5-6*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 5-6)  
**Tech Stack**: Node.js + Express + MongoDB + Redis + Socket.io + Bull Queue  
**Focus**: Real-time communication, WebSockets, and event-driven architecture  

### Learning Objectives
- Master Node.js and Express.js for backend development
- Implement real-time features with Socket.io and WebSockets
- Build event-driven applications with message queues
- Handle concurrent connections and scaling challenges
- Implement advanced MongoDB operations and indexing

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
    avatar: String, // URL to avatar image
    bio: String (max: 500),
    location: String,
    website: String,
    dateOfBirth: Date,
    isOnline: Boolean (default: false),
    lastSeen: Date
  },
  preferences: {
    emailNotifications: Boolean (default: true),
    pushNotifications: Boolean (default: true),
    privacy: {
      showOnlineStatus: Boolean (default: true),
      allowMessages: String (enum: ['everyone', 'contacts', 'none']),
      showLastSeen: Boolean (default: true)
    }
  },
  stats: {
    messagesSent: Number (default: 0),
    filesShared: Number (default: 0),
    loginCount: Number (default: 0)
  },
  isVerified: Boolean (default: false),
  isActive: Boolean (default: true),
  createdAt: Date,
  updatedAt: Date
}
```

### Conversations Collection
```javascript
{
  _id: ObjectId,
  type: String (enum: ['direct', 'group']),
  name: String, // For group chats
  description: String, // For group chats
  participants: [{
    userId: ObjectId (ref: 'User'),
    role: String (enum: ['admin', 'moderator', 'member']),
    joinedAt: Date,
    lastReadAt: Date,
    notifications: {
      enabled: Boolean (default: true),
      muteUntil: Date
    }
  }],
  settings: {
    isPublic: Boolean (default: false),
    allowInvites: Boolean (default: true),
    maxParticipants: Number (default: 100),
    messageRetention: Number (default: 30) // days
  },
  lastMessage: {
    content: String,
    senderId: ObjectId,
    sentAt: Date,
    type: String (enum: ['text', 'image', 'file', 'system'])
  },
  createdAt: Date,
  updatedAt: Date
}
```

### Messages Collection
```javascript
{
  _id: ObjectId,
  conversationId: ObjectId (ref: 'Conversation', required),
  senderId: ObjectId (ref: 'User', required),
  content: {
    text: String,
    type: String (enum: ['text', 'image', 'file', 'system', 'emoji']),
    metadata: {
      // For images
      imageUrl: String,
      imageSize: Number,
      imageDimensions: { width: Number, height: Number },
      // For files
      fileName: String,
      fileUrl: String,
      fileSize: Number,
      fileType: String,
      // For system messages
      systemType: String (enum: ['user_joined', 'user_left', 'name_changed'])
    }
  },
  replyTo: ObjectId (ref: 'Message'), // For replies
  reactions: [{
    userId: ObjectId,
    emoji: String,
    addedAt: Date
  }],
  readBy: [{
    userId: ObjectId,
    readAt: Date
  }],
  isEdited: Boolean (default: false),
  editedAt: Date,
  isDeleted: Boolean (default: false),
  deletedAt: Date,
  createdAt: Date,
  updatedAt: Date
}
```

### Notifications Collection
```javascript
{
  _id: ObjectId,
  userId: ObjectId (ref: 'User', required),
  type: String (enum: ['message', 'mention', 'invitation', 'system']),
  title: String (required),
  message: String (required),
  data: {
    conversationId: ObjectId,
    messageId: ObjectId,
    senderId: ObjectId,
    actionUrl: String
  },
  isRead: Boolean (default: false),
  readAt: Date,
  priority: String (enum: ['low', 'medium', 'high'], default: 'medium'),
  createdAt: Date,
  expiresAt: Date
}
```

### File Uploads Collection
```javascript
{
  _id: ObjectId,
  originalName: String (required),
  fileName: String (required),
  filePath: String (required),
  mimeType: String (required),
  fileSize: Number (required),
  uploadedBy: ObjectId (ref: 'User', required),
  conversationId: ObjectId (ref: 'Conversation'),
  messageId: ObjectId (ref: 'Message'),
  metadata: {
    width: Number, // For images
    height: Number, // For images
    duration: Number, // For videos/audio
    thumbnailUrl: String // For videos/images
  },
  isPublic: Boolean (default: false),
  downloadCount: Number (default: 0),
  createdAt: Date,
  expiresAt: Date // For temporary files
}
```

---

## 🛠️ Express.js Application Structure

### Main Application Setup
```javascript
// app.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const { createServer } = require('http');
const { Server } = require('socket.io');
const Redis = require('ioredis');

const authRoutes = require('./routes/auth');
const userRoutes = require('./routes/users');
const conversationRoutes = require('./routes/conversations');
const messageRoutes = require('./routes/messages');
const notificationRoutes = require('./routes/notifications');
const uploadRoutes = require('./routes/uploads');

const socketHandler = require('./socket/socketHandler');
const errorHandler = require('./middleware/errorHandler');
const { authenticateSocket } = require('./middleware/socketAuth');

const app = express();
const server = createServer(app);

// Socket.io setup
const io = new Server(server, {
  cors: {
    origin: process.env.CLIENT_URL || "http://localhost:3000",
    methods: ["GET", "POST"]
  }
});

// Redis setup
const redis = new Redis(process.env.REDIS_URL || 'redis://localhost:6379');

// Middleware
app.use(helmet());
app.use(cors({
  origin: process.env.CLIENT_URL || "http://localhost:3000",
  credentials: true
}));

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // limit each IP to 100 requests per windowMs
  message: 'Too many requests from this IP'
});
app.use('/api/', limiter);

app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Routes
app.use('/api/auth', authRoutes);
app.use('/api/users', userRoutes);
app.use('/api/conversations', conversationRoutes);
app.use('/api/messages', messageRoutes);
app.use('/api/notifications', notificationRoutes);
app.use('/api/uploads', uploadRoutes);

// Socket.io authentication middleware
io.use(authenticateSocket);

// Socket.io connection handling
io.on('connection', (socket) => {
  socketHandler(io, socket, redis);
});

// Error handling
app.use(errorHandler);

// Database connection
mongoose.connect(process.env.MONGODB_URI || 'mongodb://localhost:27017/realtime-app', {
  useNewUrlParser: true,
  useUnifiedTopology: true
}).then(() => {
  console.log('Connected to MongoDB');
}).catch((error) => {
  console.error('MongoDB connection error:', error);
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

module.exports = { app, io, redis };
```

### Socket.io Handler
```javascript
// socket/socketHandler.js
const Conversation = require('../models/Conversation');
const Message = require('../models/Message');
const User = require('../models/User');
const Notification = require('../models/Notification');

const socketHandler = async (io, socket, redis) => {
  const userId = socket.userId;
  console.log(`User ${userId} connected`);

  // Join user to their personal room
  socket.join(`user_${userId}`);

  // Update user online status
  await User.findByIdAndUpdate(userId, {
    'profile.isOnline': true,
    'profile.lastSeen': new Date()
  });

  // Join user to all their conversation rooms
  const conversations = await Conversation.find({
    'participants.userId': userId
  });
  
  conversations.forEach(conversation => {
    socket.join(`conversation_${conversation._id}`);
  });

  // Handle joining conversation
  socket.on('join_conversation', async (data) => {
    const { conversationId } = data;
    const conversation = await Conversation.findById(conversationId);
    
    if (conversation && conversation.participants.some(p => p.userId.toString() === userId)) {
      socket.join(`conversation_${conversationId}`);
      socket.emit('joined_conversation', { conversationId });
      
      // Notify others in the conversation
      socket.to(`conversation_${conversationId}`).emit('user_joined', {
        userId,
        conversationId
      });
    }
  });

  // Handle leaving conversation
  socket.on('leave_conversation', async (data) => {
    const { conversationId } = data;
    socket.leave(`conversation_${conversationId}`);
    
    // Notify others in the conversation
    socket.to(`conversation_${conversationId}`).emit('user_left', {
      userId,
      conversationId
    });
  });

  // Handle sending messages
  socket.on('send_message', async (data) => {
    try {
      const { conversationId, content, type = 'text', replyTo } = data;
      
      // Verify user is in conversation
      const conversation = await Conversation.findById(conversationId);
      if (!conversation || !conversation.participants.some(p => p.userId.toString() === userId)) {
        socket.emit('error', { message: 'Not authorized to send message to this conversation' });
        return;
      }

      // Create message
      const message = new Message({
        conversationId,
        senderId: userId,
        content,
        replyTo
      });

      await message.save();

      // Update conversation last message
      await Conversation.findByIdAndUpdate(conversationId, {
        lastMessage: {
          content: content.text || `[${type}]`,
          senderId: userId,
          sentAt: message.createdAt,
          type
        },
        updatedAt: new Date()
      });

      // Emit message to conversation
      const populatedMessage = await Message.findById(message._id)
        .populate('senderId', 'username profile.firstName profile.lastName profile.avatar')
        .populate('replyTo', 'content senderId')
        .populate('replyTo.senderId', 'username');

      io.to(`conversation_${conversationId}`).emit('new_message', populatedMessage);

      // Send notifications to offline users
      const offlineParticipants = conversation.participants.filter(p => {
        return p.userId.toString() !== userId && !io.sockets.adapter.rooms.has(`user_${p.userId}`);
      });

      for (const participant of offlineParticipants) {
        const notification = new Notification({
          userId: participant.userId,
          type: 'message',
          title: 'New message',
          message: `${populatedMessage.senderId.username}: ${content.text?.substring(0, 100) || `[${type}]`}`,
          data: {
            conversationId,
            messageId: message._id,
            senderId: userId
          }
        });

        await notification.save();
      }

      // Update user stats
      await User.findByIdAndUpdate(userId, {
        $inc: { 'stats.messagesSent': 1 }
      });

    } catch (error) {
      console.error('Error sending message:', error);
      socket.emit('error', { message: 'Failed to send message' });
    }
  });

  // Handle typing indicators
  socket.on('typing_start', (data) => {
    const { conversationId } = data;
    socket.to(`conversation_${conversationId}`).emit('user_typing', {
      userId,
      conversationId,
      isTyping: true
    });
  });

  socket.on('typing_stop', (data) => {
    const { conversationId } = data;
    socket.to(`conversation_${conversationId}`).emit('user_typing', {
      userId,
      conversationId,
      isTyping: false
    });
  });

  // Handle message reactions
  socket.on('add_reaction', async (data) => {
    try {
      const { messageId, emoji } = data;
      
      const message = await Message.findById(messageId);
      if (!message) return;

      // Check if user already reacted with this emoji
      const existingReaction = message.reactions.find(
        r => r.userId.toString() === userId && r.emoji === emoji
      );

      if (existingReaction) {
        // Remove reaction
        message.reactions = message.reactions.filter(
          r => !(r.userId.toString() === userId && r.emoji === emoji)
        );
      } else {
        // Add reaction
        message.reactions.push({
          userId,
          emoji,
          addedAt: new Date()
        });
      }

      await message.save();

      // Emit reaction update
      io.to(`conversation_${message.conversationId}`).emit('reaction_updated', {
        messageId,
        reactions: message.reactions
      });

    } catch (error) {
      console.error('Error adding reaction:', error);
    }
  });

  // Handle file uploads
  socket.on('file_upload_progress', (data) => {
    const { conversationId, progress } = data;
    socket.to(`conversation_${conversationId}`).emit('file_upload_progress', {
      userId,
      conversationId,
      progress
    });
  });

  // Handle disconnection
  socket.on('disconnect', async () => {
    console.log(`User ${userId} disconnected`);
    
    // Update user offline status
    await User.findByIdAndUpdate(userId, {
      'profile.isOnline': false,
      'profile.lastSeen': new Date()
    });

    // Notify all conversations that user went offline
    const conversations = await Conversation.find({
      'participants.userId': userId
    });

    conversations.forEach(conversation => {
      socket.to(`conversation_${conversation._id}`).emit('user_offline', {
        userId,
        conversationId: conversation._id
      });
    });
  });
};

module.exports = socketHandler;
```

### Message Routes
```javascript
// routes/messages.js
const express = require('express');
const router = express.Router();
const auth = require('../middleware/auth');
const Message = require('../models/Message');
const Conversation = require('../models/Conversation');

// Get messages for a conversation
router.get('/conversation/:conversationId', auth, async (req, res) => {
  try {
    const { conversationId } = req.params;
    const { page = 1, limit = 50 } = req.query;
    const skip = (page - 1) * limit;

    // Verify user is in conversation
    const conversation = await Conversation.findById(conversationId);
    if (!conversation || !conversation.participants.some(p => p.userId.toString() === req.user.id)) {
      return res.status(403).json({ error: 'Not authorized to view this conversation' });
    }

    const messages = await Message.find({
      conversationId,
      isDeleted: false
    })
    .populate('senderId', 'username profile.firstName profile.lastName profile.avatar')
    .populate('replyTo', 'content senderId')
    .populate('replyTo.senderId', 'username')
    .sort({ createdAt: -1 })
    .limit(parseInt(limit))
    .skip(skip);

    const total = await Message.countDocuments({
      conversationId,
      isDeleted: false
    });

    res.json({
      messages: messages.reverse(),
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        pages: Math.ceil(total / limit)
      }
    });

  } catch (error) {
    console.error('Error fetching messages:', error);
    res.status(500).json({ error: 'Server error' });
  }
});

// Edit a message
router.put('/:messageId', auth, async (req, res) => {
  try {
    const { messageId } = req.params;
    const { content } = req.body;

    const message = await Message.findOne({
      _id: messageId,
      senderId: req.user.id,
      isDeleted: false
    });

    if (!message) {
      return res.status(404).json({ error: 'Message not found' });
    }

    // Check if message is too old to edit (e.g., 15 minutes)
    const fifteenMinutesAgo = new Date(Date.now() - 15 * 60 * 1000);
    if (message.createdAt < fifteenMinutesAgo) {
      return res.status(400).json({ error: 'Message is too old to edit' });
    }

    message.content.text = content;
    message.isEdited = true;
    message.editedAt = new Date();

    await message.save();

    // Emit edit to conversation
    req.app.get('io').to(`conversation_${message.conversationId}`).emit('message_edited', {
      messageId: message._id,
      content: message.content,
      editedAt: message.editedAt
    });

    res.json({ message: 'Message updated successfully' });

  } catch (error) {
    console.error('Error editing message:', error);
    res.status(500).json({ error: 'Server error' });
  }
});

// Delete a message
router.delete('/:messageId', auth, async (req, res) => {
  try {
    const { messageId } = req.params;

    const message = await Message.findOne({
      _id: messageId,
      senderId: req.user.id,
      isDeleted: false
    });

    if (!message) {
      return res.status(404).json({ error: 'Message not found' });
    }

    // Soft delete
    message.isDeleted = true;
    message.deletedAt = new Date();
    await message.save();

    // Emit deletion to conversation
    req.app.get('io').to(`conversation_${message.conversationId}`).emit('message_deleted', {
      messageId: message._id
    });

    res.json({ message: 'Message deleted successfully' });

  } catch (error) {
    console.error('Error deleting message:', error);
    res.status(500).json({ error: 'Server error' });
  }
});

// Mark messages as read
router.post('/mark-read', auth, async (req, res) => {
  try {
    const { messageIds, conversationId } = req.body;

    if (messageIds && messageIds.length > 0) {
      await Message.updateMany(
        {
          _id: { $in: messageIds },
          conversationId,
          'readBy.userId': { $ne: req.user.id }
        },
        {
          $push: {
            readBy: {
              userId: req.user.id,
              readAt: new Date()
            }
          }
        }
      );
    }

    // Update conversation last read time
    await Conversation.updateOne(
      {
        _id: conversationId,
        'participants.userId': req.user.id
      },
      {
        $set: {
          'participants.$.lastReadAt': new Date()
        }
      }
    );

    res.json({ message: 'Messages marked as read' });

  } catch (error) {
    console.error('Error marking messages as read:', error);
    res.status(500).json({ error: 'Server error' });
  }
});

module.exports = router;
```

---

## 🧪 Testing Requirements

### Jest Configuration
```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  setupFilesAfterEnv: ['<rootDir>/tests/setup.js'],
  testMatch: ['**/__tests__/**/*.js', '**/?(*.)+(spec|test).js'],
  collectCoverageFrom: [
    'controllers/**/*.js',
    'models/**/*.js',
    'routes/**/*.js',
    'socket/**/*.js',
    '!**/node_modules/**'
  ],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html']
};
```

### Test Examples
```javascript
// tests/socket.test.js
const request = require('supertest');
const { createServer } = require('http');
const { Server } = require('socket.io-client');
const mongoose = require('mongoose');

const app = require('../app');
const User = require('../models/User');
const Conversation = require('../models/Conversation');

describe('Socket.io Integration', () => {
  let server;
  let clientSocket;
  let user;
  let conversation;

  beforeAll(async () => {
    // Connect to test database
    await mongoose.connect(process.env.MONGODB_TEST_URI);
    
    // Create test user
    user = new User({
      email: 'test@example.com',
      username: 'testuser',
      passwordHash: 'hashedpassword',
      profile: {
        firstName: 'Test',
        lastName: 'User'
      }
    });
    await user.save();

    // Create test conversation
    conversation = new Conversation({
      type: 'direct',
      participants: [
        { userId: user._id, role: 'member' }
      ]
    });
    await conversation.save();
  });

  beforeEach((done) => {
    server = createServer(app);
    server.listen(() => {
      const port = server.address().port;
      
      // Create authenticated socket connection
      clientSocket = io(`http://localhost:${port}`, {
        auth: {
          token: generateTestToken(user._id)
        }
      });

      clientSocket.on('connect', done);
    });
  });

  afterEach(() => {
    clientSocket.close();
    server.close();
  });

  afterAll(async () => {
    await mongoose.connection.close();
  });

  test('should connect with valid token', (done) => {
    clientSocket.on('connect', () => {
      expect(clientSocket.connected).toBe(true);
      done();
    });
  });

  test('should send message and receive it', (done) => {
    const messageContent = {
      text: 'Hello, world!',
      type: 'text'
    };

    clientSocket.emit('send_message', {
      conversationId: conversation._id,
      content: messageContent
    });

    clientSocket.on('new_message', (message) => {
      expect(message.content.text).toBe(messageContent.text);
      expect(message.senderId._id.toString()).toBe(user._id.toString());
      done();
    });
  });

  test('should handle typing indicators', (done) => {
    clientSocket.emit('typing_start', {
      conversationId: conversation._id
    });

    clientSocket.on('user_typing', (data) => {
      expect(data.userId.toString()).toBe(user._id.toString());
      expect(data.isTyping).toBe(true);
      done();
    });
  });
});

// tests/api/messages.test.js
describe('Messages API', () => {
  let authToken;
  let user;
  let conversation;

  beforeAll(async () => {
    // Create test user and get auth token
    user = await createTestUser();
    authToken = await getAuthToken(user);
    conversation = await createTestConversation(user._id);
  });

  test('GET /api/messages/conversation/:id should return messages', async () => {
    const response = await request(app)
      .get(`/api/messages/conversation/${conversation._id}`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(response.body.messages).toBeDefined();
    expect(response.body.pagination).toBeDefined();
  });

  test('PUT /api/messages/:id should edit message', async () => {
    const message = await createTestMessage(user._id, conversation._id);
    
    const response = await request(app)
      .put(`/api/messages/${message._id}`)
      .set('Authorization', `Bearer ${authToken}`)
      .send({ content: { text: 'Edited message' } })
      .expect(200);

    expect(response.body.message).toBe('Message updated successfully');
  });

  test('DELETE /api/messages/:id should delete message', async () => {
    const message = await createTestMessage(user._id, conversation._id);
    
    const response = await request(app)
      .delete(`/api/messages/${message._id}`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);

    expect(response.body.message).toBe('Message deleted successfully');
  });
});
```

---

## 🚀 Deployment Requirements

### Docker Configuration
```dockerfile
# Dockerfile
FROM node:18-alpine

# Install dependencies
RUN apk add --no-cache \
    python3 \
    make \
    g++ \
    cairo-dev \
    jpeg-dev \
    pango-dev \
    musl-dev \
    giflib-dev \
    pixman-dev \
    pangomm-dev \
    libjpeg-turbo-dev \
    freetype-dev

WORKDIR /app

# Copy package files
COPY package*.json ./
RUN npm ci --only=production

# Copy application code
COPY . .

# Create uploads directory
RUN mkdir -p uploads/temp uploads/permanent

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Change ownership of uploads directory
RUN chown -R nodejs:nodejs uploads

USER nodejs

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["node", "server.js"]
```

### Environment Configuration
```bash
# .env
# Database
MONGODB_URI=mongodb://localhost:27017/realtime-app

# Redis
REDIS_URL=redis://localhost:6379

# JWT
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Application
NODE_ENV=development
PORT=3000
CLIENT_URL=http://localhost:3000

# File Upload
MAX_FILE_SIZE=10MB
UPLOAD_DIR=./uploads
ALLOWED_FILE_TYPES=image/jpeg,image/png,image/gif,application/pdf,text/plain

# Email (for notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

---

## 📚 Learning Resources

### Essential Reading
- [Express.js Documentation](https://expressjs.com/) - Web framework
- [Socket.io Documentation](https://socket.io/docs/) - Real-time communication
- [MongoDB Documentation](https://docs.mongodb.com/) - NoSQL database
- [Node.js Documentation](https://nodejs.org/docs/) - Runtime environment

### Udemy Courses (Focus Sections)
- **Node.js Complete Course** - Sections 3-8 (Express, Real-time, Testing)
- **Real-time Applications** - Sections 2-6 (Socket.io, WebSockets, Scaling)

### Practice Projects
- Build a simple chat application with Socket.io
- Create a real-time notification system
- Implement a basic multiplayer game backend

---

## ✅ Success Criteria

### Week 5 Goals
- [ ] Set up Node.js and Express application
- [ ] Implement MongoDB models and schemas
- [ ] Create Socket.io real-time communication
- [ ] Build basic message and conversation APIs
- [ ] Implement user authentication and authorization

### Week 6 Goals
- [ ] Add file upload and sharing functionality
- [ ] Implement advanced features (typing indicators, reactions)
- [ ] Create notification system
- [ ] Add comprehensive testing
- [ ] Deploy with Docker
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional real-time chat application
- [ ] Comprehensive API documentation
- [ ] Real-time features with Socket.io
- [ ] File upload and sharing capabilities
- [ ] 80%+ test coverage with Jest
- [ ] Docker deployment configuration
- [ ] Performance monitoring and optimization

---

## 🔗 Related Projects
- **Previous Project**: [Project 2: Data Processing Platform](../Backend_Intensive/Project_2_Data_Processing_Platform.html)
- **Next Project**: [Project 4: Production System](../Backend_Intensive/Project_4_Production_System.html)
- **Training Plan**: [Backend Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html)
- **Main README**: [Repository Overview](../../README.html)
