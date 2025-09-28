# Project 2: Django Social Media API
*Full-stack Python social media platform with Django and Django REST Framework*

## Project Overview

Build a comprehensive social media platform using Django and Django REST Framework, showcasing full-stack Python development, admin interface customization, and real-time features. This project demonstrates your expertise in building complex web applications with user relationships, content management, and background task processing.

## Technology Stack

- **Framework**: Django 4.2+ with Django REST Framework
- **Database**: PostgreSQL 15+
- **ORM**: Django ORM
- **Authentication**: Django's built-in auth + JWT
- **Background Tasks**: Celery + Redis
- **Real-time**: Django Channels + WebSockets
- **Testing**: Django TestCase + pytest
- **Documentation**: Django REST Framework browsable API
- **Deployment**: Docker + AWS

## Core Features

### 1. User Management & Profiles
- User registration and authentication
- User profiles with avatars and bio
- User following/followers system
- User search and discovery
- Privacy settings and account management
- Email verification and password reset

### 2. Content Management
- Post creation and editing
- Image and video uploads
- Post likes, comments, and shares
- Post categories and tags
- Content moderation and reporting
- Post scheduling and drafts

### 3. Social Features
- News feed with algorithm
- User mentions and hashtags
- Direct messaging system
- Notifications system
- User activity feed
- Content sharing and reposting

### 4. Real-time Features
- Live notifications
- Real-time chat
- Live post updates
- Online user status
- Real-time likes and comments

### 5. Admin Features
- Django admin customization
- User management
- Content moderation
- Analytics dashboard
- System monitoring

## API Endpoints

### Authentication
```
POST /api/auth/register/
POST /api/auth/login/
POST /api/auth/logout/
POST /api/auth/refresh/
GET /api/auth/user/
PUT /api/auth/user/
```

### Users
```
GET /api/users/
GET /api/users/{user_id}/
PUT /api/users/{user_id}/
POST /api/users/{user_id}/follow/
DELETE /api/users/{user_id}/follow/
GET /api/users/{user_id}/followers/
GET /api/users/{user_id}/following/
```

### Posts
```
GET /api/posts/
POST /api/posts/
GET /api/posts/{post_id}/
PUT /api/posts/{post_id}/
DELETE /api/posts/{post_id}/
POST /api/posts/{post_id}/like/
DELETE /api/posts/{post_id}/like/
POST /api/posts/{post_id}/comment/
```

### Feed
```
GET /api/feed/
GET /api/feed/trending/
GET /api/feed/search/
```

### Messages
```
GET /api/messages/
POST /api/messages/
GET /api/messages/{message_id}/
PUT /api/messages/{message_id}/
DELETE /api/messages/{message_id}/
```

## Database Schema

### Users Table
```python
class User(AbstractUser):
    bio = models.TextField(blank=True, max_length=500)
    avatar = models.ImageField(upload_to='avatars/', blank=True)
    birth_date = models.DateField(null=True, blank=True)
    location = models.CharField(max_length=100, blank=True)
    website = models.URLField(blank=True)
    is_verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
```

### Posts Table
```python
class Post(models.Model):
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name='posts')
    content = models.TextField()
    image = models.ImageField(upload_to='posts/', blank=True)
    video = models.FileField(upload_to='posts/videos/', blank=True)
    is_published = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
```

### Follows Table
```python
class Follow(models.Model):
    follower = models.ForeignKey(User, on_delete=models.CASCADE, related_name='following')
    following = models.ForeignKey(User, on_delete=models.CASCADE, related_name='followers')
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ['follower', 'following']
```

## Implementation Guide

### Week 1: Foundation & Core Features

#### Day 1-2: Project Setup & Models
1. **Django Project Setup**
   ```bash
   django-admin startproject social_media
   cd social_media
   python manage.py startapp posts
   python manage.py startapp users
   python manage.py startapp notifications
   ```

2. **Project Structure**
   ```
   social_media/
   ├── social_media/
   │   ├── __init__.py
   │   ├── settings.py
   │   ├── urls.py
   │   └── wsgi.py
   ├── posts/
   ├── users/
   ├── notifications/
   ├── templates/
   ├── static/
   ├── media/
   ├── requirements.txt
   └── manage.py
   ```

3. **Database Models**
   - User model with custom fields
   - Post model with media support
   - Follow model for relationships
   - Like and Comment models

#### Day 3-5: Authentication & User Management
1. **Custom User Model**
   - Extend Django's User model
   - Add profile fields
   - Implement user registration

2. **Django REST Framework Setup**
   - Install and configure DRF
   - Create serializers
   - Implement authentication views

3. **User API Endpoints**
   - User registration and login
   - User profile management
   - User search and discovery

#### Day 6-7: Post Management
1. **Post Model & Serializers**
   - Create Post model with media support
   - Implement post serializers
   - Add post validation

2. **Post API Endpoints**
   - Post CRUD operations
   - Post listing with pagination
   - Post search and filtering

### Week 2: Advanced Features & Real-time

#### Day 8-10: Social Features
1. **Follow System**
   - Follow/unfollow functionality
   - Followers and following lists
   - Follow recommendations

2. **Like and Comment System**
   - Post likes and comments
   - Like and comment counts
   - Comment threading

#### Day 11-12: Real-time Features
1. **Django Channels Setup**
   - Install and configure Channels
   - WebSocket consumer setup
   - Real-time notifications

2. **Live Features**
   - Real-time post updates
   - Live notifications
   - Online user status

#### Day 13-14: Testing & Deployment
1. **Comprehensive Testing**
   - Unit tests for models
   - API endpoint tests
   - Integration tests
   - Test coverage analysis

2. **Admin Customization & Deployment**
   - Customize Django admin
   - Docker containerization
   - AWS deployment

## Testing Requirements

### Test Coverage
- **Unit Tests**: 90%+ coverage
- **API Tests**: All endpoints tested
- **Model Tests**: All model methods
- **Integration Tests**: Full user flows

### Test Structure
```python
# tests/test_models.py
class PostModelTest(TestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
    
    def test_post_creation(self):
        post = Post.objects.create(
            author=self.user,
            content='Test post content'
        )
        self.assertEqual(post.author, self.user)
        self.assertEqual(post.content, 'Test post content')

# tests/test_api.py
class PostAPITest(APITestCase):
    def setUp(self):
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.client.force_authenticate(user=self.user)
    
    def test_create_post(self):
        data = {'content': 'Test post content'}
        response = self.client.post('/api/posts/', data)
        self.assertEqual(response.status_code, 201)
```

## Performance Requirements

### Response Times
- **API Endpoints**: < 200ms average
- **Database Queries**: < 100ms average
- **Real-time Updates**: < 50ms average

### Scalability
- **Concurrent Users**: 1000+ simultaneous
- **Database Optimization**: Query optimization
- **Caching**: Redis for session management

## Security Requirements

### Authentication & Authorization
- Django's built-in authentication
- JWT tokens for API access
- Permission-based access control
- CSRF protection

### Data Protection
- Input validation and sanitization
- File upload security
- XSS protection
- SQL injection prevention

## Deployment Requirements

### Docker Configuration
```dockerfile
FROM python:3.11-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
EXPOSE 8000
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### AWS Services
- **EC2**: Application hosting
- **RDS**: PostgreSQL database
- **S3**: Media file storage
- **ElastiCache**: Redis for caching

## Success Criteria

### Technical Requirements
- [ ] Django application with DRF
- [ ] PostgreSQL database with proper relationships
- [ ] JWT authentication system
- [ ] Real-time features with WebSockets
- [ ] Comprehensive test suite (90%+ coverage)
- [ ] Custom Django admin interface
- [ ] Docker containerization
- [ ] AWS deployment

### Functional Requirements
- [ ] User registration and authentication
- [ ] Post creation and management
- [ ] Follow system
- [ ] Like and comment system
- [ ] Real-time notifications
- [ ] Admin dashboard
- [ ] API documentation
- [ ] Error handling and logging

### Quality Requirements
- [ ] Clean, documented code
- [ ] Django best practices
- [ ] Proper model relationships
- [ ] Efficient database queries
- [ ] Security best practices
- [ ] Performance optimization

## Bonus Features

### Advanced Features
- [ ] Post scheduling
- [ ] Content moderation
- [ ] Analytics dashboard
- [ ] Mobile API optimization
- [ ] Push notifications
- [ ] Content recommendations

### DevOps Features
- [ ] CI/CD pipeline
- [ ] Automated testing
- [ ] Database migrations
- [ ] Health checks
- [ ] Monitoring and alerting
- [ ] Backup and recovery

## Timeline

- **Week 1**: Core features and authentication
- **Week 2**: Advanced features, real-time, and deployment
- **Total**: 2 weeks (14 days)

## Resources

### Documentation
- [Django Documentation](https://docs.djangoproject.com/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Django Channels](https://channels.readthedocs.io/)

### Tutorials
- [Django Tutorial](https://docs.djangoproject.com/en/stable/intro/tutorial01/)
- [DRF Tutorial](https://www.django-rest-framework.org/tutorial/quickstart/)
- [Django Channels Tutorial](https://channels.readthedocs.io/en/stable/tutorial/)

---

**Ready to build a full-stack Python social media platform? Let's start coding!**
