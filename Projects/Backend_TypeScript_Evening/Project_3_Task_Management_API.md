# Project 3: Task Management API
*3-week project for TypeScript Backend Evening (Weeks 7-9)*

## Project Overview
Create a comprehensive task management API with real-time features, advanced filtering, and team collaboration. This project focuses on advanced TypeScript features, testing strategies, and performance optimization while building a practical project management tool.

## Learning Objectives
- Master advanced TypeScript features and patterns
- Implement comprehensive testing strategies
- Add real-time features with WebSockets
- Optimize application performance
- Build scalable and maintainable code

## Technology Stack
- **Language**: TypeScript 5.0+
- **Framework**: NestJS
- **Database**: PostgreSQL
- **ORM**: TypeORM
- **Caching**: Redis
- **Real-time**: WebSockets
- **Testing**: Jest, Supertest
- **Containerization**: Docker

## Project Timeline (3 Weeks)

### **Week 7: Advanced Features & Real-time**
**Goal**: Implement advanced features and real-time functionality

#### **Day 1-2: WebSocket Integration**
- WebSocket gateway setup
- Real-time task updates
- Live collaboration features
- Connection management
- Event broadcasting

#### **Day 3-4: Advanced Task Features**
- Task dependencies and subtasks
- Task templates and recurring tasks
- Time tracking and reporting
- File attachments
- Task comments and activity logs

#### **Day 5: Team Collaboration**
- Team and project management
- User roles and permissions
- Task assignment and delegation
- Notification system
- Activity feeds

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Advanced features and WebSocket theory
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 8: Testing & Quality Assurance**
**Goal**: Implement comprehensive testing and quality assurance

#### **Day 1-2: Unit Testing**
- Service layer testing
- Repository testing
- Utility function testing
- Mock strategies
- Test coverage analysis

#### **Day 3-4: Integration Testing**
- API endpoint testing
- Database integration testing
- WebSocket testing
- Authentication testing
- Error handling testing

#### **Day 5: End-to-End Testing**
- Complete workflow testing
- User journey testing
- Performance testing
- Load testing
- Security testing

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Testing strategies and implementation
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

### **Week 9: Performance & Optimization**
**Goal**: Optimize application performance and add monitoring

#### **Day 1-2: Performance Optimization**
- Database query optimization
- Caching strategies with Redis
- Memory optimization
- API response optimization
- Connection pooling

#### **Day 3-4: Monitoring & Logging**
- Application monitoring
- Error tracking and alerting
- Performance metrics
- Log aggregation
- Health checks

#### **Day 5: Containerization & Deployment**
- Docker containerization
- Docker Compose setup
- Production configuration
- Environment management
- Deployment preparation

#### **Evening Schedule (2.5 hours)**
- **7:00 PM - 8:00 PM**: Performance optimization and monitoring
- **8:00 PM - 9:00 PM**: Hands-on coding
- **9:00 PM - 9:30 PM**: Review and testing

## Detailed Requirements

### **Database Schema**

#### **Projects Table**
```sql
CREATE TABLE projects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    owner_id UUID REFERENCES users(id) ON DELETE CASCADE,
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'completed', 'archived'
    start_date DATE,
    end_date DATE,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Tasks Table**
```sql
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
    parent_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT,
    status VARCHAR(20) DEFAULT 'todo', -- 'todo', 'in_progress', 'completed', 'cancelled'
    priority VARCHAR(10) DEFAULT 'medium', -- 'low', 'medium', 'high', 'urgent'
    assignee_id UUID REFERENCES users(id) ON DELETE SET NULL,
    creator_id UUID REFERENCES users(id) ON DELETE CASCADE,
    due_date TIMESTAMP,
    completed_at TIMESTAMP,
    estimated_hours DECIMAL(5,2),
    actual_hours DECIMAL(5,2),
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Task Dependencies Table**
```sql
CREATE TABLE task_dependencies (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    depends_on_task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(task_id, depends_on_task_id)
);
```

#### **Task Comments Table**
```sql
CREATE TABLE task_comments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    author_id UUID REFERENCES users(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

#### **Task Attachments Table**
```sql
CREATE TABLE task_attachments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    filename VARCHAR(255) NOT NULL,
    original_name VARCHAR(255) NOT NULL,
    file_size INTEGER NOT NULL,
    mime_type VARCHAR(100) NOT NULL,
    file_url VARCHAR(500) NOT NULL,
    uploaded_by UUID REFERENCES users(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW()
);
```

#### **Time Entries Table**
```sql
CREATE TABLE time_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID REFERENCES tasks(id) ON DELETE CASCADE,
    user_id UUID REFERENCES users(id) ON DELETE CASCADE,
    description TEXT,
    hours DECIMAL(5,2) NOT NULL,
    date DATE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);
```

### **API Endpoints**

#### **Projects**
```
GET    /projects
GET    /projects/:id
POST   /projects
PUT    /projects/:id
DELETE /projects/:id
GET    /projects/:id/tasks
GET    /projects/:id/members
POST   /projects/:id/members
DELETE /projects/:id/members/:userId
```

#### **Tasks**
```
GET    /tasks
GET    /tasks/:id
POST   /tasks
PUT    /tasks/:id
DELETE /tasks/:id
GET    /tasks/:id/subtasks
GET    /tasks/:id/dependencies
POST   /tasks/:id/dependencies
DELETE /tasks/:id/dependencies/:dependsOnId
GET    /tasks/:id/comments
POST   /tasks/:id/comments
GET    /tasks/:id/attachments
POST   /tasks/:id/attachments
GET    /tasks/:id/time-entries
POST   /tasks/:id/time-entries
```

#### **WebSocket Events**
```
task.created
task.updated
task.deleted
task.assigned
task.completed
comment.added
attachment.uploaded
time.entry.added
```

### **TypeScript Implementation**

#### **Task Entity with Advanced Features**
```typescript
import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn, ManyToOne, OneToMany, ManyToMany, JoinColumn, JoinTable } from 'typeorm';
import { Project } from './project.entity';
import { User } from './user.entity';
import { TaskComment } from './task-comment.entity';
import { TaskAttachment } from './task-attachment.entity';
import { TimeEntry } from './time-entry.entity';

@Entity('tasks')
export class Task {
  @PrimaryGeneratedColumn('uuid')
  id: string;

  @Column()
  projectId: string;

  @Column({ nullable: true })
  parentId: string;

  @Column()
  title: string;

  @Column({ nullable: true })
  description: string;

  @Column({ default: 'todo' })
  status: 'todo' | 'in_progress' | 'completed' | 'cancelled';

  @Column({ default: 'medium' })
  priority: 'low' | 'medium' | 'high' | 'urgent';

  @Column({ nullable: true })
  assigneeId: string;

  @Column()
  creatorId: string;

  @Column({ nullable: true })
  dueDate: Date;

  @Column({ nullable: true })
  completedAt: Date;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  estimatedHours: number;

  @Column({ type: 'decimal', precision: 5, scale: 2, nullable: true })
  actualHours: number;

  @ManyToOne(() => Project, project => project.tasks, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'projectId' })
  project: Project;

  @ManyToOne(() => User, user => user.assignedTasks, { onDelete: 'SET NULL' })
  @JoinColumn({ name: 'assigneeId' })
  assignee: User;

  @ManyToOne(() => User, user => user.createdTasks, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'creatorId' })
  creator: User;

  @ManyToOne(() => Task, task => task.subtasks, { onDelete: 'CASCADE' })
  @JoinColumn({ name: 'parentId' })
  parent: Task;

  @OneToMany(() => Task, task => task.parent)
  subtasks: Task[];

  @OneToMany(() => TaskComment, comment => comment.task)
  comments: TaskComment[];

  @OneToMany(() => TaskAttachment, attachment => attachment.task)
  attachments: TaskAttachment[];

  @OneToMany(() => TimeEntry, timeEntry => timeEntry.task)
  timeEntries: TimeEntry[];

  @ManyToMany(() => Task, task => task.dependsOn, { cascade: true })
  @JoinTable({
    name: 'task_dependencies',
    joinColumn: { name: 'taskId', referencedColumnName: 'id' },
    inverseJoinColumn: { name: 'dependsOnTaskId', referencedColumnName: 'id' },
  })
  dependencies: Task[];

  @ManyToMany(() => Task, task => task.dependencies)
  dependsOn: Task[];

  @CreateDateColumn()
  createdAt: Date;

  @UpdateDateColumn()
  updatedAt: Date;
}
```

#### **WebSocket Gateway**
```typescript
import { WebSocketGateway, WebSocketServer, SubscribeMessage, OnGatewayConnection, OnGatewayDisconnect } from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';
import { UseGuards } from '@nestjs/common';
import { JwtAuthGuard } from '../auth/jwt-auth.guard';
import { TasksService } from './tasks.service';

@WebSocketGateway({
  cors: {
    origin: process.env.CLIENT_URL,
    credentials: true,
  },
})
export class TasksGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  private connectedUsers = new Map<string, string>();

  constructor(private tasksService: TasksService) {}

  handleConnection(client: Socket) {
    const userId = client.handshake.auth.userId;
    if (userId) {
      this.connectedUsers.set(client.id, userId);
      client.join(`user:${userId}`);
    }
  }

  handleDisconnect(client: Socket) {
    this.connectedUsers.delete(client.id);
  }

  @UseGuards(JwtAuthGuard)
  @SubscribeMessage('join_project')
  handleJoinProject(client: Socket, projectId: string) {
    client.join(`project:${projectId}`);
  }

  @UseGuards(JwtAuthGuard)
  @SubscribeMessage('leave_project')
  handleLeaveProject(client: Socket, projectId: string) {
    client.leave(`project:${projectId}`);
  }

  async emitTaskCreated(task: any, projectId: string) {
    this.server.to(`project:${projectId}`).emit('task.created', task);
  }

  async emitTaskUpdated(task: any, projectId: string) {
    this.server.to(`project:${projectId}`).emit('task.updated', task);
  }

  async emitTaskDeleted(taskId: string, projectId: string) {
    this.server.to(`project:${projectId}`).emit('task.deleted', { taskId });
  }

  async emitCommentAdded(comment: any, projectId: string) {
    this.server.to(`project:${projectId}`).emit('comment.added', comment);
  }
}
```

#### **Advanced Tasks Service**
```typescript
import { Injectable, NotFoundException, ForbiddenException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, SelectQueryBuilder } from 'typeorm';
import { Task } from './entities/task.entity';
import { CreateTaskDto } from './dto/create-task.dto';
import { UpdateTaskDto } from './dto/update-task.dto';
import { TaskQueryDto } from './dto/task-query.dto';
import { TasksGateway } from './tasks.gateway';

@Injectable()
export class TasksService {
  constructor(
    @InjectRepository(Task)
    private tasksRepository: Repository<Task>,
    private tasksGateway: TasksGateway,
  ) {}

  async create(createTaskDto: CreateTaskDto, creatorId: string): Promise<Task> {
    const task = this.tasksRepository.create({
      ...createTaskDto,
      creatorId,
    });
    
    const savedTask = await this.tasksRepository.save(task);
    
    // Emit real-time event
    await this.tasksGateway.emitTaskCreated(savedTask, createTaskDto.projectId);
    
    return savedTask;
  }

  async findAll(query: TaskQueryDto): Promise<{ tasks: Task[]; total: number }> {
    const queryBuilder = this.createQueryBuilder(query);
    
    const [tasks, total] = await queryBuilder
      .skip((query.page - 1) * query.limit)
      .take(query.limit)
      .getManyAndCount();

    return { tasks, total };
  }

  async findOne(id: string): Promise<Task> {
    const task = await this.tasksRepository.findOne({
      where: { id },
      relations: ['project', 'assignee', 'creator', 'subtasks', 'comments', 'attachments', 'timeEntries', 'dependencies'],
    });
    if (!task) {
      throw new NotFoundException('Task not found');
    }
    return task;
  }

  async update(id: string, updateTaskDto: UpdateTaskDto, userId: string): Promise<Task> {
    const task = await this.findOne(id);
    
    // Check permissions
    if (task.creatorId !== userId && task.assigneeId !== userId) {
      throw new ForbiddenException('You can only update tasks you created or are assigned to');
    }

    Object.assign(task, updateTaskDto);
    
    // Update completion status
    if (updateTaskDto.status === 'completed' && !task.completedAt) {
      task.completedAt = new Date();
    } else if (updateTaskDto.status !== 'completed' && task.completedAt) {
      task.completedAt = null;
    }

    const savedTask = await this.tasksRepository.save(task);
    
    // Emit real-time event
    await this.tasksGateway.emitTaskUpdated(savedTask, task.projectId);
    
    return savedTask;
  }

  async remove(id: string, userId: string): Promise<void> {
    const task = await this.findOne(id);
    
    // Check permissions
    if (task.creatorId !== userId) {
      throw new ForbiddenException('You can only delete tasks you created');
    }

    await this.tasksRepository.remove(task);
    
    // Emit real-time event
    await this.tasksGateway.emitTaskDeleted(id, task.projectId);
  }

  async findSubtasks(parentId: string): Promise<Task[]> {
    return this.tasksRepository.find({
      where: { parentId },
      relations: ['assignee', 'creator'],
      order: { createdAt: 'ASC' },
    });
  }

  async addDependency(taskId: string, dependsOnTaskId: string): Promise<void> {
    const task = await this.findOne(taskId);
    const dependsOnTask = await this.findOne(dependsOnTaskId);
    
    // Check for circular dependencies
    if (await this.wouldCreateCircularDependency(taskId, dependsOnTaskId)) {
      throw new ForbiddenException('This would create a circular dependency');
    }

    task.dependencies = [...(task.dependencies || []), dependsOnTask];
    await this.tasksRepository.save(task);
  }

  async removeDependency(taskId: string, dependsOnTaskId: string): Promise<void> {
    const task = await this.findOne(taskId);
    task.dependencies = task.dependencies?.filter(dep => dep.id !== dependsOnTaskId) || [];
    await this.tasksRepository.save(task);
  }

  async getTaskStatistics(projectId: string): Promise<any> {
    const stats = await this.tasksRepository
      .createQueryBuilder('task')
      .select('task.status', 'status')
      .addSelect('COUNT(*)', 'count')
      .where('task.projectId = :projectId', { projectId })
      .groupBy('task.status')
      .getRawMany();

    return stats.reduce((acc, stat) => {
      acc[stat.status] = parseInt(stat.count);
      return acc;
    }, {});
  }

  private createQueryBuilder(query: TaskQueryDto): SelectQueryBuilder<Task> {
    const queryBuilder = this.tasksRepository
      .createQueryBuilder('task')
      .leftJoinAndSelect('task.project', 'project')
      .leftJoinAndSelect('task.assignee', 'assignee')
      .leftJoinAndSelect('task.creator', 'creator');

    if (query.projectId) {
      queryBuilder.andWhere('task.projectId = :projectId', { projectId: query.projectId });
    }

    if (query.assigneeId) {
      queryBuilder.andWhere('task.assigneeId = :assigneeId', { assigneeId: query.assigneeId });
    }

    if (query.status) {
      queryBuilder.andWhere('task.status = :status', { status: query.status });
    }

    if (query.priority) {
      queryBuilder.andWhere('task.priority = :priority', { priority: query.priority });
    }

    if (query.search) {
      queryBuilder.andWhere(
        '(task.title ILIKE :search OR task.description ILIKE :search)',
        { search: `%${query.search}%` }
      );
    }

    queryBuilder.orderBy('task.createdAt', 'DESC');
    return queryBuilder;
  }

  private async wouldCreateCircularDependency(taskId: string, dependsOnTaskId: string): Promise<boolean> {
    // Implement circular dependency detection algorithm
    const visited = new Set<string>();
    const stack = new Set<string>();
    
    const dfs = async (currentTaskId: string): Promise<boolean> => {
      if (stack.has(currentTaskId)) return true;
      if (visited.has(currentTaskId)) return false;
      
      visited.add(currentTaskId);
      stack.add(currentTaskId);
      
      const task = await this.tasksRepository.findOne({
        where: { id: currentTaskId },
        relations: ['dependencies'],
      });
      
      if (task?.dependencies) {
        for (const dep of task.dependencies) {
          if (await dfs(dep.id)) return true;
        }
      }
      
      stack.delete(currentTaskId);
      return false;
    };
    
    return dfs(dependsOnTaskId);
  }
}
```

### **Testing Implementation**

#### **Unit Test Example**
```typescript
import { Test, TestingModule } from '@nestjs/testing';
import { TasksService } from './tasks.service';
import { getRepositoryToken } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Task } from './entities/task.entity';
import { TasksGateway } from './tasks.gateway';

describe('TasksService', () => {
  let service: TasksService;
  let repository: Repository<Task>;
  let gateway: TasksGateway;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        TasksService,
        {
          provide: getRepositoryToken(Task),
          useValue: {
            create: jest.fn(),
            save: jest.fn(),
            find: jest.fn(),
            findOne: jest.fn(),
            remove: jest.fn(),
            createQueryBuilder: jest.fn(),
          },
        },
        {
          provide: TasksGateway,
          useValue: {
            emitTaskCreated: jest.fn(),
            emitTaskUpdated: jest.fn(),
            emitTaskDeleted: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get<TasksService>(TasksService);
    repository = module.get<Repository<Task>>(getRepositoryToken(Task));
    gateway = module.get<TasksGateway>(TasksGateway);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  describe('create', () => {
    it('should create a new task and emit event', async () => {
      const createTaskDto = {
        title: 'Test Task',
        description: 'Test Description',
        projectId: 'project-id',
      };
      const creatorId = 'user-id';
      const task = { id: 'task-id', ...createTaskDto, creatorId };

      jest.spyOn(repository, 'create').mockReturnValue(task as Task);
      jest.spyOn(repository, 'save').mockResolvedValue(task as Task);
      jest.spyOn(gateway, 'emitTaskCreated').mockResolvedValue(undefined);

      const result = await service.create(createTaskDto, creatorId);
      
      expect(result).toEqual(task);
      expect(repository.create).toHaveBeenCalledWith({ ...createTaskDto, creatorId });
      expect(repository.save).toHaveBeenCalledWith(task);
      expect(gateway.emitTaskCreated).toHaveBeenCalledWith(task, createTaskDto.projectId);
    });
  });
});
```

## Deliverables

### **Code Repository**
- Advanced TypeScript implementation
- WebSocket real-time features
- Comprehensive testing suite
- Performance optimizations
- Docker containerization

### **Documentation**
- API documentation with WebSocket events
- Testing guide and coverage report
- Performance optimization guide
- Docker deployment guide
- Monitoring setup guide

### **Demo Application**
- Fully functional task management system
- Real-time collaboration features
- Advanced filtering and search
- Team management capabilities
- Performance monitoring

## Success Criteria

### **Functional Requirements**
- [ ] Real-time task updates
- [ ] Advanced task features (dependencies, subtasks)
- [ ] Team collaboration
- [ ] Time tracking
- [ ] File attachments

### **Technical Requirements**
- [ ] Advanced TypeScript usage
- [ ] WebSocket implementation
- [ ] Comprehensive testing (90%+ coverage)
- [ ] Performance optimization
- [ ] Docker containerization

### **Quality Requirements**
- [ ] Clean architecture
- [ ] Error handling
- [ ] Security best practices
- [ ] Monitoring and logging
- [ ] Documentation completeness

## Next Steps

1. **Implement WebSocket features**
2. **Add advanced task functionality**
3. **Write comprehensive tests**
4. **Optimize performance**
5. **Set up monitoring**
6. **Containerize application**

Ready to build your task management API? Let's start coding! 🚀
