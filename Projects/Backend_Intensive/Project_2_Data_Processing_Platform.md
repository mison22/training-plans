# Project 2: Data Processing Platform
*Backend Intensive Plan - Weeks 3-4*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 3-4)  
**Tech Stack**: Python + FastAPI + Django + PostgreSQL + Redis + Celery  
**Focus**: Data processing, ETL pipelines, and analytics with Python ecosystem  

### Learning Objectives
- Master Python web frameworks (FastAPI and Django)
- Implement data processing pipelines and ETL operations
- Build analytics and reporting systems
- Handle large datasets with pandas and NumPy
- Implement background job processing with Celery

---

## 📊 Database Schema Design

### Data Sources Table (Django Models)
```python
# models/data_sources.py
from django.db import models
from django.contrib.postgres.fields import JSONField
import uuid

class DataSource(models.Model):
    SOURCE_TYPES = [
        ('csv', 'CSV File'),
        ('api', 'API Endpoint'),
        ('database', 'Database'),
        ('webhook', 'Webhook'),
        ('manual', 'Manual Upload'),
    ]
    
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('processing', 'Processing'),
        ('completed', 'Completed'),
        ('failed', 'Failed'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    name = models.CharField(max_length=200, unique=True)
    source_type = models.CharField(max_length=20, choices=SOURCE_TYPES)
    connection_config = JSONField(default=dict)  # Connection details
    processing_config = JSONField(default=dict)  # Processing rules
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    last_sync = models.DateTimeField(null=True, blank=True)
    next_sync = models.DateTimeField(null=True, blank=True)
    sync_frequency = models.DurationField(null=True, blank=True)  # How often to sync
    is_active = models.BooleanField(default=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        db_table = 'data_sources'
        indexes = [
            models.Index(fields=['source_type', 'is_active']),
            models.Index(fields=['status', 'next_sync']),
        ]

class DataImport(models.Model):
    STATUS_CHOICES = [
        ('pending', 'Pending'),
        ('processing', 'Processing'),
        ('validating', 'Validating'),
        ('completed', 'Completed'),
        ('failed', 'Failed'),
    ]
    
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    data_source = models.ForeignKey(DataSource, on_delete=models.CASCADE, related_name='imports')
    file_path = models.CharField(max_length=500, null=True, blank=True)
    file_size = models.BigIntegerField(null=True, blank=True)
    record_count = models.IntegerField(null=True, blank=True)
    processed_count = models.IntegerField(default=0)
    error_count = models.IntegerField(default=0)
    status = models.CharField(max_length=20, choices=STATUS_CHOICES, default='pending')
    error_log = models.TextField(null=True, blank=True)
    metadata = JSONField(default=dict)
    started_at = models.DateTimeField(null=True, blank=True)
    completed_at = models.DateTimeField(null=True, blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'data_imports'
        indexes = [
            models.Index(fields=['data_source', 'status']),
            models.Index(fields=['created_at']),
        ]
```

### Processed Data Tables (FastAPI/SQLAlchemy)
```python
# models/processed_data.py
from sqlalchemy import Column, String, DateTime, Integer, Float, Text, JSON, Index
from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.ext.declarative import declarative_base
import uuid

Base = declarative_base()

class SalesData(Base):
    __tablename__ = 'sales_data'
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    transaction_id = Column(String(100), unique=True, nullable=False)
    date = Column(DateTime, nullable=False)
    customer_id = Column(String(100), nullable=False)
    product_id = Column(String(100), nullable=False)
    product_name = Column(String(200), nullable=False)
    category = Column(String(100), nullable=False)
    quantity = Column(Integer, nullable=False)
    unit_price = Column(Float, nullable=False)
    total_amount = Column(Float, nullable=False)
    discount_amount = Column(Float, default=0.0)
    tax_amount = Column(Float, default=0.0)
    payment_method = Column(String(50))
    region = Column(String(100))
    salesperson_id = Column(String(100))
    raw_data = Column(JSON)  # Store original data for reference
    created_at = Column(DateTime, nullable=False)
    updated_at = Column(DateTime, nullable=False)
    
    __table_args__ = (
        Index('idx_sales_date', 'date'),
        Index('idx_sales_customer', 'customer_id'),
        Index('idx_sales_product', 'product_id'),
        Index('idx_sales_category', 'category'),
        Index('idx_sales_region', 'region'),
    )

class CustomerData(Base):
    __tablename__ = 'customer_data'
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    customer_id = Column(String(100), unique=True, nullable=False)
    first_name = Column(String(100), nullable=False)
    last_name = Column(String(100), nullable=False)
    email = Column(String(200), nullable=False)
    phone = Column(String(50))
    date_of_birth = Column(DateTime)
    gender = Column(String(10))
    address = Column(Text)
    city = Column(String(100))
    state = Column(String(100))
    zip_code = Column(String(20))
    country = Column(String(100))
    registration_date = Column(DateTime)
    last_purchase_date = Column(DateTime)
    total_purchases = Column(Float, default=0.0)
    purchase_count = Column(Integer, default=0)
    customer_segment = Column(String(50))
    raw_data = Column(JSON)
    created_at = Column(DateTime, nullable=False)
    updated_at = Column(DateTime, nullable=False)
    
    __table_args__ = (
        Index('idx_customer_email', 'email'),
        Index('idx_customer_segment', 'customer_segment'),
        Index('idx_customer_region', 'city', 'state'),
    )

class ProductData(Base):
    __tablename__ = 'product_data'
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    product_id = Column(String(100), unique=True, nullable=False)
    name = Column(String(200), nullable=False)
    description = Column(Text)
    category = Column(String(100), nullable=False)
    subcategory = Column(String(100))
    brand = Column(String(100))
    sku = Column(String(100))
    price = Column(Float, nullable=False)
    cost = Column(Float)
    weight = Column(Float)
    dimensions = Column(JSON)  # {length, width, height}
    color = Column(String(50))
    size = Column(String(50))
    material = Column(String(100))
    supplier = Column(String(200))
    stock_quantity = Column(Integer, default=0)
    reorder_level = Column(Integer, default=0)
    is_active = Column(Boolean, default=True)
    launch_date = Column(DateTime)
    raw_data = Column(JSON)
    created_at = Column(DateTime, nullable=False)
    updated_at = Column(DateTime, nullable=False)
    
    __table_args__ = (
        Index('idx_product_category', 'category'),
        Index('idx_product_brand', 'brand'),
        Index('idx_product_supplier', 'supplier'),
        Index('idx_product_active', 'is_active'),
    )
```

### Analytics Tables
```python
# models/analytics.py
class DailySalesMetrics(Base):
    __tablename__ = 'daily_sales_metrics'
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    date = Column(DateTime, nullable=False)
    total_revenue = Column(Float, nullable=False)
    total_orders = Column(Integer, nullable=False)
    total_units_sold = Column(Integer, nullable=False)
    average_order_value = Column(Float, nullable=False)
    unique_customers = Column(Integer, nullable=False)
    new_customers = Column(Integer, nullable=False)
    returning_customers = Column(Integer, nullable=False)
    top_product_id = Column(String(100))
    top_product_revenue = Column(Float)
    region_breakdown = Column(JSON)  # {region: revenue}
    category_breakdown = Column(JSON)  # {category: revenue}
    payment_method_breakdown = Column(JSON)  # {method: count}
    created_at = Column(DateTime, nullable=False)
    
    __table_args__ = (
        Index('idx_daily_metrics_date', 'date'),
        Index('idx_daily_metrics_revenue', 'total_revenue'),
    )

class CustomerSegments(Base):
    __tablename__ = 'customer_segments'
    
    id = Column(UUID(as_uuid=True), primary_key=True, default=uuid.uuid4)
    segment_name = Column(String(100), nullable=False)
    criteria = Column(JSON, nullable=False)  # Segmentation criteria
    customer_count = Column(Integer, nullable=False)
    total_revenue = Column(Float, nullable=False)
    average_order_value = Column(Float, nullable=False)
    average_orders_per_customer = Column(Float, nullable=False)
    last_purchase_avg_days = Column(Float)
    top_categories = Column(JSON)  # [category, revenue]
    customer_ids = Column(JSON)  # List of customer IDs in segment
    created_at = Column(DateTime, nullable=False)
    updated_at = Column(DateTime, nullable=False)
    
    __table_args__ = (
        Index('idx_segments_name', 'segment_name'),
        Index('idx_segments_revenue', 'total_revenue'),
    )
```

---

## 🛠️ FastAPI Application Structure

### Main Application
```python
# main.py
from fastapi import FastAPI, HTTPException, Depends, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
import uvicorn
from contextlib import asynccontextmanager

from app.core.config import settings
from app.core.database import engine, Base
from app.api.v1.api import api_router
from app.core.celery_app import celery_app

# Create database tables
Base.metadata.create_all(bind=engine)

@asynccontextmanager
async def lifespan(app: FastAPI):
    # Startup
    print("Starting up...")
    yield
    # Shutdown
    print("Shutting down...")

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    description="Data Processing Platform API",
    lifespan=lifespan
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.ALLOWED_HOSTS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include API router
app.include_router(api_router, prefix=settings.API_V1_STR)

@app.get("/health")
async def health_check():
    return {
        "status": "healthy",
        "version": settings.VERSION,
        "database": "connected"
    }

if __name__ == "__main__":
    uvicorn.run(
        "main:app",
        host="0.0.0.0",
        port=8000,
        reload=True if settings.ENVIRONMENT == "development" else False
    )
```

### Data Processing Endpoints
```python
# api/v1/endpoints/data_processing.py
from fastapi import APIRouter, Depends, HTTPException, BackgroundTasks, UploadFile, File
from sqlalchemy.orm import Session
from typing import List, Optional
import pandas as pd
import io

from app.core.database import get_db
from app.core.security import get_current_user
from app.models.user import User
from app.schemas.data_processing import (
    DataSourceCreate, DataSourceResponse, DataImportResponse,
    ProcessingJobResponse, AnalyticsResponse
)
from app.services.data_processing import DataProcessingService
from app.tasks.data_processing import process_data_file

router = APIRouter()

@router.post("/data-sources", response_model=DataSourceResponse)
async def create_data_source(
    data_source: DataSourceCreate,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Create a new data source configuration"""
    service = DataProcessingService(db)
    return await service.create_data_source(data_source, current_user.id)

@router.get("/data-sources", response_model=List[DataSourceResponse])
async def list_data_sources(
    skip: int = 0,
    limit: int = 100,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """List all data sources"""
    service = DataProcessingService(db)
    return await service.list_data_sources(skip, limit, current_user.id)

@router.post("/upload-file/{data_source_id}")
async def upload_data_file(
    data_source_id: str,
    background_tasks: BackgroundTasks,
    file: UploadFile = File(...),
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Upload and process a data file"""
    if not file.filename.endswith(('.csv', '.xlsx', '.json')):
        raise HTTPException(status_code=400, detail="Unsupported file format")
    
    # Read file content
    content = await file.read()
    
    # Start background processing
    task = process_data_file.delay(
        data_source_id=data_source_id,
        file_content=content,
        filename=file.filename,
        user_id=current_user.id
    )
    
    return {
        "message": "File uploaded successfully",
        "task_id": task.id,
        "status": "processing"
    }

@router.get("/processing-jobs/{job_id}", response_model=ProcessingJobResponse)
async def get_processing_job(
    job_id: str,
    current_user: User = Depends(get_current_user)
):
    """Get processing job status"""
    from app.core.celery_app import celery_app
    
    task = celery_app.AsyncResult(job_id)
    
    if task.state == 'PENDING':
        response = {
            'job_id': job_id,
            'status': 'pending',
            'progress': 0
        }
    elif task.state == 'PROGRESS':
        response = {
            'job_id': job_id,
            'status': 'processing',
            'progress': task.info.get('progress', 0),
            'current_step': task.info.get('current_step', ''),
            'total_records': task.info.get('total_records', 0),
            'processed_records': task.info.get('processed_records', 0)
        }
    elif task.state == 'SUCCESS':
        response = {
            'job_id': job_id,
            'status': 'completed',
            'progress': 100,
            'result': task.result
        }
    else:  # FAILURE
        response = {
            'job_id': job_id,
            'status': 'failed',
            'error': str(task.info)
        }
    
    return response

@router.get("/analytics/dashboard", response_model=AnalyticsResponse)
async def get_analytics_dashboard(
    start_date: Optional[str] = None,
    end_date: Optional[str] = None,
    current_user: User = Depends(get_current_user),
    db: Session = Depends(get_db)
):
    """Get analytics dashboard data"""
    service = DataProcessingService(db)
    return await service.get_analytics_dashboard(start_date, end_date)
```

---

## 🐍 Celery Background Tasks

### Data Processing Tasks
```python
# tasks/data_processing.py
from celery import Celery
from app.core.celery_app import celery_app
from app.services.data_processing import DataProcessingService
from app.core.database import SessionLocal
import pandas as pd
import io
import json
from datetime import datetime

@celery_app.task(bind=True)
def process_data_file(self, data_source_id: str, file_content: bytes, filename: str, user_id: str):
    """Process uploaded data file in background"""
    db = SessionLocal()
    
    try:
        # Update task progress
        self.update_state(
            state='PROGRESS',
            meta={
                'progress': 10,
                'current_step': 'Reading file',
                'total_records': 0,
                'processed_records': 0
            }
        )
        
        # Initialize service
        service = DataProcessingService(db)
        
        # Read file based on extension
        if filename.endswith('.csv'):
            df = pd.read_csv(io.StringIO(file_content.decode('utf-8')))
        elif filename.endswith('.xlsx'):
            df = pd.read_excel(io.BytesIO(file_content))
        elif filename.endswith('.json'):
            df = pd.read_json(io.StringIO(file_content.decode('utf-8')))
        else:
            raise ValueError("Unsupported file format")
        
        total_records = len(df)
        
        # Update progress
        self.update_state(
            state='PROGRESS',
            meta={
                'progress': 20,
                'current_step': 'Validating data',
                'total_records': total_records,
                'processed_records': 0
            }
        )
        
        # Validate and clean data
        validated_df = service.validate_data(df, data_source_id)
        
        # Update progress
        self.update_state(
            state='PROGRESS',
            meta={
                'progress': 40,
                'current_step': 'Processing records',
                'total_records': total_records,
                'processed_records': 0
            }
        )
        
        # Process data in batches
        batch_size = 1000
        processed_count = 0
        error_count = 0
        
        for i in range(0, len(validated_df), batch_size):
            batch = validated_df.iloc[i:i+batch_size]
            
            try:
                # Process batch
                service.process_data_batch(batch, data_source_id)
                processed_count += len(batch)
                
                # Update progress
                progress = 40 + (processed_count / total_records) * 50
                self.update_state(
                    state='PROGRESS',
                    meta={
                        'progress': int(progress),
                        'current_step': 'Processing records',
                        'total_records': total_records,
                        'processed_records': processed_count
                    }
                )
                
            except Exception as e:
                error_count += len(batch)
                print(f"Error processing batch {i}: {str(e)}")
        
        # Generate analytics
        self.update_state(
            state='PROGRESS',
            meta={
                'progress': 90,
                'current_step': 'Generating analytics',
                'total_records': total_records,
                'processed_records': processed_count
            }
        )
        
        analytics = service.generate_analytics(data_source_id)
        
        # Update progress
        self.update_state(
            state='PROGRESS',
            meta={
                'progress': 100,
                'current_step': 'Completed',
                'total_records': total_records,
                'processed_records': processed_count
            }
        )
        
        return {
            'total_records': total_records,
            'processed_records': processed_count,
            'error_count': error_count,
            'analytics': analytics,
            'completed_at': datetime.utcnow().isoformat()
        }
        
    except Exception as e:
        self.update_state(
            state='FAILURE',
            meta={'error': str(e)}
        )
        raise e
    finally:
        db.close()

@celery_app.task
def generate_daily_analytics():
    """Generate daily analytics (scheduled task)"""
    db = SessionLocal()
    
    try:
        service = DataProcessingService(db)
        service.generate_daily_analytics()
        
        return {
            'status': 'success',
            'message': 'Daily analytics generated',
            'timestamp': datetime.utcnow().isoformat()
        }
    except Exception as e:
        return {
            'status': 'error',
            'message': str(e),
            'timestamp': datetime.utcnow().isoformat()
        }
    finally:
        db.close()

@celery_app.task
def cleanup_old_data():
    """Clean up old data files and temporary data"""
    db = SessionLocal()
    
    try:
        service = DataProcessingService(db)
        cleaned_count = service.cleanup_old_data()
        
        return {
            'status': 'success',
            'message': f'Cleaned up {cleaned_count} old records',
            'timestamp': datetime.utcnow().isoformat()
        }
    except Exception as e:
        return {
            'status': 'error',
            'message': str(e),
            'timestamp': datetime.utcnow().isoformat()
        }
    finally:
        db.close()
```

---

## 🧪 Testing Requirements

### Pytest Configuration
```python
# pytest.ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = 
    --verbose
    --tb=short
    --strict-markers
    --disable-warnings
    --cov=app
    --cov-report=html
    --cov-report=term-missing
markers =
    unit: Unit tests
    integration: Integration tests
    slow: Slow running tests
```

### Test Examples
```python
# tests/test_data_processing.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
import pandas as pd

from app.main import app
from app.core.database import get_db, Base
from app.models.data_processing import DataSource

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False})
TestingSessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

def override_get_db():
    try:
        db = TestingSessionLocal()
        yield db
    finally:
        db.close()

app.dependency_overrides[get_db] = override_get_db

@pytest.fixture
def client():
    Base.metadata.create_all(bind=engine)
    with TestClient(app) as test_client:
        yield test_client
    Base.metadata.drop_all(bind=engine)

@pytest.fixture
def sample_csv_data():
    return """transaction_id,date,customer_id,product_id,quantity,unit_price
TXN001,2024-01-01,CUST001,PROD001,2,25.50
TXN002,2024-01-01,CUST002,PROD002,1,45.00
TXN003,2024-01-02,CUST001,PROD003,3,15.75"""

def test_upload_and_process_csv(client, sample_csv_data):
    """Test CSV file upload and processing"""
    # Upload file
    response = client.post(
        "/api/v1/data-processing/upload-file/test-source-id",
        files={"file": ("test.csv", sample_csv_data, "text/csv")}
    )
    
    assert response.status_code == 200
    data = response.json()
    assert "task_id" in data
    assert data["status"] == "processing"

def test_analytics_dashboard(client):
    """Test analytics dashboard endpoint"""
    response = client.get("/api/v1/data-processing/analytics/dashboard")
    
    assert response.status_code == 200
    data = response.json()
    assert "summary" in data
    assert "metrics" in data
    assert "charts" in data

# tests/test_celery_tasks.py
import pytest
from unittest.mock import patch, MagicMock
from app.tasks.data_processing import process_data_file

@patch('app.tasks.data_processing.DataProcessingService')
def test_process_data_file_task(mock_service_class):
    """Test data processing Celery task"""
    # Mock service
    mock_service = MagicMock()
    mock_service_class.return_value = mock_service
    
    # Mock pandas read_csv
    with patch('pandas.read_csv') as mock_read_csv:
        mock_df = pd.DataFrame({
            'transaction_id': ['TXN001', 'TXN002'],
            'date': ['2024-01-01', '2024-01-01'],
            'customer_id': ['CUST001', 'CUST002'],
            'product_id': ['PROD001', 'PROD002'],
            'quantity': [2, 1],
            'unit_price': [25.50, 45.00]
        })
        mock_read_csv.return_value = mock_df
        
        # Mock validate_data and process_data_batch
        mock_service.validate_data.return_value = mock_df
        mock_service.process_data_batch.return_value = None
        mock_service.generate_analytics.return_value = {"total_revenue": 96.0}
        
        # Execute task
        result = process_data_file(
            data_source_id="test-source",
            file_content=b"test,data\n1,2",
            filename="test.csv",
            user_id="test-user"
        )
        
        # Assertions
        assert result["total_records"] == 2
        assert result["processed_records"] == 2
        assert result["error_count"] == 0
        assert "analytics" in result
```

---

## 🚀 Deployment Requirements

### Docker Compose Configuration
```yaml
# docker-compose.yml
version: '3.8'

services:
  # FastAPI Application
  fastapi-app:
    build: ./fastapi-app
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/data_platform
      - REDIS_URL=redis://redis:6379
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - ./data:/app/data
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload

  # Django Admin Interface
  django-admin:
    build: ./django-admin
    ports:
      - "8001:8001"
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/data_platform
    depends_on:
      - postgres
    command: python manage.py runserver 0.0.0.0:8001

  # Celery Worker
  celery-worker:
    build: ./fastapi-app
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/data_platform
      - REDIS_URL=redis://redis:6379
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    volumes:
      - ./data:/app/data
    command: celery -A app.core.celery_app worker --loglevel=info

  # Celery Beat (Scheduler)
  celery-beat:
    build: ./fastapi-app
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/data_platform
      - REDIS_URL=redis://redis:6379
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis
    command: celery -A app.core.celery_app beat --loglevel=info

  # PostgreSQL Database
  postgres:
    image: postgres:15
    environment:
      - POSTGRES_DB=data_platform
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # Flower (Celery Monitoring)
  flower:
    build: ./fastapi-app
    ports:
      - "5555:5555"
    environment:
      - REDIS_URL=redis://redis:6379
      - CELERY_BROKER_URL=redis://redis:6379
    depends_on:
      - redis
    command: celery -A app.core.celery_app flower --port=5555

volumes:
  postgres_data:
  redis_data:
```

### Environment Configuration
```bash
# .env
# Database
DATABASE_URL=postgresql://user:password@localhost:5432/data_platform

# Redis
REDIS_URL=redis://localhost:6379

# Celery
CELERY_BROKER_URL=redis://localhost:6379
CELERY_RESULT_BACKEND=redis://localhost:6379

# Application
ENVIRONMENT=development
DEBUG=true
SECRET_KEY=your-secret-key-here

# File Storage
UPLOAD_DIR=/app/data/uploads
MAX_FILE_SIZE=100MB

# Email (for notifications)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
```

---

## 📚 Learning Resources

### Essential Reading
- [FastAPI Documentation](https://fastapi.tiangolo.com/) - Modern Python web framework
- [Django Documentation](https://docs.djangoproject.com/) - Full-featured Python framework
- [Pandas Documentation](https://pandas.pydata.org/docs/) - Data manipulation
- [Celery Documentation](https://docs.celeryproject.org/) - Distributed task queue

### Udemy Courses (Focus Sections)
- **FastAPI Complete Course** - Sections 2-6 (Advanced features, Background tasks)
- **Django Complete Course** - Sections 3-7 (Models, Admin, REST APIs)
- **Python Data Science** - Sections 4-8 (Pandas, NumPy, Data processing)

### Practice Projects
- Build a simple data processing pipeline with Python
- Create a basic analytics dashboard with FastAPI
- Implement file upload and processing system

---

## ✅ Success Criteria

### Week 3 Goals
- [ ] Set up FastAPI and Django applications
- [ ] Implement data models and database schemas
- [ ] Create file upload and processing endpoints
- [ ] Set up Celery for background tasks
- [ ] Build basic data validation and cleaning

### Week 4 Goals
- [ ] Implement comprehensive data processing pipelines
- [ ] Create analytics and reporting features
- [ ] Add data visualization capabilities
- [ ] Implement scheduled tasks and monitoring
- [ ] Deploy with Docker Compose
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional data processing platform
- [ ] Comprehensive API documentation
- [ ] Background task processing with Celery
- [ ] Analytics dashboard with visualizations
- [ ] 80%+ test coverage with pytest
- [ ] Docker Compose deployment configuration
- [ ] Performance monitoring and logging

---

## 🔗 Related Projects
- **Previous Project**: [Project 1: E-commerce API Platform](../Backend_Intensive/Project_1_E_Commerce_API/)
- **Next Project**: [Project 3: Real-time Application](../Backend_Intensive/Project_3_Real_Time_Application/)
- **Training Plan**: [Backend Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)
- **Main README**: [Repository Overview](../../README.html)
