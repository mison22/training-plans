# Project 1: Personal Portfolio API
*3-week project for TypeScript Backend Evening (Weeks 1-3)*

## Project Overview
Create a personal portfolio API that showcases your skills and projects. This project focuses on TypeScript fundamentals, NestJS basics, and basic CRUD operations while building a practical application you can use for your own portfolio.

## Learning Objectives
- Master TypeScript language features and best practices
- Learn NestJS framework fundamentals
- Implement basic CRUD operations
- Set up database integration with TypeORM
- Create a well-documented REST API

## Technology Stack
- **Language**: TypeScript 5.0+
- **Framework**: NestJS
- **Database**: PostgreSQL
- **ORM**: TypeORM
- **Authentication**: JWT
- **Documentation**: Swagger/OpenAPI

## Project Timeline (3 Weeks)

### **Week 1: Project Setup & Basic API**
**Goal**: Set up project structure and create basic API endpoints

#### **Day 1-2: Project Setup**
- Initialize NestJS project with TypeScript
- Configure TypeScript compiler options
- Set up project structure and modules
- Install and configure TypeORM
- Set up PostgreSQL database

#### **Day 3-4: Basic API Structure**
- Create main application module
- Set up routing and controllers
- Implement basic CRUD operations
- Add request validation with DTOs
- Set up error handling

#### **Day 5: Database Integration**
- Create database entities
- Set up database migrations
- Implement repository pattern
- Add database seeding
- Test database operations

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Theory and setup
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 2: Core Features Implementation**
**Goal**: Implement core portfolio features

#### **Day 1-2: User Management**
- User registration and login
- JWT token implementation
- Password hashing with bcrypt
- User profile management
- Input validation and sanitization

#### **Day 3-4: Portfolio Content**
- Project management (CRUD)
- Skill management (CRUD)
- Experience management (CRUD)
- Education management (CRUD)
- File upload for images

#### **Day 5: API Documentation**
- Set up Swagger/OpenAPI
- Document all endpoints
- Add request/response examples
- Create API testing collection
- Write API documentation

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Feature implementation
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 3: Advanced Features & Testing**
**Goal**: Add advanced features and comprehensive testing

#### **Day 1-2: Advanced Features**
- Search and filtering
- Pagination for large datasets
- Sorting and ordering
- Data validation and constraints
- Error handling improvements

#### **Day 3-4: Testing Implementation**
- Unit tests for services
- Integration tests for controllers
- Database testing setup
- Mock data and fixtures
- Test coverage analysis

#### **Day 5: Documentation & Deployment**
- Complete API documentation
- Set up Docker containerization
- Basic deployment preparation
- Performance optimization
- Code review and refactoring

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Advanced features
- **8:00 PM - 9:00 PM**: Testing and deployment
- **9:00 PM - 9:30 PM**: Review and documentation

## Detailed Requirements

### **Database Schema**

#### **Users Table**
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    bio TEXT,
    avatar_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Projects Table**
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    technologies TEXT[],
    github_url VARCHAR(500),
    live_url VARCHAR(500),
    image_url VARCHAR(500),
    featured BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Skills Table**
```sql
CREATE TABLE skills (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    name VARCHAR(100) NOT NULL,
    category VARCHAR(50) NOT NULL, -- 'frontend', 'backend', 'database', 'tools'
    proficiency INTEGER CHECK (proficiency >= 1 AND proficiency <= 5),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Experience Table**
```sql
CREATE TABLE experience (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    company VARCHAR(255) NOT NULL,
    position VARCHAR(255) NOT NULL,
    description TEXT,
    start_date DATE NOT NULL,
    end_date DATE,
    current BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### **API Endpoints**

#### **Authentication**
```
POST   /auth/register
POST   /auth/login
POST   /auth/refresh
POST   /auth/logout
GET    /auth/profile
PUT    /auth/profile
```

#### **Projects**
```
GET    /projects
GET    /projects/:id
POST   /projects
PUT    /projects/:id
DELETE /projects/:id
GET    /projects/featured
```

#### **Skills**
```
GET    /skills
GET    /skills/:id
POST   /skills
PUT    /skills/:id
DELETE /skills/:id
GET    /skills/category/:category
```

#### **Experience**
```
GET    /experience
GET    /experience/:id
POST   /experience
PUT    /experience/:id
DELETE /experience/:id
```

### **TypeScript Implementation**

#### **User Entity**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, OneToMany } from 'typeorm';
import { Project } from './project.entity';
import { Skill } from './skill.entity';
import { Experience } from './experience.entity';

@Entity('users')
export class User {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column({ unique: true })
  email: string;

  @Column()
  passwordHash: string;

  @Column()
  firstName: string;

  @Column()
  lastName: string;

  @Column({ nullable: true })
  bio: string;

  @Column({ nullable: true })
  avatarUrl: string;

  @OneToMany(() => Project, project => project.user)
  projects: Project[];

  @OneToMany(() => Skill, skill => skill.user)
  skills: Skill[];

  @OneToMany(() => Experience, experience => experience.user)
  experience: Experience[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

#### **Project Entity**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, ManyToOne, JoinColumn } from 'typeorm';
import { User } from './user.entity';

@Entity('projects')
export class Project {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  userId: string;

  @Column()
  title: string;

  @Column({ nullable: true })
  description: string;

  @Column('text', { array: true, default: [] })
  technologies: string[];

  @Column({ nullable: true })
  githubUrl: string;

  @Column({ nullable: true })
  liveUrl: string;

  @Column({ nullable: true })
  imageUrl: string;

  @Column({ default: false })
  featured: boolean;

  @ManyToOne(() => User, user => user.projects, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'userId' })
  user: User;

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

#### **Project Service**
```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Project } from './entities/project.entity';
import { CreateProjectDto } from './dto/create-project.dto';
import { UpdateProjectDto } from './dto/update-project.dto';

@Injectable()
export class ProjectsService {
  constructor(
    @InjectRepository(Project)
    private projectsRepository: Repository<Project>,
  ) {}

  async create(createProjectDto: CreateProjectDto, userId: string): Promise<Project> {
    const project = this.projectsRepository.create({
      ...createProjectDto,
      userId,
    });
    return this.projectsRepository.save(project);
  }

  async findAll(userId: string): Promise<Project[]> {
    return this.projectsRepository.find({
      where: { userId },
      order: { createdAt: 'DESC' },
    });
  }

  async findOne(id: string, userId: string): Promise<Project> {
    const project = await this.projectsRepository.findOne({
      where: { id, userId },
    });
    if (!project) {
      throw new NotFoundException('Project not found');
    }
    return project;
  }

  async update(id: string, updateProjectDto: UpdateProjectDto, userId: string): Promise<Project> {
    const project = await this.findOne(id, userId);
    Object.assign(project, updateProjectDto);
    return this.projectsRepository.save(project);
  }

  async remove(id: string, userId: string): Promise<void> {
    const project = await this.findOne(id, userId);
    await this.projectsRepository.remove(project);
  }

  async findFeatured(userId: string): Promise<Project[]> {
    return this.projectsRepository.find({
      where: { userId, featured: true },
      order: { createdAt: 'DESC' },
    });
  }
}
```

### **Testing Requirements**

#### **Unit Tests**
- Test all service methods
- Test validation logic
- Test error handling
- Mock database operations
- Achieve 90%+ coverage

#### **Integration Tests**
- Test API endpoints
- Test authentication flow
- Test database operations
- Test error responses
- Test validation

#### **Test Example**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { ProjectsService } from './projects.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Project } from './entities/project.entity';

describe('ProjectsService', () => {
  let service: ProjectsService;
  let repository: Repository<Project>;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        ProjectsService,
        {
          provide: getRepositoryToken(Project),
          useValue: {
            create: jest.fn(),
            save: jest.fn(),
            find: jest.fn(),
            findOne: jest.fn(),
            remove: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<ProjectsService>(ProjectsService);
    repository = module.get<Repository<Project>>(getRepositoryToken(Project));
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('create', () => {
    it('should create a new project', async () => {
      const createProjectDto = {
        title: 'Test Project',
        description: 'Test Description',
        technologies: ['TypeScript', 'NestJS'],
      };
      const userId = 'user-id';
      const project = { id: 'project-id', ...createProjectDto, userId };

      jest.spyOn(repository, 'create').mockReturnValue(project as Project);
      jest.spyOn(repository, 'save').mockResolvedValue(project as Project);

      const result = await service.create(createProjectDto, userId);
      expect(result).toEqual(project);
      expect(repository.create).toHaveBeenCalledWith({ ...createProjectDto, userId });
      expect(repository.save).toHaveBeenCalledWith(project);
    });
  });
});
```

## Deliverables

### **Code Repository**
- Well-structured NestJS application
- TypeScript configuration
- Database migrations
- Comprehensive test suite
- API documentation

### **Documentation**
- README with setup instructions
- API documentation (Swagger)
- Database schema documentation
- Testing guide
- Deployment guide

### **Demo Application**
- Fully functional portfolio API
- Test data and examples
- API testing collection
- Performance benchmarks
- Security audit

## Success Criteria

### **Functional Requirements**
- [ ] User registration and authentication
- [ ] CRUD operations for all entities
- [ ] JWT token authentication
- [ ] Input validation and error handling
- [ ] API documentation

### **Technical Requirements**
- [ ] TypeScript best practices
- [ ] NestJS framework usage
- [ ] Database integration with TypeORM
- [ ] Comprehensive testing (90%+ coverage)
- [ ] Clean architecture

### **Quality Requirements**
- [ ] Code documentation
- [ ] Error handling
- [ ] Security best practices
- [ ] Performance optimization
- [ ] Deployment readiness

## Next Steps

1. **Set up development environment**
2. **Initialize NestJS project**
3. **Configure database and TypeORM**
4. **Implement basic CRUD operations**
5. **Add authentication and validation**
6. **Write comprehensive tests**
7. **Deploy and document**

Ready to build your personal portfolio API? Let's start coding! 🚀
