# Project 2: Blog Management System
*3-week project for TypeScript Backend Evening (Weeks 4-6)*

## Project Overview
Create a comprehensive blog management system with user authentication, content management, and commenting features. This project focuses on database integration, advanced API design, and user management while building a practical blogging platform.

## Learning Objectives
- Master database relationships and complex queries
- Implement advanced authentication and authorization
- Design RESTful APIs with proper error handling
- Add file upload and media management
- Create comprehensive API documentation

## Technology Stack
- **Language**: TypeScript 5.0+
- **Framework**: NestJS
- **Database**: PostgreSQL
- **ORM**: TypeORM
- **Authentication**: JWT with refresh tokens
- **File Upload**: Multer
- **Documentation**: Swagger/OpenAPI
- **Validation**: Class-validator

## Project Timeline (3 Weeks)

### **Week 4: User Management & Authentication**
**Goal**: Implement comprehensive user management and authentication

#### **Day 1-2: Advanced Authentication**
- JWT access and refresh tokens
- Role-based access control (Admin, Author, Reader)
- Password reset functionality
- Email verification
- Account activation/deactivation

#### **Day 3-4: User Profile Management**
- User profile CRUD operations
- Avatar upload and management
- User preferences and settings
- Account security features
- User activity tracking

#### **Day 5: Authorization Middleware**
- Role-based route protection
- Permission-based access control
- Resource ownership validation
- Admin panel access control
- Audit logging

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Authentication theory and implementation
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 5: Content Management System**
**Goal**: Build comprehensive content management features

#### **Day 1-2: Blog Post Management**
- Create, read, update, delete blog posts
- Draft and published post states
- Post scheduling and publishing
- Post categorization and tagging
- Featured posts and sticky posts

#### **Day 3-4: Media Management**
- Image upload and processing
- File type validation and security
- Image resizing and optimization
- Media library management
- CDN integration preparation

#### **Day 5: Content Organization**
- Categories and tags management
- Post search and filtering
- Content relationships
- SEO metadata management
- Content versioning

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Content management features
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 6: Comments & Social Features**
**Goal**: Add commenting system and social features

#### **Day 1-2: Commenting System**
- Nested comment structure
- Comment moderation and approval
- Comment likes and reactions
- Spam detection and prevention
- Comment notifications

#### **Day 3-4: Social Features**
- Post likes and bookmarks
- User following system
- Activity feeds
- Notifications system
- Social sharing

#### **Day 5: API Documentation & Testing**
- Complete Swagger documentation
- API testing with Postman
- Integration testing
- Performance testing
- Security testing

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Social features and testing
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

## Detailed Requirements

### **Database Schema**

#### **Users Table (Extended)**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    bio TEXT,
    avatar_url VARCHAR(500),
    role VARCHAR(20) DEFAULT 'reader', -- 'admin', 'author', 'reader'
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Categories Table**
```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(100) NOT NULL,
    slug VARCHAR(100) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7), -- Hex color code
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Posts Table**
```sql
CREATE TABLE posts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    author_id UUID REFERENCES users(id) ON DELETE CASCADE,
    category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
    title VARCHAR(255) NOT NULL,
    slug VARCHAR(255) UNIQUE NOT NULL,
    excerpt TEXT,
    content TEXT NOT NULL,
    featured_image_url VARCHAR(500),
    status VARCHAR(20) DEFAULT 'draft', -- 'draft', 'published', 'archived'
    featured BOOLEAN DEFAULT FALSE,
    sticky BOOLEAN DEFAULT FALSE,
    published_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Tags Table**
```sql
CREATE TABLE tags (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    color VARCHAR(7),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Post Tags Table (Many-to-Many)**
```sql
CREATE TABLE post_tags (
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
    tag_id UUID REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);
```

#### **Comments Table**
```sql
CREATE TABLE comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
    author_id UUID REFERENCES users(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES comments(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    is_approved BOOLEAN DEFAULT FALSE,
    is_spam BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Post Likes Table**
```sql
CREATE TABLE post_likes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    post_id UUID REFERENCES posts(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(post_id, user_id)
);
```

### **API Endpoints**

#### **Authentication**
```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout
POST   /auth/forgot-password
POST   /auth/reset-password
POST   /auth/verify-email
GET    /auth/profile
PUT    /auth/profile
```

#### **Users**
```
GET    /users
GET    /users/:id
PUT    /users/:id
DELETE /users/:id
GET    /users/:id/posts
GET    /users/:id/followers
GET    /users/:id/following
POST   /users/:id/follow
DELETE /users/:id/follow
```

#### **Posts**
```
GET    /posts
GET    /posts/:id
POST   /posts
PUT    /posts/:id
DELETE /posts/:id
GET    /posts/featured
GET    /posts/category/:categoryId
GET    /posts/tag/:tagId
GET    /posts/search
POST   /posts/:id/like
DELETE /posts/:id/like
```

#### **Categories**
```
GET    /categories
GET    /categories/:id
POST   /categories
PUT    /categories/:id
DELETE /categories/:id
```

#### **Comments**
```
GET    /posts/:postId/comments
POST   /posts/:postId/comments
PUT    /comments/:id
DELETE /comments/:id
POST   /comments/:id/like
```

### **TypeScript Implementation**

#### **Post Entity with Relationships**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, ManyToOne, ManyToMany, OneToMany, JoinColumn, JoinTable } from 'typeorm';
import { User } from './user.entity';
import { Category } from './category.entity';
import { Tag } from './tag.entity';
import { Comment } from './comment.entity';
import { PostLike } from './post-like.entity';

@Entity('posts')
export class Post {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  authorId: string;

  @Column({ nullable: true })
  categoryId: string;

  @Column()
  title: string;

  @Column({ unique: true })
  slug: string;

  @Column({ nullable: true })
  excerpt: string;

  @Column('text')
  content: string;

  @Column({ nullable: true })
  featuredImageUrl: string;

  @Column({ default: 'draft' })
  status: 'draft' | 'published' | 'archived';

  @Column({ default: false })
  featured: boolean;

  @Column({ default: false })
  sticky: boolean;

  @Column({ nullable: true })
  publishedAt: Date;

  @ManyToOne(() => User, user => user.posts, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'authorId' })
  author: User;

  @ManyToOne(() => Category, category => category.posts, { onDelete: 'SET NULL' })
  @JoinColumn({ name: 'categoryId' })
  category: Category;

  @ManyToMany(() => Tag, tag => tag.posts)
  @JoinTable({
    name: 'post_tags',
    joinColumn: { name: 'postId', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'tagId', referencedColumnName: 'id' },
  })
  tags: Tag[];

  @OneToMany(() => Comment, comment => comment.post)
  comments: Comment[];

  @OneToMany(() => PostLike, like => like.post)
  likes: PostLike[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

#### **Posts Service with Advanced Features**
```typescript
import { Injectable, NotFoundException, ForbiddenException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, SelectQueryBuilder } from 'typeorm';
import { Post } from './entities/post.entity';
import { CreatePostDto } from './dto/create-post.dto';
import { UpdatePostDto } from './dto/update-post.dto';
import { PostQueryDto } from './dto/post-query.dto';

@Injectable()
export class PostsService {
  constructor(
    @InjectRepository(Post)
    private postsRepository: Repository<Post>,
  ) {}

  async create(createPostDto: CreatePostDto, authorId: string): Promise<Post> {
    const post = this.postsRepository.create({
      ...createPostDto,
      authorId,
      slug: this.generateSlug(createPostDto.title),
    });
    return this.postsRepository.save(post);
  }

  async findAll(query: PostQueryDto): Promise<{ posts: Post[]; total: number }> {
    const queryBuilder = this.createQueryBuilder(query);
    
    const [posts, total] = await queryBuilder
      .skip((query.page - 1) * query.limit)
      .take(query.limit)
      .getManyAndCount();

    return { posts, total };
  }

  async findOne(id: string): Promise<Post> {
    const post = await this.postsRepository.findOne({
      where: { id },
      relations: ['author', 'category', 'tags', 'comments', 'likes'],
    });
    if (!post) {
      throw new NotFoundException('Post not found');
    }
    return post;
  }

  async update(id: string, updatePostDto: UpdatePostDto, userId: string): Promise<Post> {
    const post = await this.findOne(id);
    
    // Check if user is the author or admin
    if (post.authorId !== userId) {
      throw new ForbiddenException('You can only update your own posts');
    }

    Object.assign(post, updatePostDto);
    return this.postsRepository.save(post);
  }

  async remove(id: string, userId: string): Promise<void> {
    const post = await this.findOne(id);
    
    // Check if user is the author or admin
    if (post.authorId !== userId) {
      throw new ForbiddenException('You can only delete your own posts');
    }

    await this.postsRepository.remove(post);
  }

  async findFeatured(): Promise<Post[]> {
    return this.postsRepository.find({
      where: { featured: true, status: 'published' },
      relations: ['author', 'category', 'tags'],
      order: { publishedAt: 'DESC' },
    });
  }

  async searchPosts(searchTerm: string): Promise<Post[]> {
    return this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .leftJoinAndSelect('post.category', 'category')
      .leftJoinAndSelect('post.tags', 'tags')
      .where('post.title ILIKE :searchTerm', { searchTerm: `%${searchTerm}%` })
      .orWhere('post.content ILIKE :searchTerm', { searchTerm: `%${searchTerm}%` })
      .andWhere('post.status = :status', { status: 'published' })
      .orderBy('post.publishedAt', 'DESC')
      .getMany();
  }

  private createQueryBuilder(query: PostQueryDto): SelectQueryBuilder<Post> {
    const queryBuilder = this.postsRepository
      .createQueryBuilder('post')
      .leftJoinAndSelect('post.author', 'author')
      .leftJoinAndSelect('post.category', 'category')
      .leftJoinAndSelect('post.tags', 'tags');

    if (query.categoryId) {
      queryBuilder.andWhere('post.categoryId = :categoryId', { categoryId: query.categoryId });
    }

    if (query.tagId) {
      queryBuilder.andWhere('tags.id = :tagId', { tagId: query.tagId });
    }

    if (query.status) {
      queryBuilder.andWhere('post.status = :status', { status: query.status });
    } else {
      queryBuilder.andWhere('post.status = :status', { status: 'published' });
    }

    if (query.search) {
      queryBuilder.andWhere(
        '(post.title ILIKE :search OR post.content ILIKE :search)',
        { search: `%${query.search}%` }
      );
    }

    queryBuilder.orderBy('post.publishedAt', 'DESC');
    return queryBuilder;
  }

  private generateSlug(title: string): string {
    return title
      .toLowerCase()
      .replace(/[^a-z0-9 -]/g, '')
      .replace(/\s+/g, '-')
      .replace(/-+/g, '-')
      .trim('-');
  }
}
```

### **Advanced Features**

#### **File Upload Service**
```typescript
import { Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { S3 } from 'aws-sdk';
import { v4 as uuid } from 'uuid';

@Injectable()
export class FileUploadService {
  private s3: S3;

  constructor(private configService: ConfigService) {
    this.s3 = new S3({
      accessKeyId: this.configService.get('AWS_ACCESS_KEY_ID'),
      secretAccessKey: this.configService.get('AWS_SECRET_ACCESS_KEY'),
      region: this.configService.get('AWS_REGION'),
    });
  }

  async uploadFile(file: Express.Multer.File, folder: string): Promise<string> {
    const key = `${folder}/${uuid()}-${file.originalname}`;
    
    const uploadResult = await this.s3
      .upload({
        Bucket: this.configService.get('AWS_S3_BUCKET'),
        Body: file.buffer,
        Key: key,
        ContentType: file.mimetype,
      })
      .promise();

    return uploadResult.Location;
  }

  async deleteFile(key: string): Promise<void> {
    await this.s3
      .deleteObject({
        Bucket: this.configService.get('AWS_S3_BUCKET'),
        Key: key,
      })
      .promise();
  }
}
```

#### **Comment Service with Nested Structure**
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Comment } from './entities/comment.entity';
import { CreateCommentDto } from './dto/create-comment.dto';

@Injectable()
export class CommentsService {
  constructor(
    @InjectRepository(Comment)
    private commentsRepository: Repository<Comment>,
  ) {}

  async create(createCommentDto: CreateCommentDto, authorId: string): Promise<Comment> {
    const comment = this.commentsRepository.create({
      ...createCommentDto,
      authorId,
    });
    return this.commentsRepository.save(comment);
  }

  async findByPost(postId: string): Promise<Comment[]> {
    return this.commentsRepository.find({
      where: { postId, isApproved: true, isSpam: false },
      relations: ['author', 'replies'],
      order: { createdAt: 'ASC' },
    });
  }

  async findReplies(parentId: string): Promise<Comment[]> {
    return this.commentsRepository.find({
      where: { parentId, isApproved: true, isSpam: false },
      relations: ['author'],
      order: { createdAt: 'ASC' },
    });
  }

  async approve(id: string): Promise<Comment> {
    const comment = await this.commentsRepository.findOne({ where: { id } });
    if (!comment) {
      throw new NotFoundException('Comment not found');
    }
    comment.isApproved = true;
    return this.commentsRepository.save(comment);
  }

  async markAsSpam(id: string): Promise<Comment> {
    const comment = await this.commentsRepository.findOne({ where: { id } });
    if (!comment) {
      throw new NotFoundException('Comment not found');
    }
    comment.isSpam = true;
    return this.commentsRepository.save(comment);
  }
}
```

## Deliverables

### **Code Repository**
- Well-structured NestJS application
- Advanced TypeScript implementation
- Comprehensive database schema
- File upload functionality
- Complete test suite

### **Documentation**
- API documentation (Swagger)
- Database schema documentation
- Authentication flow documentation
- File upload guide
- Deployment instructions

### **Demo Application**
- Fully functional blog management system
- User authentication and authorization
- Content management features
- Commenting system
- Admin panel functionality

## Success Criteria

### **Functional Requirements**
- [ ] User registration and authentication
- [ ] Role-based access control
- [ ] Blog post management
- [ ] Commenting system
- [ ] File upload functionality

### **Technical Requirements**
- [ ] Advanced TypeScript usage
- [ ] Complex database relationships
- [ ] RESTful API design
- [ ] Comprehensive testing
- [ ] Security best practices

### **Quality Requirements**
- [ ] Clean architecture
- [ ] Error handling
- [ ] Input validation
- [ ] Performance optimization
- [ ] Documentation completeness

## Next Steps

1. **Set up advanced authentication**
2. **Implement content management**
3. **Add commenting system**
4. **Integrate file upload**
5. **Write comprehensive tests**
6. **Deploy and document**

Ready to build your blog management system? Let's start coding! 🚀
