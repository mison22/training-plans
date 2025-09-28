# Project 3: Project Management Tool
*Full Stack Intensive Plan - Weeks 5-6*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html/)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 5-6)  
**Tech Stack**: Vue.js + TypeScript + Django + PostgreSQL + Redis  
**Focus**: Complex state management, advanced UI components, and full-stack integration  

### Learning Objectives
- Master Vue.js with TypeScript and Composition API
- Build Django REST API with advanced features
- Implement complex state management with Pinia
- Handle file uploads and media processing
- Create advanced UI components and data visualization

---

## 📊 Database Schema Design (Django Models)

### Projects Model
```python
# models.py
from django.db import models
from django.contrib.auth.models import User
import uuid

class Project(models.Model):
    STATUS_CHOICES = [
        ('planning', 'Planning'),
        ('active', 'Active'),
        ('on_hold', 'On Hold'),
        ('completed', 'Completed'),
        ('cancelled', 'Cancelled'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='planning')
    start_date = models.DateField(null=True, blank=True)
    end_date = models.DateField(null=True, blank=True)
    budget = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    owner = models.ForeignKey(User, on_delete=models.CASCADE, related_name='owned_projects')
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']

class ProjectMember(models.Model):
    ROLE_CHOICES = [
        ('owner', 'Owner'),
        ('admin', 'Admin'),
        ('member', 'Member'),
        ('viewer', 'Viewer'),
    ]
    
    project = models.ForeignKey(Project, on_delete=models.CASCADE, related_name='members')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    role = models.CharField(max_length=20, choices=ROLE_CHOICES, default='member')
    joined_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ['project', 'user']

class Task(models.Model):
    PRIORITY_CHOICES = [
        ('low', 'Low'),
        ('medium', 'Medium'),
        ('high', 'High'),
        ('urgent', 'Urgent'),
    ]
    
    STATUS_CHOICES = [
        ('todo', 'To Do'),
        ('in_progress', 'In Progress'),
        ('review', 'Review'),
        ('done', 'Done'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    title = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    project = models.ForeignKey(Project, on_delete=models.CASCADE, related_name='tasks')
    assignee = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True, related_name='assigned_tasks')
    creator = models.ForeignKey(User, on_delete=models.CASCADE, related_name='created_tasks')
    priority = models.CharField(max_length=20, choices=PRIORITY_CHOICES, default='medium')
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='todo')
    due_date = models.DateTimeField(null=True, blank=True)
    estimated_hours = models.DecimalField(max_digits=5, decimal_places=2, null=True, blank=True)
    actual_hours = models.DecimalField(max_digits=5, decimal_places=2, default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']

class TaskComment(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    task = models.ForeignKey(Task, on_delete=models.CASCADE, related_name='comments')
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['created_at']

class TaskAttachment(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    task = models.ForeignKey(Task, on_delete=models.CASCADE, related_name='attachments')
    file = models.FileField(upload_to='task_attachments/')
    original_name = models.CharField(max_length=255)
    file_size = models.BigIntegerField()
    uploaded_by = models.ForeignKey(User, on_delete=models.CASCADE)
    uploaded_at = models.DateTimeField(auto_now_add=True)

class TimeEntry(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    task = models.ForeignKey(Task, on_delete=models.CASCADE, related_name='time_entries')
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    description = models.CharField(max_length=200, blank=True)
    hours = models.DecimalField(max_digits=5, decimal_places=2)
    date = models.DateField()
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-date']
```

---

## 🛠️ Frontend Components (Vue.js + TypeScript)

### Project Dashboard
```vue
<!-- components/ProjectDashboard.vue -->
<template>
  <div class="project-dashboard">
    <div class="dashboard-header">
      <h1>{{ project?.name }}</h1>
      <div class="project-actions">
        <button @click="showCreateTask = true" class="btn-primary">
          Create Task
        </button>
        <button @click="showProjectSettings = true" class="btn-secondary">
          Settings
        </button>
      </div>
    </div>

    <div class="dashboard-stats">
      <div class="stat-card">
        <h3>{{ taskStats.total }}</h3>
        <p>Total Tasks</p>
      </div>
      <div class="stat-card">
        <h3>{{ taskStats.completed }}</h3>
        <p>Completed</p>
      </div>
      <div class="stat-card">
        <h3>{{ taskStats.inProgress }}</h3>
        <p>In Progress</p>
      </div>
      <div class="stat-card">
        <h3>{{ taskStats.overdue }}</h3>
        <p>Overdue</p>
      </div>
    </div>

    <div class="dashboard-content">
      <div class="tasks-section">
        <TaskBoard 
          :tasks="tasks" 
          @task-updated="handleTaskUpdate"
          @task-moved="handleTaskMove"
        />
      </div>
      
      <div class="activity-section">
        <ActivityFeed :activities="activities" />
      </div>
    </div>

    <!-- Modals -->
    <CreateTaskModal 
      v-if="showCreateTask"
      :project-id="projectId"
      @close="showCreateTask = false"
      @task-created="handleTaskCreated"
    />
    
    <ProjectSettingsModal 
      v-if="showProjectSettings"
      :project="project"
      @close="showProjectSettings = false"
      @project-updated="handleProjectUpdate"
    />
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import { useProjectStore } from '@/stores/project';
import { useTaskStore } from '@/stores/task';
import TaskBoard from '@/components/TaskBoard.vue';
import ActivityFeed from '@/components/ActivityFeed.vue';
import CreateTaskModal from '@/components/CreateTaskModal.vue';
import ProjectSettingsModal from '@/components/ProjectSettingsModal.vue';

interface Task {
  id: string;
  title: string;
  status: string;
  priority: string;
  assignee?: {
    id: string;
    first_name: string;
    last_name: string;
  };
  due_date?: string;
}

interface TaskStats {
  total: number;
  completed: number;
  inProgress: number;
  overdue: number;
}

const route = useRoute();
const projectStore = useProjectStore();
const taskStore = useTaskStore();

const projectId = computed(() => route.params.id as string);
const project = computed(() => projectStore.currentProject);
const tasks = computed(() => taskStore.tasks);
const activities = computed(() => projectStore.activities);

const showCreateTask = ref(false);
const showProjectSettings = ref(false);

const taskStats = computed<TaskStats>(() => {
  const taskList = tasks.value;
  return {
    total: taskList.length,
    completed: taskList.filter(t => t.status === 'done').length,
    inProgress: taskList.filter(t => t.status === 'in_progress').length,
    overdue: taskList.filter(t => 
      t.due_date && new Date(t.due_date) < new Date() && t.status !== 'done'
    ).length
  };
});

onMounted(async () => {
  await Promise.all([
    projectStore.fetchProject(projectId.value),
    taskStore.fetchProjectTasks(projectId.value),
    projectStore.fetchProjectActivities(projectId.value)
  ]);
});

const handleTaskUpdate = (task: Task) => {
  taskStore.updateTask(task);
};

const handleTaskMove = (taskId: string, newStatus: string) => {
  taskStore.moveTask(taskId, newStatus);
};

const handleTaskCreated = (task: Task) => {
  taskStore.addTask(task);
  showCreateTask.value = false;
};

const handleProjectUpdate = (updatedProject: any) => {
  projectStore.updateProject(updatedProject);
  showProjectSettings.value = false;
};
</script>

<style scoped>
.project-dashboard {
  padding: 2rem;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.dashboard-stats {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1rem;
  margin-bottom: 2rem;
}

.stat-card {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.stat-card h3 {
  font-size: 2rem;
  margin: 0;
  color: #2563eb;
}

.dashboard-content {
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 2rem;
}

@media (max-width: 768px) {
  .dashboard-content {
    grid-template-columns: 1fr;
  }
}
</style>
```

### Task Board Component
```vue
<!-- components/TaskBoard.vue -->
<template>
  <div class="task-board">
    <div 
      v-for="column in columns" 
      :key="column.status"
      class="task-column"
    >
      <div class="column-header">
        <h3>{{ column.title }}</h3>
        <span class="task-count">{{ getTasksByStatus(column.status).length }}</span>
      </div>
      
      <div 
        class="task-list"
        @drop="handleDrop($event, column.status)"
        @dragover.prevent
        @dragenter.prevent
      >
        <TaskCard
          v-for="task in getTasksByStatus(column.status)"
          :key="task.id"
          :task="task"
          @drag-start="handleDragStart"
          @task-updated="handleTaskUpdate"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import TaskCard from '@/components/TaskCard.vue';
import type { Task } from '@/types/Task';

interface Props {
  tasks: Task[];
}

interface Emits {
  (e: 'task-updated', task: Task): void;
  (e: 'task-moved', taskId: string, newStatus: string): void;
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();

const columns = [
  { status: 'todo', title: 'To Do', color: '#6b7280' },
  { status: 'in_progress', title: 'In Progress', color: '#3b82f6' },
  { status: 'review', title: 'Review', color: '#f59e0b' },
  { status: 'done', title: 'Done', color: '#10b981' }
];

const getTasksByStatus = (status: string) => {
  return props.tasks.filter(task => task.status === status);
};

let draggedTask: Task | null = null;

const handleDragStart = (task: Task) => {
  draggedTask = task;
};

const handleDrop = (event: DragEvent, newStatus: string) => {
  event.preventDefault();
  
  if (draggedTask && draggedTask.status !== newStatus) {
    emit('task-moved', draggedTask.id, newStatus);
  }
  
  draggedTask = null;
};

const handleTaskUpdate = (task: Task) => {
  emit('task-updated', task);
};
</script>

<style scoped>
.task-board {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 1rem;
  height: 100%;
}

.task-column {
  background: #f8fafc;
  border-radius: 8px;
  padding: 1rem;
  min-height: 500px;
}

.column-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid #e2e8f0;
}

.column-header h3 {
  margin: 0;
  font-size: 1.1rem;
  font-weight: 600;
}

.task-count {
  background: #e2e8f0;
  color: #64748b;
  padding: 0.25rem 0.5rem;
  border-radius: 12px;
  font-size: 0.875rem;
  font-weight: 500;
}

.task-list {
  min-height: 400px;
}

.task-list.drag-over {
  background: #dbeafe;
  border: 2px dashed #3b82f6;
}
</style>
```

### Task Card Component
```vue
<!-- components/TaskCard.vue -->
<template>
  <div 
    class="task-card"
    draggable="true"
    @dragstart="handleDragStart"
    @click="showTaskDetail = true"
  >
    <div class="task-header">
      <h4>{{ task.title }}</h4>
      <div class="task-priority" :class="priorityClass">
        {{ task.priority }}
      </div>
    </div>
    
    <p v-if="task.description" class="task-description">
      {{ task.description }}
    </p>
    
    <div class="task-meta">
      <div v-if="task.assignee" class="assignee">
        <img 
          :src="task.assignee.avatar || '/default-avatar.png'" 
          :alt="task.assignee.first_name"
          class="assignee-avatar"
        />
        <span>{{ task.assignee.first_name }} {{ task.assignee.last_name }}</span>
      </div>
      
      <div v-if="task.due_date" class="due-date" :class="dueDateClass">
        <i class="icon-calendar"></i>
        {{ formatDate(task.due_date) }}
      </div>
    </div>
    
    <div class="task-actions">
      <button @click.stop="toggleStatus" class="btn-small">
        {{ getNextStatusLabel() }}
      </button>
      <button @click.stop="showComments = !showComments" class="btn-small">
        Comments ({{ task.comments_count || 0 }})
      </button>
    </div>
    
    <div v-if="showComments" class="task-comments">
      <TaskComments :task-id="task.id" />
    </div>
  </div>

  <!-- Task Detail Modal -->
  <TaskDetailModal 
    v-if="showTaskDetail"
    :task="task"
    @close="showTaskDetail = false"
    @task-updated="handleTaskUpdate"
  />
</template>

<script setup lang="ts">
import { computed, ref } from 'vue';
import TaskComments from '@/components/TaskComments.vue';
import TaskDetailModal from '@/components/TaskDetailModal.vue';
import { formatDate } from '@/utils/date';
import type { Task } from '@/types/Task';

interface Props {
  task: Task;
}

interface Emits {
  (e: 'drag-start', task: Task): void;
  (e: 'task-updated', task: Task): void;
}

const props = defineProps<Props>();
const emit = defineEmits<Emits>();

const showTaskDetail = ref(false);
const showComments = ref(false);

const priorityClass = computed(() => {
  const classes = {
    low: 'priority-low',
    medium: 'priority-medium',
    high: 'priority-high',
    urgent: 'priority-urgent'
  };
  return classes[props.task.priority as keyof typeof classes] || 'priority-medium';
});

const dueDateClass = computed(() => {
  if (!props.task.due_date) return '';
  
  const dueDate = new Date(props.task.due_date);
  const today = new Date();
  const diffDays = Math.ceil((dueDate.getTime() - today.getTime()) / (1000 * 3600 * 24));
  
  if (diffDays < 0) return 'overdue';
  if (diffDays <= 3) return 'due-soon';
  return '';
});

const statusFlow = {
  todo: 'in_progress',
  in_progress: 'review',
  review: 'done',
  done: 'todo'
};

const statusLabels = {
  todo: 'Start',
  in_progress: 'Send for Review',
  review: 'Mark Done',
  done: 'Reopen'
};

const handleDragStart = () => {
  emit('drag-start', props.task);
};

const toggleStatus = () => {
  const newStatus = statusFlow[props.task.status as keyof typeof statusFlow];
  const updatedTask = { ...props.task, status: newStatus };
  emit('task-updated', updatedTask);
};

const getNextStatusLabel = () => {
  const nextStatus = statusFlow[props.task.status as keyof typeof statusFlow];
  return statusLabels[nextStatus as keyof typeof statusLabels];
};

const handleTaskUpdate = (updatedTask: Task) => {
  emit('task-updated', updatedTask);
};
</script>

<style scoped>
.task-card {
  background: white;
  border-radius: 8px;
  padding: 1rem;
  margin-bottom: 0.75rem;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  cursor: pointer;
  transition: all 0.2s ease;
}

.task-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transform: translateY(-2px);
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 0.5rem;
}

.task-header h4 {
  margin: 0;
  font-size: 1rem;
  font-weight: 600;
  color: #1f2937;
  flex: 1;
  margin-right: 0.5rem;
}

.task-priority {
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 500;
  text-transform: uppercase;
}

.priority-low { background: #d1fae5; color: #065f46; }
.priority-medium { background: #fef3c7; color: #92400e; }
.priority-high { background: #fed7d7; color: #991b1b; }
.priority-urgent { background: #fecaca; color: #7f1d1d; }

.task-description {
  color: #6b7280;
  font-size: 0.875rem;
  margin-bottom: 1rem;
  line-height: 1.4;
}

.task-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  font-size: 0.875rem;
}

.assignee {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.assignee-avatar {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  object-fit: cover;
}

.due-date {
  display: flex;
  align-items: center;
  gap: 0.25rem;
  color: #6b7280;
}

.due-date.overdue {
  color: #dc2626;
  font-weight: 500;
}

.due-date.due-soon {
  color: #f59e0b;
  font-weight: 500;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-small {
  padding: 0.25rem 0.5rem;
  font-size: 0.75rem;
  border: 1px solid #d1d5db;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn-small:hover {
  background: #f3f4f6;
}

.task-comments {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid #e5e7eb;
}
</style>
```

---

## 🛠️ Backend API (Django)

### Views
```python
# views.py
from rest_framework import generics, status
from rest_framework.decorators import api_view, permission_classes
from rest_framework.permissions import IsAuthenticated
from rest_framework.response import Response
from django.shortcuts import get_object_or_404
from django.db.models import Q, Count, Sum
from django.utils import timezone

from .models import Project, Task, ProjectMember, TimeEntry
from .serializers import (
    ProjectSerializer, TaskSerializer, TaskCreateSerializer,
    TimeEntrySerializer, ProjectStatsSerializer
)

class ProjectListCreateView(generics.ListCreateAPIView):
    serializer_class = ProjectSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Project.objects.filter(
            Q(owner=self.request.user) | 
            Q(members__user=self.request.user)
        ).distinct().order_by('-created_at')

    def perform_create(self, serializer):
        project = serializer.save(owner=self.request.user)
        # Add owner as project member
        ProjectMember.objects.create(
            project=project,
            user=self.request.user,
            role='owner'
        )

class ProjectDetailView(generics.RetrieveUpdateDestroyAPIView):
    serializer_class = ProjectSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        return Project.objects.filter(
            Q(owner=self.request.user) | 
            Q(members__user=self.request.user)
        ).distinct()

class TaskListCreateView(generics.ListCreateAPIView):
    permission_classes = [IsAuthenticated]

    def get_serializer_class(self):
        if self.request.method == 'POST':
            return TaskCreateSerializer
        return TaskSerializer

    def get_queryset(self):
        project_id = self.kwargs['project_id']
        return Task.objects.filter(
            project_id=project_id,
            project__members__user=self.request.user
        ).select_related('assignee', 'creator').order_by('-created_at')

    def perform_create(self, serializer):
        project = get_object_or_404(
            Project.objects.filter(
                Q(owner=self.request.user) | 
                Q(members__user=self.request.user)
            ),
            id=self.kwargs['project_id']
        )
        serializer.save(creator=self.request.user, project=project)

@api_view(['POST'])
@permission_classes([IsAuthenticated])
def move_task(request, task_id):
    """Move task to different status"""
    task = get_object_or_404(
        Task.objects.filter(
            project__members__user=request.user
        ),
        id=task_id
    )
    
    new_status = request.data.get('status')
    if new_status not in dict(Task.STATUS_CHOICES):
        return Response(
            {'error': 'Invalid status'}, 
            status=status.HTTP_400_BAD_REQUEST
        )
    
    task.status = new_status
    task.save()
    
    return Response(TaskSerializer(task).data)

@api_view(['GET'])
@permission_classes([IsAuthenticated])
def project_stats(request, project_id):
    """Get project statistics"""
    project = get_object_or_404(
        Project.objects.filter(
            Q(owner=request.user) | 
            Q(members__user=request.user)
        ),
        id=project_id
    )
    
    tasks = Task.objects.filter(project=project)
    
    stats = {
        'total_tasks': tasks.count(),
        'tasks_by_status': dict(tasks.values('status').annotate(count=Count('id')).values_list('status', 'count')),
        'tasks_by_priority': dict(tasks.values('priority').annotate(count=Count('id')).values_list('priority', 'count')),
        'overdue_tasks': tasks.filter(
            due_date__lt=timezone.now(),
            status__in=['todo', 'in_progress', 'review']
        ).count(),
        'total_time_logged': tasks.aggregate(
            total_hours=Sum('actual_hours')
        )['total_hours'] or 0,
        'completion_rate': round(
            (tasks.filter(status='done').count() / tasks.count() * 100) 
            if tasks.count() > 0 else 0, 2
        )
    }
    
    return Response(stats)

class TimeEntryListCreateView(generics.ListCreateAPIView):
    serializer_class = TimeEntrySerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        task_id = self.kwargs['task_id']
        return TimeEntry.objects.filter(
            task_id=task_id,
            task__project__members__user=self.request.user
        ).order_by('-date')

    def perform_create(self, serializer):
        task = get_object_or_404(
            Task.objects.filter(
                project__members__user=self.request.user
            ),
            id=self.kwargs['task_id']
        )
        time_entry = serializer.save(user=self.request.user, task=task)
        
        # Update task actual hours
        total_hours = TimeEntry.objects.filter(task=task).aggregate(
            total=Sum('hours')
        )['total'] or 0
        task.actual_hours = total_hours
        task.save()
        
        return time_entry
```

### Serializers
```python
# serializers.py
from rest_framework import serializers
from django.contrib.auth.models import User
from .models import Project, Task, ProjectMember, TimeEntry, TaskComment

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ['id', 'username', 'first_name', 'last_name', 'email']

class ProjectMemberSerializer(serializers.ModelSerializer):
    user = UserSerializer(read_only=True)
    
    class Meta:
        model = ProjectMember
        fields = ['id', 'user', 'role', 'joined_at']

class ProjectSerializer(serializers.ModelSerializer):
    owner = UserSerializer(read_only=True)
    members = ProjectMemberSerializer(many=True, read_only=True)
    task_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Project
        fields = [
            'id', 'name', 'description', 'status', 'start_date', 
            'end_date', 'budget', 'owner', 'members', 'task_count',
            'created_at', 'updated_at'
        ]
    
    def get_task_count(self, obj):
        return obj.tasks.count()

class TaskSerializer(serializers.ModelSerializer):
    assignee = UserSerializer(read_only=True)
    creator = UserSerializer(read_only=True)
    comments_count = serializers.SerializerMethodField()
    
    class Meta:
        model = Task
        fields = [
            'id', 'title', 'description', 'project', 'assignee', 'creator',
            'priority', 'status', 'due_date', 'estimated_hours', 
            'actual_hours', 'comments_count', 'created_at', 'updated_at'
        ]
    
    def get_comments_count(self, obj):
        return obj.comments.count()

class TaskCreateSerializer(serializers.ModelSerializer):
    class Meta:
        model = Task
        fields = [
            'title', 'description', 'assignee', 'priority', 
            'due_date', 'estimated_hours'
        ]
    
    def validate_assignee(self, value):
        if value:
            project = self.context['view'].get_object()
            if not ProjectMember.objects.filter(
                project=project, user=value
            ).exists():
                raise serializers.ValidationError(
                    "Assignee must be a project member"
                )
        return value

class TimeEntrySerializer(serializers.ModelSerializer):
    user = UserSerializer(read_only=True)
    
    class Meta:
        model = TimeEntry
        fields = [
            'id', 'task', 'user', 'description', 'hours', 
            'date', 'created_at', 'updated_at'
        ]

class TaskCommentSerializer(serializers.ModelSerializer):
    author = UserSerializer(read_only=True)
    
    class Meta:
        model = TaskComment
        fields = ['id', 'content', 'author', 'created_at', 'updated_at']
```

---

## 🧪 Testing Requirements

### Frontend Testing (Vitest + Vue Test Utils)
```typescript
// tests/components/TaskCard.test.ts
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import TaskCard from '@/components/TaskCard.vue';

const mockTask = {
  id: '1',
  title: 'Test Task',
  description: 'Test description',
  status: 'todo',
  priority: 'high',
  assignee: {
    id: '1',
    first_name: 'John',
    last_name: 'Doe',
    avatar: 'avatar.jpg'
  },
  due_date: '2024-12-31',
  comments_count: 3
};

describe('TaskCard', () => {
  it('renders task information correctly', () => {
    const wrapper = mount(TaskCard, {
      props: { task: mockTask }
    });

    expect(wrapper.text()).toContain('Test Task');
    expect(wrapper.text()).toContain('Test description');
    expect(wrapper.text()).toContain('John Doe');
    expect(wrapper.text()).toContain('Comments (3)');
  });

  it('applies correct priority class', () => {
    const wrapper = mount(TaskCard, {
      props: { task: mockTask }
    });

    expect(wrapper.find('.task-priority').classes()).toContain('priority-high');
  });

  it('emits drag-start event when dragged', () => {
    const wrapper = mount(TaskCard, {
      props: { task: mockTask }
    });

    wrapper.trigger('dragstart');
    
    expect(wrapper.emitted('drag-start')).toBeTruthy();
    expect(wrapper.emitted('drag-start')?.[0]).toEqual([mockTask]);
  });

  it('shows overdue class for past due dates', () => {
    const overdueTask = {
      ...mockTask,
      due_date: '2020-01-01'
    };

    const wrapper = mount(TaskCard, {
      props: { task: overdueTask }
    });

    expect(wrapper.find('.due-date').classes()).toContain('overdue');
  });
});
```

### Backend Testing (Django TestCase)
```python
# tests/test_views.py
from django.test import TestCase
from django.contrib.auth.models import User
from rest_framework.test import APIClient
from rest_framework import status
from datetime import datetime, timedelta

from ..models import Project, Task, ProjectMember

class ProjectAPITestCase(TestCase):
    def setUp(self):
        self.client = APIClient()
        self.user = User.objects.create_user(
            username='testuser',
            email='test@example.com',
            password='testpass123'
        )
        self.client.force_authenticate(user=self.user)

    def test_create_project(self):
        data = {
            'name': 'Test Project',
            'description': 'Test description',
            'start_date': '2024-01-01',
            'end_date': '2024-12-31'
        }
        
        response = self.client.post('/api/projects/', data)
        
        self.assertEqual(response.status_code, status.HTTP_201_CREATED)
        self.assertEqual(Project.objects.count(), 1)
        self.assertEqual(Project.objects.first().name, 'Test Project')
        self.assertEqual(Project.objects.first().owner, self.user)

    def test_project_stats(self):
        project = Project.objects.create(
            name='Test Project',
            owner=self.user
        )
        
        # Create tasks
        Task.objects.create(
            title='Task 1',
            project=project,
            creator=self.user,
            status='done'
        )
        Task.objects.create(
            title='Task 2',
            project=project,
            creator=self.user,
            status='todo',
            due_date=datetime.now() - timedelta(days=1)
        )
        
        response = self.client.get(f'/api/projects/{project.id}/stats/')
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        self.assertEqual(response.data['total_tasks'], 2)
        self.assertEqual(response.data['overdue_tasks'], 1)
        self.assertEqual(response.data['completion_rate'], 50.0)

    def test_move_task(self):
        project = Project.objects.create(
            name='Test Project',
            owner=self.user
        )
        
        task = Task.objects.create(
            title='Test Task',
            project=project,
            creator=self.user,
            status='todo'
        )
        
        response = self.client.post(
            f'/api/tasks/{task.id}/move/',
            {'status': 'in_progress'}
        )
        
        self.assertEqual(response.status_code, status.HTTP_200_OK)
        task.refresh_from_db()
        self.assertEqual(task.status, 'in_progress')
```

---

## 🚀 Deployment Requirements

### Vue.js Deployment (Netlify/Vercel)
```json
// package.json
{
  "name": "project-management-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "vue-tsc && vite build",
    "preview": "vite preview",
    "test": "vitest"
  },
  "dependencies": {
    "vue": "^3.3.0",
    "vue-router": "^4.2.0",
    "pinia": "^2.1.0",
    "@vueuse/core": "^10.0.0",
    "axios": "^1.4.0",
    "date-fns": "^2.30.0"
  },
  "devDependencies": {
    "@vitejs/plugin-vue": "^4.2.0",
    "typescript": "^5.0.0",
    "vue-tsc": "^1.6.0",
    "vitest": "^0.32.0",
    "@vue/test-utils": "^2.3.0"
  }
}
```

### Django Deployment (Railway/Heroku)
```python
# settings.py
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent

SECRET_KEY = os.environ.get('SECRET_KEY', 'your-secret-key')

DEBUG = os.environ.get('DEBUG', 'False').lower() == 'true'

ALLOWED_HOSTS = os.environ.get('ALLOWED_HOSTS', '').split(',')

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'corsheaders',
    'projects',
    'tasks',
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

ROOT_URLCONF = 'project_management.urls'

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASSWORD'),
        'HOST': os.environ.get('DB_HOST'),
        'PORT': os.environ.get('DB_PORT'),
    }
}

STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')

MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

CORS_ALLOWED_ORIGINS = [
    "http://localhost:3000",
    "https://yourdomain.com",
]

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 20
}
```

---

## 📚 Learning Resources

### Essential Reading
- [Vue.js Documentation](https://vuejs.org/guide/) - Progressive JavaScript framework
- [Pinia Documentation](https://pinia.vuejs.org/) - State management for Vue
- [Django REST Framework](https://www.django-rest-framework.org/) - API development
- [Django Documentation](https://docs.djangoproject.com/) - Web framework

### Udemy Courses (Focus Sections)
- **Vue.js Complete Course** - Sections 3-8 (Composition API, State management, Testing)
- **Django Complete Course** - Sections 4-8 (REST APIs, Authentication, Deployment)
- **Project Management Tools** - Sections 2-6 (Kanban boards, Time tracking)

### Practice Projects
- Build a simple task management app with Vue.js
- Create a team collaboration tool
- Implement a time tracking system

---

## ✅ Success Criteria

### Week 5 Goals
- [ ] Set up Vue.js frontend with TypeScript and Pinia
- [ ] Set up Django backend with PostgreSQL
- [ ] Implement project and task management
- [ ] Create drag-and-drop task board
- [ ] Build user authentication and permissions

### Week 6 Goals
- [ ] Add time tracking and reporting features
- [ ] Implement file uploads and attachments
- [ ] Create advanced UI components
- [ ] Add comprehensive testing
- [ ] Deploy frontend and backend
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional project management tool
- [ ] Interactive drag-and-drop task board
- [ ] Time tracking and reporting system
- [ ] File upload and attachment handling
- [ ] User roles and permissions
- [ ] 80%+ test coverage on both frontend and backend
- [ ] Deployed applications with live demos

---

## 🔗 Related Projects
- **Previous Project**: [Project 2: Social Media Dashboard](../Full_Stack/Project_2_Social_Media_Dashboard/)
- **Next Project**: [Project 4: Data Visualization Platform](../Full_Stack/Project_4_Data_Visualization_Platform/)
- **Training Plan**: [Full Stack Intensive Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html/)
- **Main README**: [Repository Overview](../../README.html)
