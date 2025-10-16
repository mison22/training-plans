# Ruby on Rails Backend Projects
*Production-ready Rails applications for your portfolio*

## Project Overview

This directory contains 4 comprehensive Ruby on Rails backend projects designed to showcase your expertise in modern Rails development. Each project focuses on different aspects of Rails backend development, from traditional monolithic applications to microservices and production deployment.

## Projects

### 1. Rails E-commerce Platform
- **Framework**: Ruby on Rails + PostgreSQL
- **Features**: User auth (Devise), product catalog, shopping cart, payments (Stripe), admin panel
- **Timeline**: 2 weeks
- **Focus**: Traditional Rails MVC, authentication/authorization, payment integration
- 📝 **[Detailed Requirements](./Project_1_Rails_E_Commerce.html)**

### 2. Social Media API
- **Framework**: Rails API + PostgreSQL + Redis
- **Features**: User profiles, posts, real-time notifications (Action Cable), image uploads
- **Timeline**: 2 weeks
- **Focus**: RESTful API design, real-time features, JWT authentication
- 📝 **[Detailed Requirements](./Project_2_Social_Media_API.html)**

### 3. Rails Microservices Platform
- **Framework**: Rails + RabbitMQ + PostgreSQL + Redis
- **Features**: Service communication, authentication service, data consistency
- **Timeline**: 2 weeks
- **Focus**: Microservices architecture, message queues, distributed systems
- 📝 **[Detailed Requirements](./Project_3_Microservices_Platform.html)**

### 4. Rails Production System
- **Framework**: Rails + Docker + Kubernetes + AWS
- **Features**: Full DevOps pipeline, monitoring, scaling, CI/CD
- **Timeline**: 2 weeks
- **Focus**: Production deployment, infrastructure as code, monitoring
- 📝 **[Detailed Requirements](./Project_4_Production_System.html)**

## Getting Started

1. **Choose a project** based on your learning goals
2. **Read the detailed requirements** for your chosen project
3. **Set up your development environment** with the required tools
4. **Follow the implementation guide** step by step
5. **Deploy to production** and document your work

## Technology Stack

### Core Framework
- **Ruby on Rails**: Full-stack web application framework
- **Ruby**: Dynamic, object-oriented programming language
- **Active Record**: Rails ORM for database management

### Databases
- **PostgreSQL**: Primary relational database
- **Redis**: Caching and background job processing
- **MongoDB**: NoSQL option for specific use cases

### Authentication & Authorization
- **Devise**: Flexible authentication solution
- **Pundit**: Authorization library for policies
- **JWT**: Token-based authentication for APIs

### Testing
- **RSpec**: Behavior-driven development framework
- **FactoryBot**: Test data generation
- **Capybara**: Integration testing
- **SimpleCov**: Code coverage analysis

### Background Processing
- **Sidekiq**: Background job processing
- **Active Job**: Rails abstraction for job queues
- **Redis**: Job queue storage

### Real-time Features
- **Action Cable**: WebSocket integration
- **Redis**: Pub/sub for real-time features

### Payment Processing
- **Stripe**: Payment gateway integration
- **Payment Intent API**: Secure payment processing

### Admin Tools
- **ActiveAdmin**: Admin dashboard framework
- **Rails Admin**: Alternative admin interface

### Deployment & DevOps
- **Docker**: Containerization
- **Kubernetes**: Container orchestration
- **Heroku**: Platform as a Service
- **AWS**: Cloud infrastructure (EC2, RDS, S3, EKS)

### Monitoring & Observability
- **Prometheus**: Metrics collection
- **Grafana**: Metrics visualization
- **New Relic/Datadog**: Application performance monitoring
- **Sentry/Rollbar**: Error tracking
- **ELK Stack**: Centralized logging

## Success Criteria

Each project should demonstrate:
- [ ] Clean, well-documented code following Rails conventions
- [ ] Comprehensive test coverage (90%+)
- [ ] Production-ready deployment
- [ ] API documentation (for API projects)
- [ ] Security best practices
- [ ] Performance optimization
- [ ] Scalability considerations

## Portfolio Tips

1. **Document everything** - README files, API docs, deployment guides
2. **Show your process** - Git commits, design decisions, challenges overcome
3. **Deploy live demos** - Make your projects accessible online
4. **Write about your experience** - Blog posts, technical articles
5. **Contribute to open source** - Use your Rails knowledge to contribute

## Rails Best Practices

### Code Organization
- Follow Rails conventions and structure
- Use concerns for shared behavior
- Keep controllers thin, models fat
- Service objects for complex business logic

### Testing
- Write tests before implementation (TDD)
- Test models, controllers, and integrations
- Use factories instead of fixtures
- Maintain high test coverage

### Performance
- N+1 query prevention with `includes`
- Database indexing on foreign keys
- Fragment caching for views
- Russian doll caching strategy
- Background jobs for heavy operations

### Security
- Use strong parameters
- CSRF protection (enabled by default)
- SQL injection prevention (use Active Record)
- XSS prevention (escape user input)
- Secure password storage (bcrypt)

## Learning Resources

### Official Documentation
- [Rails Guides](https://guides.rubyonrails.org/) - Comprehensive Rails documentation
- [Ruby Documentation](https://ruby-doc.org/) - Ruby language reference
- [API Documentation](https://api.rubyonrails.org/) - Rails API reference

### Community
- [Ruby on Rails Subreddit](https://www.reddit.com/r/rails/)
- [Rails Forum](https://discuss.rubyonrails.org/)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/ruby-on-rails)

### Screencasts & Tutorials
- [GoRails](https://gorails.com/) - Rails video tutorials
- [Drifting Ruby](https://www.driftingruby.com/) - Weekly Rails screencasts
- [RailsCasts](http://railscasts.com/) - Classic Rails tutorials

### Books
- [The Rails Tutorial](https://www.railstutorial.org/) - Michael Hartl
- [Agile Web Development with Rails](https://pragprog.com/titles/rails7/) - Sam Ruby
- [The Ruby on Rails Performance Apocrypha](https://www.speedshop.co/2021/01/14/announcing-apocrypha.html)

## Common Patterns

### Service Objects
```ruby
# app/services/create_order_service.rb
class CreateOrderService
  def initialize(user, cart)
    @user = user
    @cart = cart
  end
  
  def call
    Order.transaction do
      order = create_order
      process_payment(order)
      send_confirmation(order)
      order
    end
  rescue => e
    handle_error(e)
  end
  
  private
  
  def create_order
    # Order creation logic
  end
end
```

### Form Objects
```ruby
# app/forms/registration_form.rb
class RegistrationForm
  include ActiveModel::Model
  
  attr_accessor :email, :password, :terms_accepted
  
  validates :email, :password, presence: true
  validates :terms_accepted, acceptance: true
  
  def save
    return false unless valid?
    
    User.create(email: email, password: password)
  end
end
```

### Decorators
```ruby
# app/decorators/user_decorator.rb
class UserDecorator < SimpleDelegator
  def full_name_with_email
    "#{full_name} (#{email})"
  end
  
  def formatted_created_at
    created_at.strftime("%B %d, %Y")
  end
end
```

---

**Ready to build amazing Rails applications? Pick a project and start coding!**

📋 **[Back to Main Projects](../README.html)** | 🏠 **[Back to README](../../README.html)**

