# Project 3: Rails Microservices Platform
*Service-oriented architecture with message queues and distributed systems*

## Project Overview

Build a microservices platform using Rails, demonstrating service decomposition, inter-service communication, message queues, and distributed data management. This project showcases expertise in building scalable, maintainable distributed systems.

## Technology Stack

- **Framework**: Rails 7.1+ (multiple services)
- **Language**: Ruby 3.2+
- **Databases**: PostgreSQL (per service) + Redis
- **Message Queue**: RabbitMQ
- **Service Communication**: REST + gRPC
- **API Gateway**: Kong or custom Rails gateway
- **Service Discovery**: Consul (optional)
- **Testing**: RSpec + VCR
- **Containerization**: Docker + Docker Compose
- **Orchestration**: Kubernetes
- **Monitoring**: Prometheus + Grafana
- **Deployment**: AWS ECS/EKS

## Microservices Architecture

### Services Overview

1. **API Gateway Service**
   - Request routing
   - Authentication/Authorization
   - Rate limiting
   - Request aggregation

2. **Auth Service**
   - User authentication
   - JWT token generation
   - OAuth integration
   - Session management

3. **User Service**
   - User profile management
   - User preferences
   - User search

4. **Product Service**
   - Product catalog
   - Inventory management
   - Product search

5. **Order Service**
   - Order creation
   - Order processing
   - Order history

6. **Notification Service**
   - Email notifications
   - Push notifications
   - SMS notifications
   - Notification preferences

## Core Features

### 1. Service Communication
- RESTful APIs between services
- gRPC for high-performance calls
- Message queue for async operations
- Event-driven architecture

### 2. Data Management
- Database per service pattern
- Saga pattern for distributed transactions
- Event sourcing for critical operations
- CQRS for read/write separation

### 3. Authentication & Authorization
- Centralized auth service
- JWT token propagation
- Service-to-service auth
- Role-based access control

### 4. Resilience & Reliability
- Circuit breaker pattern
- Retry mechanisms
- Fallback strategies
- Health checks

### 5. Monitoring & Observability
- Distributed tracing
- Centralized logging
- Metrics collection
- Alert management

## Implementation Guide

### Week 5: Service Setup & Communication

#### Day 1-2: Project Structure
```bash
# Create microservices directory structure
mkdir rails_microservices
cd rails_microservices

# Create services
rails new auth_service --api --database=postgresql
rails new user_service --api --database=postgresql
rails new product_service --api --database=postgresql
rails new order_service --api --database=postgresql
rails new notification_service --api --database=postgresql
rails new api_gateway --api --database=postgresql

# Create docker-compose.yml
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  postgres:
    image: postgres:15
    environment:
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
  
  auth_service:
    build: ./auth_service
    ports:
      - "3001:3000"
    depends_on:
      - postgres
      - redis
  
  user_service:
    build: ./user_service
    ports:
      - "3002:3000"
    depends_on:
      - postgres
  
  product_service:
    build: ./product_service
    ports:
      - "3003:3000"
    depends_on:
      - postgres
  
  order_service:
    build: ./order_service
    ports:
      - "3004:3000"
    depends_on:
      - postgres
      - rabbitmq
  
  notification_service:
    build: ./notification_service
    ports:
      - "3005:3000"
    depends_on:
      - rabbitmq
  
  api_gateway:
    build: ./api_gateway
    ports:
      - "3000:3000"
    depends_on:
      - auth_service
      - user_service
      - product_service
      - order_service
```

#### Day 3-5: Auth Service
```ruby
# auth_service/app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  
  validates :email, presence: true, uniqueness: true
  validates :password, length: { minimum: 6 }, if: -> { password.present? }
  
  def generate_jwt
    JWT.encode(
      {
        user_id: id,
        email: email,
        exp: 24.hours.from_now.to_i
      },
      Rails.application.credentials.secret_key_base
    )
  end
end

# auth_service/app/controllers/api/v1/auth_controller.rb
class Api::V1::AuthController < ApplicationController
  def register
    user = User.new(user_params)
    
    if user.save
      token = user.generate_jwt
      
      # Publish event to message queue
      PublishEvent.call('user.registered', {
        user_id: user.id,
        email: user.email
      })
      
      render json: { token: token, user: user }, status: :created
    else
      render json: { errors: user.errors.full_messages }, status: :unprocessable_entity
    end
  end
  
  def login
    user = User.find_by(email: params[:email])
    
    if user&.authenticate(params[:password])
      token = user.generate_jwt
      render json: { token: token }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
  
  def validate_token
    # Used by other services to validate tokens
    token = request.headers['Authorization']&.split(' ')&.last
    
    begin
      payload = JWT.decode(token, Rails.application.credentials.secret_key_base)[0]
      render json: { valid: true, user_id: payload['user_id'] }
    rescue JWT::DecodeError
      render json: { valid: false }, status: :unauthorized
    end
  end
  
  private
  
  def user_params
    params.require(:user).permit(:email, :password, :password_confirmation)
  end
end
```

#### Day 6-7: API Gateway
```ruby
# api_gateway/app/controllers/gateway_controller.rb
class GatewayController < ApplicationController
  before_action :authenticate_request
  
  def route_request
    service = determine_service(request.path)
    method = request.method.downcase
    
    response = ServiceClient.call(
      service: service,
      method: method,
      path: request.path,
      body: request.body.read,
      headers: forward_headers
    )
    
    render json: response.body, status: response.code
  rescue => e
    render json: { error: e.message }, status: :service_unavailable
  end
  
  private
  
  def authenticate_request
    token = request.headers['Authorization']
    
    # Call auth service to validate token
    response = HTTP.get(
      "#{ENV['AUTH_SERVICE_URL']}/api/v1/auth/validate",
      headers: { 'Authorization' => token }
    )
    
    unless response.status.success?
      render json: { error: 'Unauthorized' }, status: :unauthorized
    end
  end
  
  def determine_service(path)
    case path
    when %r{^/api/v1/users}
      ENV['USER_SERVICE_URL']
    when %r{^/api/v1/products}
      ENV['PRODUCT_SERVICE_URL']
    when %r{^/api/v1/orders}
      ENV['ORDER_SERVICE_URL']
    else
      raise "Unknown service for path: #{path}"
    end
  end
  
  def forward_headers
    {
      'Authorization' => request.headers['Authorization'],
      'Content-Type' => 'application/json'
    }
  end
end

# api_gateway/app/services/service_client.rb
class ServiceClient
  include CircuitBreaker
  
  def self.call(service:, method:, path:, body: nil, headers: {})
    with_circuit_breaker(service) do
      HTTP
        .headers(headers)
        .timeout(connect: 2, read: 5)
        .request(method, "#{service}#{path}", body: body)
    end
  end
  
  def self.with_circuit_breaker(service_name)
    circuit = CircuitBreaker.get(service_name)
    
    if circuit.open?
      raise CircuitOpenError, "Circuit breaker open for #{service_name}"
    end
    
    begin
      response = yield
      circuit.record_success
      response
    rescue => e
      circuit.record_failure
      raise e
    end
  end
end
```

### Week 6: Message Queue & Event-Driven Architecture

#### Day 8-10: RabbitMQ Integration
```ruby
# lib/message_queue/publisher.rb
module MessageQueue
  class Publisher
    def self.publish(exchange_name, routing_key, message)
      connection = Bunny.new(ENV['RABBITMQ_URL'] || 'amqp://localhost')
      connection.start
      
      channel = connection.create_channel
      exchange = channel.topic(exchange_name, durable: true)
      
      exchange.publish(
        message.to_json,
        routing_key: routing_key,
        persistent: true
      )
      
      connection.close
    end
  end
end

# lib/message_queue/consumer.rb
module MessageQueue
  class Consumer
    def initialize(queue_name, exchange_name, routing_key)
      @queue_name = queue_name
      @exchange_name = exchange_name
      @routing_key = routing_key
    end
    
    def start
      connection = Bunny.new(ENV['RABBITMQ_URL'] || 'amqp://localhost')
      connection.start
      
      channel = connection.create_channel
      exchange = channel.topic(@exchange_name, durable: true)
      queue = channel.queue(@queue_name, durable: true)
      queue.bind(exchange, routing_key: @routing_key)
      
      queue.subscribe(block: true) do |delivery_info, properties, body|
        message = JSON.parse(body)
        process_message(message)
      end
    end
    
    def process_message(message)
      raise NotImplementedError, 'Subclasses must implement process_message'
    end
  end
end

# order_service/app/consumers/user_registered_consumer.rb
class UserRegisteredConsumer < MessageQueue::Consumer
  def initialize
    super('order_service_user_registered', 'user_events', 'user.registered')
  end
  
  def process_message(message)
    # Create user record in order service
    OrderServiceUser.create!(
      external_user_id: message['user_id'],
      email: message['email']
    )
    
    Rails.logger.info "User #{message['user_id']} registered in order service"
  end
end
```

#### Day 11-12: Saga Pattern for Distributed Transactions
```ruby
# order_service/app/services/create_order_saga.rb
class CreateOrderSaga
  def initialize(order_params)
    @order_params = order_params
    @saga_state = {}
  end
  
  def execute
    begin
      # Step 1: Reserve inventory
      reserve_inventory
      
      # Step 2: Process payment
      process_payment
      
      # Step 3: Create order
      create_order
      
      # Step 4: Send notification
      send_notification
      
      @saga_state[:order]
    rescue => e
      compensate
      raise e
    end
  end
  
  private
  
  def reserve_inventory
    response = HTTP.post(
      "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reserve",
      json: { product_id: @order_params[:product_id], quantity: @order_params[:quantity] }
    )
    
    if response.status.success?
      @saga_state[:reservation_id] = JSON.parse(response.body)['reservation_id']
    else
      raise "Failed to reserve inventory"
    end
  end
  
  def process_payment
    response = HTTP.post(
      "#{ENV['PAYMENT_SERVICE_URL']}/api/v1/payments",
      json: { amount: @order_params[:amount], user_id: @order_params[:user_id] }
    )
    
    if response.status.success?
      @saga_state[:payment_id] = JSON.parse(response.body)['payment_id']
    else
      raise "Payment processing failed"
    end
  end
  
  def create_order
    order = Order.create!(@order_params.merge(
      reservation_id: @saga_state[:reservation_id],
      payment_id: @saga_state[:payment_id]
    ))
    @saga_state[:order] = order
  end
  
  def send_notification
    MessageQueue::Publisher.publish(
      'order_events',
      'order.created',
      {
        order_id: @saga_state[:order].id,
        user_id: @order_params[:user_id]
      }
    )
  end
  
  def compensate
    # Rollback in reverse order
    rollback_notification if @saga_state[:order]
    rollback_order if @saga_state[:order]
    rollback_payment if @saga_state[:payment_id]
    rollback_inventory if @saga_state[:reservation_id]
  end
  
  def rollback_inventory
    HTTP.delete(
      "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reservations/#{@saga_state[:reservation_id]}"
    )
  end
  
  def rollback_payment
    HTTP.post(
      "#{ENV['PAYMENT_SERVICE_URL']}/api/v1/payments/#{@saga_state[:payment_id]}/refund"
    )
  end
  
  def rollback_order
    @saga_state[:order].update!(status: 'cancelled')
  end
  
  def rollback_notification
    MessageQueue::Publisher.publish(
      'order_events',
      'order.cancelled',
      { order_id: @saga_state[:order].id }
    )
  end
end
```

#### Day 13-14: Testing & Monitoring
```ruby
# spec/services/create_order_saga_spec.rb
require 'rails_helper'

RSpec.describe CreateOrderSaga do
  let(:order_params) do
    {
      user_id: 1,
      product_id: 1,
      quantity: 2,
      amount: 100.00
    }
  end
  
  describe '#execute' do
    context 'when all steps succeed' do
      before do
        stub_request(:post, "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reserve")
          .to_return(status: 200, body: { reservation_id: 'res123' }.to_json)
        
        stub_request(:post, "#{ENV['PAYMENT_SERVICE_URL']}/api/v1/payments")
          .to_return(status: 200, body: { payment_id: 'pay123' }.to_json)
      end
      
      it 'creates an order successfully' do
        saga = CreateOrderSaga.new(order_params)
        order = saga.execute
        
        expect(order).to be_persisted
        expect(order.reservation_id).to eq('res123')
        expect(order.payment_id).to eq('pay123')
      end
    end
    
    context 'when payment fails' do
      before do
        stub_request(:post, "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reserve")
          .to_return(status: 200, body: { reservation_id: 'res123' }.to_json)
        
        stub_request(:post, "#{ENV['PAYMENT_SERVICE_URL']}/api/v1/payments")
          .to_return(status: 400, body: { error: 'Payment failed' }.to_json)
        
        stub_request(:delete, "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reservations/res123")
          .to_return(status: 200)
      end
      
      it 'compensates by releasing reservation' do
        saga = CreateOrderSaga.new(order_params)
        
        expect { saga.execute }.to raise_error('Payment processing failed')
        
        # Verify compensation was called
        expect(WebMock).to have_requested(:delete, 
          "#{ENV['PRODUCT_SERVICE_URL']}/api/v1/inventory/reservations/res123")
      end
    end
  end
end
```

## Monitoring & Observability

```ruby
# config/initializers/prometheus.rb
require 'prometheus/client'

prometheus = Prometheus::Client.registry

# Define custom metrics
HTTP_REQUESTS = prometheus.counter(
  :http_requests_total,
  docstring: 'Total HTTP requests',
  labels: [:service, :method, :path, :status]
)

HTTP_DURATION = prometheus.histogram(
  :http_request_duration_seconds,
  docstring: 'HTTP request duration',
  labels: [:service, :method, :path]
)

# app/middleware/prometheus_middleware.rb
class PrometheusMiddleware
  def initialize(app)
    @app = app
  end
  
  def call(env)
    start_time = Time.current
    
    status, headers, response = @app.call(env)
    
    duration = Time.current - start_time
    
    HTTP_REQUESTS.increment(
      labels: {
        service: ENV['SERVICE_NAME'],
        method: env['REQUEST_METHOD'],
        path: env['PATH_INFO'],
        status: status
      }
    )
    
    HTTP_DURATION.observe(
      duration,
      labels: {
        service: ENV['SERVICE_NAME'],
        method: env['REQUEST_METHOD'],
        path: env['PATH_INFO']
      }
    )
    
    [status, headers, response]
  end
end
```

## Deployment

```yaml
# kubernetes/auth-service-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: auth-service
  template:
    metadata:
      labels:
        app: auth-service
    spec:
      containers:
      - name: auth-service
        image: auth-service:latest
        ports:
        - containerPort: 3000
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: auth-service-secrets
              key: database-url
        - name: REDIS_URL
          value: redis://redis:6379
        livenessProbe:
          httpGet:
            path: /health
            port: 3000
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 3000
          initialDelaySeconds: 5
          periodSeconds: 5
---
apiVersion: v1
kind: Service
metadata:
  name: auth-service
spec:
  selector:
    app: auth-service
  ports:
  - protocol: TCP
    port: 80
    targetPort: 3000
```

## Success Criteria

- [ ] 6 microservices running independently
- [ ] Message queue integration working
- [ ] Saga pattern implemented for distributed transactions
- [ ] API gateway routing requests correctly
- [ ] 85%+ test coverage across all services
- [ ] Monitoring and observability in place
- [ ] Kubernetes deployment successful
- [ ] Comprehensive documentation

---

📋 **[Back to Projects Overview](../Projects/Ruby_Rails_Backend/)** | 🏠 **[Back to Training Plan](../Intensive_Plans/Training_Plan_Ruby_Rails_Backend_Intensive.html)** | 🏠 **[Back to README](../README.html)**

### Related Projects
- **[Project 1: E-commerce Platform](./Project_1_Rails_E_Commerce.html)** - Full Rails application with Stripe
- **[Project 2: Social Media API](./Project_2_Social_Media_API.html)** - Rails API with real-time features
- **[Project 4: Production System](./Project_4_Production_System.html)** - Full DevOps pipeline

