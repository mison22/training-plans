# Project 2: Social Media Dashboard
*Full Stack Intensive Plan - Weeks 3-4*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 3-4)  
**Tech Stack**: Next.js + TypeScript + FastAPI + PostgreSQL + Redis  
**Focus**: Modern frontend with SSR, real-time features, and advanced backend APIs  

### Learning Objectives
- Master Next.js with TypeScript for modern web applications
- Build FastAPI backend with advanced features
- Implement Server-Side Rendering (SSR) and Static Site Generation (SSG)
- Handle real-time data updates and WebSocket connections
- Create responsive dashboards with data visualization

---

## 📊 Database Schema Design

### Users Table (PostgreSQL)
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  username VARCHAR(50) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  avatar_url VARCHAR(500),
  bio TEXT,
  location VARCHAR(100),
  website VARCHAR(200),
  is_verified BOOLEAN DEFAULT false,
  is_private BOOLEAN DEFAULT false,
  followers_count INTEGER DEFAULT 0,
  following_count INTEGER DEFAULT 0,
  posts_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Posts Table
```sql
CREATE TABLE posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  content TEXT NOT NULL,
  media_urls TEXT[],
  is_repost BOOLEAN DEFAULT false,
  original_post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
  likes_count INTEGER DEFAULT 0,
  comments_count INTEGER DEFAULT 0,
  reposts_count INTEGER DEFAULT 0,
  views_count INTEGER DEFAULT 0,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Follows Table
```sql
CREATE TABLE follows (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  follower_id UUID REFERENCES users(id) ON DELETE CASCADE,
  following_id UUID REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(follower_id, following_id)
);
```

---

## 🛠️ Frontend Components (Next.js + TypeScript)

### Dashboard Layout
```typescript
// components/layout/DashboardLayout.tsx
import React, { useState, useEffect } from 'react';
import { useRouter } from 'next/router';
import { User } from '../types/User';
import Sidebar from './Sidebar';
import Header from './Header';
import CreatePost from '../posts/CreatePost';
import Feed from '../posts/Feed';

interface DashboardLayoutProps {
  children: React.ReactNode;
}

const DashboardLayout: React.FC<DashboardLayoutProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const router = useRouter();

  useEffect(() => {
    fetchUser();
  }, []);

  const fetchUser = async () => {
    try {
      const response = await fetch('/api/v1/users/me', {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        const userData = await response.json();
        setUser(userData);
      } else {
        router.push('/login');
      }
    } catch (error) {
      console.error('Error fetching user:', error);
      router.push('/login');
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return <div className="loading">Loading...</div>;
  }

  if (!user) {
    return null;
  }

  return (
    <div className="dashboard-layout">
      <Sidebar user={user} />
      <div className="main-content">
        <Header user={user} />
        <div className="content-area">
          {children}
        </div>
      </div>
    </div>
  );
};

export default DashboardLayout;
```

### Feed Component
```typescript
// components/posts/Feed.tsx
import React, { useState, useEffect, useRef } from 'react';
import { Post } from '../types/Post';
import PostCard from './PostCard';
import InfiniteScroll from 'react-infinite-scroll-component';

const Feed: React.FC = () => {
  const [posts, setPosts] = useState<Post[]>([]);
  const [loading, setLoading] = useState(true);
  const [hasMore, setHasMore] = useState(true);
  const [page, setPage] = useState(1);
  const wsRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    fetchPosts();
    setupWebSocket();
    
    return () => {
      if (wsRef.current) {
        wsRef.current.close();
      }
    };
  }, []);

  const fetchPosts = async (pageNum = 1) => {
    try {
      const response = await fetch(`/api/v1/posts/feed?page=${pageNum}&limit=10`, {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        const data = await response.json();
        
        if (pageNum === 1) {
          setPosts(data.posts);
        } else {
          setPosts(prev => [...prev, ...data.posts]);
        }
        
        setHasMore(data.has_more);
      }
    } catch (error) {
      console.error('Error fetching posts:', error);
    } finally {
      setLoading(false);
    }
  };

  const setupWebSocket = () => {
    const token = localStorage.getItem('token');
    wsRef.current = new WebSocket(`ws://localhost:8000/ws/feed?token=${token}`);

    wsRef.current.onmessage = (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'new_post') {
        setPosts(prev => [data.post, ...prev]);
      } else if (data.type === 'post_updated') {
        setPosts(prev => prev.map(post => 
          post.id === data.post.id ? data.post : post
        ));
      }
    };

    wsRef.current.onclose = () => {
      // Reconnect after 5 seconds
      setTimeout(setupWebSocket, 5000);
    };
  };

  const loadMore = () => {
    const nextPage = page + 1;
    setPage(nextPage);
    fetchPosts(nextPage);
  };

  const handleLike = async (postId: string) => {
    try {
      const response = await fetch(`/api/v1/posts/${postId}/like`, {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        // Optimistic update
        setPosts(prev => prev.map(post => 
          post.id === postId 
            ? { 
                ...post, 
                is_liked: !post.is_liked,
                likes_count: post.is_liked ? post.likes_count - 1 : post.likes_count + 1
              }
            : post
        ));
      }
    } catch (error) {
      console.error('Error liking post:', error);
    }
  };

  if (loading && posts.length === 0) {
    return <div className="loading">Loading posts...</div>;
  }

  return (
    <div className="feed">
      <InfiniteScroll
        dataLength={posts.length}
        next={loadMore}
        hasMore={hasMore}
        loader={<div className="loading">Loading more posts...</div>}
      >
        {posts.map((post) => (
          <PostCard 
            key={post.id} 
            post={post} 
            onLike={handleLike}
          />
        ))}
      </InfiniteScroll>
    </div>
  );
};

export default Feed;
```

### Post Card Component
```typescript
// components/posts/PostCard.tsx
import React, { useState } from 'react';
import { Post } from '../types/Post';
import { formatDistanceToNow } from 'date-fns';
import Image from 'next/image';

interface PostCardProps {
  post: Post;
  onLike: (postId: string) => void;
}

const PostCard: React.FC<PostCardProps> = ({ post, onLike }) => {
  const [showComments, setShowComments] = useState(false);

  const handleLike = () => {
    onLike(post.id);
  };

  return (
    <div className="post-card">
      <div className="post-header">
        <Image
          src={post.user.avatar_url || '/default-avatar.png'}
          alt={post.user.username}
          width={40}
          height={40}
          className="avatar"
        />
        <div className="user-info">
          <h4>{post.user.first_name} {post.user.last_name}</h4>
          <span className="username">@{post.user.username}</span>
          <span className="timestamp">
            {formatDistanceToNow(new Date(post.created_at))} ago
          </span>
        </div>
      </div>

      <div className="post-content">
        <p>{post.content}</p>
        
        {post.media_urls && post.media_urls.length > 0 && (
          <div className="post-media">
            {post.media_urls.map((url, index) => (
              <Image
                key={index}
                src={url}
                alt={`Post media ${index + 1}`}
                width={400}
                height={300}
                className="post-image"
              />
            ))}
          </div>
        )}
      </div>

      <div className="post-actions">
        <button 
          className={`action-btn ${post.is_liked ? 'liked' : ''}`}
          onClick={handleLike}
        >
          ❤️ {post.likes_count}
        </button>
        
        <button 
          className="action-btn"
          onClick={() => setShowComments(!showComments)}
        >
          💬 {post.comments_count}
        </button>
        
        <button className="action-btn">
          🔄 {post.reposts_count}
        </button>
      </div>

      {showComments && (
        <div className="comments-section">
          {/* Comments component would go here */}
          <p>Comments loading...</p>
        </div>
      )}
    </div>
  );
};

export default PostCard;
```

---

## 🛠️ Backend API (FastAPI)

### Posts Router
```python
# routers/posts.py
from fastapi import APIRouter, Depends, HTTPException, WebSocket, WebSocketDisconnect
from sqlalchemy.orm import Session
from typing import List, Optional
import json

from database import get_db
from models import Post, User, Like
from schemas import PostCreate, PostResponse, PostUpdate
from auth import get_current_user
from websocket_manager import manager

router = APIRouter(prefix="/posts", tags=["posts"])

@router.get("/feed", response_model=List[PostResponse])
async def get_feed(
    page: int = 1,
    limit: int = 10,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Get user's feed with pagination"""
    offset = (page - 1) * limit
    
    # Get posts from users the current user follows
    posts = db.query(Post).join(
        User, Post.user_id == User.id
    ).filter(
        User.id.in_(
            db.query(User.id).join(
                Follow, Follow.following_id == User.id
            ).filter(Follow.follower_id == current_user.id)
        )
    ).order_by(Post.created_at.desc()).offset(offset).limit(limit).all()
    
    # Add user data and like status
    post_responses = []
    for post in posts:
        post_data = PostResponse.from_orm(post)
        post_data.user = post.user
        post_data.is_liked = db.query(Like).filter(
            Like.post_id == post.id,
            Like.user_id == current_user.id
        ).first() is not None
        post_responses.append(post_data)
    
    return post_responses

@router.post("/", response_model=PostResponse)
async def create_post(
    post_data: PostCreate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Create a new post"""
    post = Post(
        user_id=current_user.id,
        content=post_data.content,
        media_urls=post_data.media_urls
    )
    
    db.add(post)
    db.commit()
    db.refresh(post)
    
    # Update user's post count
    current_user.posts_count += 1
    db.commit()
    
    # Send real-time update
    await manager.broadcast_to_followers(
        current_user.id,
        {
            "type": "new_post",
            "post": PostResponse.from_orm(post).dict()
        }
    )
    
    return PostResponse.from_orm(post)

@router.post("/{post_id}/like")
async def like_post(
    post_id: str,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Like or unlike a post"""
    post = db.query(Post).filter(Post.id == post_id).first()
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")
    
    existing_like = db.query(Like).filter(
        Like.post_id == post_id,
        Like.user_id == current_user.id
    ).first()
    
    if existing_like:
        # Unlike
        db.delete(existing_like)
        post.likes_count -= 1
    else:
        # Like
        like = Like(post_id=post_id, user_id=current_user.id)
        db.add(like)
        post.likes_count += 1
    
    db.commit()
    
    # Send real-time update
    await manager.broadcast_to_all({
        "type": "post_updated",
        "post": {
            "id": post.id,
            "likes_count": post.likes_count,
            "is_liked": existing_like is None
        }
    })
    
    return {"liked": existing_like is None, "likes_count": post.likes_count}

@router.websocket("/ws/feed")
async def websocket_feed(websocket: WebSocket, token: str = None):
    """WebSocket endpoint for real-time feed updates"""
    await manager.connect(websocket, token)
    
    try:
        while True:
            data = await websocket.receive_text()
            # Handle client messages if needed
    except WebSocketDisconnect:
        manager.disconnect(websocket)
```

### WebSocket Manager
```python
# websocket_manager.py
from fastapi import WebSocket
from typing import List, Dict
import json
import asyncio

class ConnectionManager:
    def __init__(self):
        self.active_connections: Dict[str, WebSocket] = {}
        self.user_connections: Dict[str, List[str]] = {}

    async def connect(self, websocket: WebSocket, token: str = None):
        await websocket.accept()
        
        # In a real app, you'd validate the token and get user_id
        user_id = "user_123"  # This would come from token validation
        
        connection_id = id(websocket)
        self.active_connections[str(connection_id)] = websocket
        
        if user_id not in self.user_connections:
            self.user_connections[user_id] = []
        self.user_connections[user_id].append(str(connection_id))

    def disconnect(self, websocket: WebSocket):
        connection_id = str(id(websocket))
        if connection_id in self.active_connections:
            del self.active_connections[connection_id]
            
        # Remove from user connections
        for user_id, connections in self.user_connections.items():
            if connection_id in connections:
                connections.remove(connection_id)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast_to_followers(self, user_id: str, message: dict):
        """Send message to all followers of a user"""
        # This would query the database for followers
        # For now, broadcast to all connections
        await self.broadcast_to_all(message)

    async def broadcast_to_all(self, message: dict):
        """Broadcast message to all connected clients"""
        disconnected = []
        for connection_id, websocket in self.active_connections.items():
            try:
                await websocket.send_text(json.dumps(message))
            except:
                disconnected.append(connection_id)
        
        # Clean up disconnected connections
        for connection_id in disconnected:
            del self.active_connections[connection_id]

manager = ConnectionManager()
```

---

## 🧪 Testing Requirements

### Frontend Testing
```typescript
// __tests__/Feed.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { rest } from 'msw';
import { setupServer } from 'msw/node';
import Feed from '../components/posts/Feed';

// Mock WebSocket
global.WebSocket = jest.fn(() => ({
  onmessage: null,
  onclose: null,
  close: jest.fn(),
})) as any;

// Mock API server
const server = setupServer(
  rest.get('/api/v1/posts/feed', (req, res, ctx) => {
    return res(ctx.json({
      posts: [
        {
          id: '1',
          content: 'Test post',
          user: {
            id: '1',
            username: 'testuser',
            first_name: 'Test',
            last_name: 'User',
            avatar_url: 'test.jpg'
          },
          likes_count: 5,
          comments_count: 2,
          created_at: new Date().toISOString(),
          is_liked: false
        }
      ],
      has_more: false
    }));
  }),
  
  rest.post('/api/v1/posts/1/like', (req, res, ctx) => {
    return res(ctx.json({
      liked: true,
      likes_count: 6
    }));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('Feed Component', () => {
  beforeEach(() => {
    // Mock localStorage
    Object.defineProperty(window, 'localStorage', {
      value: {
        getItem: jest.fn(() => 'mock-token')
      }
    });
  });

  test('renders posts', async () => {
    render(<Feed />);

    await waitFor(() => {
      expect(screen.getByText('Test post')).toBeInTheDocument();
      expect(screen.getByText('Test User')).toBeInTheDocument();
    });
  });

  test('handles like action', async () => {
    render(<Feed />);

    await waitFor(() => {
      expect(screen.getByText('Test post')).toBeInTheDocument();
    });

    const likeButton = screen.getByText('❤️ 5');
    fireEvent.click(likeButton);

    await waitFor(() => {
      expect(screen.getByText('❤️ 6')).toBeInTheDocument();
    });
  });
});
```

### Backend Testing
```python
# tests/test_posts.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy.orm import Session

from main import app
from models import Post, User
from database import get_db

client = TestClient(app)

@pytest.fixture
def test_user(db: Session):
    user = User(
        email="test@example.com",
        username="testuser",
        first_name="Test",
        last_name="User"
    )
    db.add(user)
    db.commit()
    db.refresh(user)
    return user

@pytest.fixture
def auth_headers(test_user):
    # In a real test, you'd create a proper JWT token
    return {"Authorization": "Bearer mock-token"}

def test_create_post(test_user, auth_headers, db: Session):
    response = client.post(
        "/posts/",
        json={
            "content": "Test post content",
            "media_urls": []
        },
        headers=auth_headers
    )
    
    assert response.status_code == 200
    data = response.json()
    assert data["content"] == "Test post content"
    assert data["user_id"] == test_user.id

def test_get_feed(test_user, auth_headers, db: Session):
    # Create a test post
    post = Post(
        user_id=test_user.id,
        content="Test feed post"
    )
    db.add(post)
    db.commit()
    
    response = client.get(
        "/posts/feed?page=1&limit=10",
        headers=auth_headers
    )
    
    assert response.status_code == 200
    data = response.json()
    assert len(data) == 1
    assert data[0]["content"] == "Test feed post"

def test_like_post(test_user, auth_headers, db: Session):
    # Create a test post
    post = Post(
        user_id=test_user.id,
        content="Test post to like"
    )
    db.add(post)
    db.commit()
    
    response = client.post(
        f"/posts/{post.id}/like",
        headers=auth_headers
    )
    
    assert response.status_code == 200
    data = response.json()
    assert data["liked"] is True
    assert data["likes_count"] == 1
```

---

## 🚀 Deployment Requirements

### Next.js Deployment (Vercel)
```json
// package.json
{
  "name": "social-media-dashboard",
  "version": "1.0.0",
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "test": "jest"
  },
  "dependencies": {
    "next": "^13.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^4.9.0",
    "date-fns": "^2.29.0",
    "react-infinite-scroll-component": "^6.1.0"
  }
}
```

### FastAPI Deployment (Railway/Heroku)
```python
# main.py
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import uvicorn

from routers import posts, users, auth
from database import engine
from models import Base

# Create database tables
Base.metadata.create_all(bind=engine)

app = FastAPI(title="Social Media API", version="1.0.0")

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://yourdomain.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(posts.router)
app.include_router(users.router)
app.include_router(auth.router)

@app.get("/health")
async def health_check():
    return {"status": "healthy"}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

---

## 📚 Learning Resources

### Essential Reading
- [Next.js Documentation](https://nextjs.org/docs) - React framework with SSR
- [FastAPI Documentation](https://fastapi.tiangolo.com/) - Modern Python API framework
- [WebSocket Documentation](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) - Real-time communication
- [PostgreSQL Documentation](https://www.postgresql.org/docs/) - Database features

### Udemy Courses (Focus Sections)
- **Next.js Complete Course** - Sections 2-8 (SSR, API routes, Deployment)
- **FastAPI Complete Course** - Sections 3-7 (WebSockets, Background tasks)
- **React Complete Guide** - Sections 6-10 (Advanced patterns, Performance)

### Practice Projects
- Build a real-time chat application
- Create a social media clone with Next.js
- Implement a dashboard with data visualization

---

## ✅ Success Criteria

### Week 3 Goals
- [ ] Set up Next.js frontend with TypeScript
- [ ] Set up FastAPI backend with PostgreSQL
- [ ] Implement user authentication and profiles
- [ ] Create post creation and display functionality
- [ ] Build basic feed with infinite scroll

### Week 4 Goals
- [ ] Implement real-time features with WebSockets
- [ ] Add like, comment, and share functionality
- [ ] Create responsive dashboard design
- [ ] Add comprehensive testing
- [ ] Deploy frontend and backend
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional social media dashboard
- [ ] Real-time updates with WebSocket
- [ ] Responsive Next.js frontend
- [ ] FastAPI backend with advanced features
- [ ] User profiles and social interactions
- [ ] 80%+ test coverage on both frontend and backend
- [ ] Deployed applications with live demos

---

## 🔗 Related Projects
- **Previous Project**: [Project 1: E-commerce Platform](../Full_Stack/Project_1_E_Commerce_Platform.html)
- **Next Project**: [Project 3: Project Management Tool](../Full_Stack/Project_3_Project_Management_Tool.html)
- **Training Plan**: [Full Stack Intensive Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html)
- **Main README**: [Repository Overview](../../README.html)
