# Project 1: E-commerce API Platform
*Backend Intensive Plan - Weeks 1-2*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 1-2)  
**Tech Stack**: Ruby on Rails + PostgreSQL + Redis  
**Focus**: Rails API development, MVC architecture, and rapid prototyping  

### Learning Objectives
- Master Ruby on Rails framework and conventions
- Implement JWT authentication with Rails
- Design and implement database relationships with ActiveRecord
- Build comprehensive RESTful APIs with Rails
- Implement caching strategies with Redis

---

## 📊 Database Schema Design (Rails Migrations)

### Users Migration
```ruby
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :email, null: false, index: { unique: true }
      t.string :password_digest, null: false
      t.string :first_name, null: false
      t.string :last_name, null: false
      t.string :phone_number
      t.date :date_of_birth
      t.text :bio
      t.string :avatar_url
      t.integer :role, default: 0, null: false
      t.boolean :email_verified, default: false
      t.boolean :is_active, default: true
      t.datetime :last_login_at
      t.string :reset_password_token
      t.datetime :reset_password_sent_at
      t.string :confirmation_token
      t.datetime :confirmed_at
      t.datetime :confirmation_sent_at

      t.timestamps
    end

    add_index :users, :reset_password_token, unique: true
    add_index :users, :confirmation_token, unique: true
    add_index :users, [:email, :role]
  end
end
```

### Categories Migration
```ruby
class CreateCategories < ActiveRecord::Migration[7.0]
  def change
    create_table :categories do |t|
      t.string :name, null: false
      t.string :slug, null: false, index: { unique: true }
      t.text :description
      t.string :image_url
      t.references :parent, null: true, foreign_key: { to_table: :categories }
      t.integer :sort_order, default: 0
      t.boolean :is_active, default: true
      t.string :meta_title
      t.string :meta_description
      t.json :custom_fields

      t.timestamps
    end

    add_index :categories, [:parent_id, :sort_order]
    add_index :categories, :is_active
  end
end
```

### Products Migration
```ruby
class CreateProducts < ActiveRecord::Migration[7.0]
  def change
    create_table :products do |t|
      t.string :name, null: false
      t.string :slug, null: false, index: { unique: true }
      t.text :description
      t.text :short_description
      t.string :sku, null: false, index: { unique: true }
      t.decimal :price, precision: 10, scale: 2, null: false
      t.decimal :compare_price, precision: 10, scale: 2
      t.decimal :cost_price, precision: 10, scale: 2
      t.integer :stock_quantity, default: 0
      t.integer :low_stock_threshold, default: 5
      t.decimal :weight, precision: 8, scale: 2
      t.json :dimensions # {length: 10, width: 5, height: 3}
      t.references :category, null: false, foreign_key: true
      t.references :vendor, null: false, foreign_key: { to_table: :users }
      t.boolean :is_active, default: true
      t.boolean :is_featured, default: false
      t.string :meta_title
      t.string :meta_description
      t.text :tags, array: true, default: []
      t.integer :views_count, default: 0
      t.decimal :average_rating, precision: 3, scale: 2, default: 0
      t.integer :reviews_count, default: 0

      t.timestamps
    end

    add_index :products, [:category_id, :is_active]
    add_index :products, [:vendor_id, :is_active]
    add_index :products, [:price, :is_active]
    add_index :products, [:is_featured, :is_active]
    add_index :products, [:average_rating, :reviews_count]
    add_index :products, :tags, using: 'gin'
  end
end
```

### Product Images Migration
```ruby
class CreateProductImages < ActiveRecord::Migration[7.0]
  def change
    create_table :product_images do |t|
      t.references :product, null: false, foreign_key: true, on_delete: :cascade
      t.string :image_url, null: false
      t.string :alt_text
      t.integer :sort_order, default: 0
      t.boolean :is_primary, default: false
      t.integer :width
      t.integer :height
      t.string :file_size
      t.string :mime_type

      t.timestamps
    end

    add_index :product_images, [:product_id, :sort_order]
    add_index :product_images, [:product_id, :is_primary]
  end
end
```

### Shopping Cart Migration
```ruby
class CreateCartItems < ActiveRecord::Migration[7.0]
  def change
    create_table :cart_items do |t|
      t.references :user, null: false, foreign_key: true, on_delete: :cascade
      t.references :product, null: false, foreign_key: true, on_delete: :cascade
      t.integer :quantity, null: false, default: 1
      t.decimal :unit_price, precision: 10, scale: 2, null: false
      t.decimal :total_price, precision: 10, scale: 2, null: false
      t.json :product_snapshot # Store product details at time of adding

      t.timestamps
    end

    add_index :cart_items, [:user_id, :product_id], unique: true
    add_index :cart_items, :created_at
  end
end
```

### Orders Migration
```ruby
class CreateOrders < ActiveRecord::Migration[7.0]
  def change
    create_table :orders do |t|
      t.string :order_number, null: false, index: { unique: true }
      t.references :user, null: false, foreign_key: true
      t.integer :status, default: 0, null: false
      t.decimal :subtotal, precision: 10, scale: 2, null: false
      t.decimal :tax_amount, precision: 10, scale: 2, default: 0
      t.decimal :shipping_amount, precision: 10, scale: 2, default: 0
      t.decimal :discount_amount, precision: 10, scale: 2, default: 0
      t.decimal :total_amount, precision: 10, scale: 2, null: false
      t.string :currency, default: 'USD'
      t.json :shipping_address, null: false
      t.json :billing_address, null: false
      t.text :notes
      t.datetime :shipped_at
      t.datetime :delivered_at
      t.string :tracking_number
      t.string :payment_method
      t.string :payment_status, default: 'pending'

      t.timestamps
    end

    add_index :orders, [:user_id, :created_at]
    add_index :orders, [:status, :created_at]
    add_index :orders, [:order_number]
  end
end
```

### Order Items Migration
```ruby
class CreateOrderItems < ActiveRecord::Migration[7.0]
  def change
    create_table :order_items do |t|
      t.references :order, null: false, foreign_key: true, on_delete: :cascade
      t.references :product, null: false, foreign_key: true
      t.json :product_snapshot, null: false # Store product details at time of order
      t.integer :quantity, null: false
      t.decimal :unit_price, precision: 10, scale: 2, null: false
      t.decimal :total_price, precision: 10, scale: 2, null: false

      t.timestamps
    end

    add_index :order_items, [:order_id, :product_id]
  end
end
```

---

## 🛠️ Rails Models

### User Model
```ruby
class User < ApplicationRecord
  # Enums
  enum role: { customer: 0, vendor: 1, admin: 2 }
  enum status: { active: 0, inactive: 1, suspended: 2 }

  # Associations
  has_many :products, foreign_key: 'vendor_id', dependent: :destroy
  has_many :orders, dependent: :destroy
  has_many :cart_items, dependent: :destroy
  has_many :products_in_cart, through: :cart_items, source: :product
  has_many :reviews, dependent: :destroy

  # Validations
  validates :email, presence: true, uniqueness: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :first_name, presence: true, length: { minimum: 2, maximum: 50 }
  validates :last_name, presence: true, length: { minimum: 2, maximum: 50 }
  validates :password, presence: true, length: { minimum: 8 }, if: :password_required?
  validates :phone_number, format: { with: /\A\+?[\d\s\-\(\)]+\z/ }, allow_blank: true

  # Callbacks
  before_save :normalize_email
  before_create :generate_confirmation_token
  after_create :send_confirmation_email

  # Methods
  def full_name
    "#{first_name} #{last_name}"
  end

  def admin?
    role == 'admin'
  end

  def vendor?
    role == 'vendor'
  end

  def customer?
    role == 'customer'
  end

  def cart_total
    cart_items.sum(:total_price)
  end

  def cart_items_count
    cart_items.sum(:quantity)
  end

  private

  def normalize_email
    self.email = email.downcase.strip
  end

  def password_required?
    new_record? || password.present?
  end

  def generate_confirmation_token
    self.confirmation_token = SecureRandom.urlsafe_base64
    self.confirmation_sent_at = Time.current
  end

  def send_confirmation_email
    UserMailer.confirmation_instructions(self).deliver_later
  end
end
```

### Product Model
```ruby
class Product < ApplicationRecord
  # Associations
  belongs_to :category
  belongs_to :vendor, class_name: 'User'
  has_many :product_images, dependent: :destroy
  has_many :cart_items, dependent: :destroy
  has_many :order_items, dependent: :destroy
  has_many :reviews, dependent: :destroy

  # Validations
  validates :name, presence: true, length: { minimum: 3, maximum: 200 }
  validates :slug, presence: true, uniqueness: true, format: { with: /\A[a-z0-9\-]+\z/ }
  validates :price, presence: true, numericality: { greater_than: 0 }
  validates :stock_quantity, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :sku, presence: true, uniqueness: true

  # Callbacks
  before_validation :generate_slug
  before_save :calculate_average_rating
  after_update :check_low_stock

  # Scopes
  scope :active, -> { where(is_active: true) }
  scope :featured, -> { where(is_featured: true) }
  scope :in_stock, -> { where('stock_quantity > 0') }
  scope :low_stock, -> { where('stock_quantity <= low_stock_threshold') }
  scope :by_category, ->(category_id) { where(category_id: category_id) }
  scope :price_range, ->(min, max) { where(price: min..max) }
  scope :search, ->(query) { where("name ILIKE ? OR description ILIKE ?", "%#{query}%", "%#{query}%") }

  # Methods
  def primary_image
    product_images.find_by(is_primary: true) || product_images.first
  end

  def in_stock?
    stock_quantity > 0
  end

  def low_stock?
    stock_quantity <= low_stock_threshold
  end

  def available_quantity
    [stock_quantity, 0].max
  end

  def decrease_stock(quantity)
    update!(stock_quantity: stock_quantity - quantity)
  end

  def increase_stock(quantity)
    update!(stock_quantity: stock_quantity + quantity)
  end

  private

  def generate_slug
    self.slug = name.parameterize if slug.blank?
  end

  def calculate_average_rating
    if reviews.any?
      self.average_rating = reviews.average(:rating).round(2)
      self.reviews_count = reviews.count
    end
  end

  def check_low_stock
    if low_stock? && stock_quantity_changed?
      LowStockNotificationJob.perform_later(self)
    end
  end
end
```

---

## 🛠️ Rails API Controllers

### Authentication Controller
```ruby
class Api::V1::AuthController < ApplicationController
  before_action :authenticate_user!, only: [:logout, :refresh]

  def register
    user = User.new(user_params)
    
    if user.save
      token = generate_jwt_token(user)
      render json: {
        success: true,
        data: {
          user: user_response(user),
          access_token: token,
          expires_in: 1.hour.to_i
        }
      }, status: :created
    else
      render json: {
        success: false,
        errors: user.errors.full_messages
      }, status: :unprocessable_entity
    end
  end

  def login
    user = User.find_by(email: params[:email])
    
    if user&.authenticate(params[:password]) && user.active?
      user.update!(last_login_at: Time.current)
      token = generate_jwt_token(user)
      
      render json: {
        success: true,
        data: {
          user: user_response(user),
          access_token: token,
          expires_in: 1.hour.to_i
        }
      }
    else
      render json: {
        success: false,
        message: 'Invalid email or password'
      }, status: :unauthorized
    end
  end

  def logout
    # In a real app, you might want to blacklist the token
    render json: { success: true, message: 'Logged out successfully' }
  end

  def refresh
    token = generate_jwt_token(current_user)
    
    render json: {
      success: true,
      data: {
        access_token: token,
        expires_in: 1.hour.to_i
      }
    }
  end

  private

  def user_params
    params.require(:user).permit(:email, :password, :first_name, :last_name, :phone_number)
  end

  def user_response(user)
    {
      id: user.id,
      email: user.email,
      first_name: user.first_name,
      last_name: user.last_name,
      role: user.role,
      email_verified: user.email_verified,
      created_at: user.created_at
    }
  end

  def generate_jwt_token(user)
    payload = {
      user_id: user.id,
      email: user.email,
      role: user.role,
      exp: 1.hour.from_now.to_i
    }
    
    JWT.encode(payload, Rails.application.secrets.secret_key_base)
  end
end
```

### Products Controller
```ruby
class Api::V1::ProductsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]
  before_action :set_product, only: [:show, :update, :destroy]
  before_action :authorize_vendor!, only: [:create, :update, :destroy]

  def index
    @products = Product.active.includes(:category, :vendor, :product_images)
                      .page(params[:page])
                      .per(params[:per_page] || 20)

    # Apply filters
    @products = @products.by_category(params[:category_id]) if params[:category_id]
    @products = @products.price_range(params[:min_price], params[:max_price]) if params[:min_price] && params[:max_price]
    @products = @products.search(params[:search]) if params[:search]
    @products = @products.where(tags: params[:tags]) if params[:tags]

    # Apply sorting
    case params[:sort]
    when 'price_asc'
      @products = @products.order(:price)
    when 'price_desc'
      @products = @products.order(price: :desc)
    when 'name_asc'
      @products = @products.order(:name)
    when 'name_desc'
      @products = @products.order(name: :desc)
    when 'rating_desc'
      @products = @products.order(average_rating: :desc)
    else
      @products = @products.order(created_at: :desc)
    end

    render json: {
      success: true,
      data: {
        products: @products.map { |product| product_response(product) },
        pagination: pagination_response(@products)
      }
    }
  end

  def show
    @product.increment!(:views_count)
    
    render json: {
      success: true,
      data: {
        product: detailed_product_response(@product)
      }
    }
  end

  def create
    @product = current_user.products.build(product_params)
    
    if @product.save
      render json: {
        success: true,
        data: {
          product: detailed_product_response(@product)
        }
      }, status: :created
    else
      render json: {
        success: false,
        errors: @product.errors.full_messages
      }, status: :unprocessable_entity
    end
  end

  def update
    if @product.update(product_params)
      render json: {
        success: true,
        data: {
          product: detailed_product_response(@product)
        }
      }
    else
      render json: {
        success: false,
        errors: @product.errors.full_messages
      }, status: :unprocessable_entity
    end
  end

  def destroy
    @product.destroy
    render json: { success: true, message: 'Product deleted successfully' }
  end

  private

  def set_product
    @product = Product.find(params[:id])
  end

  def product_params
    params.require(:product).permit(
      :name, :description, :short_description, :sku, :price, :compare_price,
      :cost_price, :stock_quantity, :low_stock_threshold, :weight,
      :category_id, :is_active, :is_featured, :meta_title, :meta_description,
      :tags, dimensions: {}, product_images_attributes: [:id, :image_url, :alt_text, :sort_order, :is_primary, :_destroy]
    )
  end

  def authorize_vendor!
    unless current_user.vendor? || current_user.admin?
      render json: { success: false, message: 'Unauthorized' }, status: :forbidden
    end
  end

  def product_response(product)
    {
      id: product.id,
      name: product.name,
      slug: product.slug,
      price: product.price,
      compare_price: product.compare_price,
      stock_quantity: product.stock_quantity,
      is_active: product.is_active,
      is_featured: product.is_featured,
      average_rating: product.average_rating,
      reviews_count: product.reviews_count,
      category: {
        id: product.category.id,
        name: product.category.name
      },
      vendor: {
        id: product.vendor.id,
        name: product.vendor.full_name
      },
      primary_image: product.primary_image&.image_url,
      created_at: product.created_at
    }
  end

  def detailed_product_response(product)
    product_response(product).merge({
      description: product.description,
      short_description: product.short_description,
      sku: product.sku,
      cost_price: product.cost_price,
      low_stock_threshold: product.low_stock_threshold,
      weight: product.weight,
      dimensions: product.dimensions,
      tags: product.tags,
      views_count: product.views_count,
      images: product.product_images.map do |image|
        {
          id: image.id,
          url: image.image_url,
          alt_text: image.alt_text,
          sort_order: image.sort_order,
          is_primary: image.is_primary
        }
      end,
      updated_at: product.updated_at
    })
  end

  def pagination_response(collection)
    {
      current_page: collection.current_page,
      per_page: collection.limit_value,
      total_pages: collection.total_pages,
      total_count: collection.total_count,
      has_next_page: collection.next_page.present?,
      has_prev_page: collection.prev_page.present?
    }
  end
end
```

---

## 🧪 Testing Requirements

### RSpec Configuration
```ruby
# spec/rails_helper.rb
RSpec.configure do |config|
  config.include FactoryBot::Syntax::Methods
  config.include Devise::Test::IntegrationHelpers, type: :request
  
  config.before(:suite) do
    DatabaseCleaner.strategy = :transaction
    DatabaseCleaner.clean_with(:truncation)
  end

  config.around(:each) do |example|
    DatabaseCleaner.cleaning do
      example.run
    end
  end
end
```

### Factory Definitions
```ruby
# spec/factories/users.rb
FactoryBot.define do
  factory :user do
    email { Faker::Internet.email }
    password { 'password123' }
    first_name { Faker::Name.first_name }
    last_name { Faker::Name.last_name }
    phone_number { Faker::PhoneNumber.phone_number }
    role { :customer }
    email_verified { true }

    trait :vendor do
      role { :vendor }
    end

    trait :admin do
      role { :admin }
    end

    trait :unverified do
      email_verified { false }
      confirmation_token { SecureRandom.urlsafe_base64 }
    end
  end
end

# spec/factories/products.rb
FactoryBot.define do
  factory :product do
    name { Faker::Commerce.product_name }
    description { Faker::Lorem.paragraph }
    price { Faker::Commerce.price(range: 10.0..1000.0) }
    stock_quantity { Faker::Number.between(from: 0, to: 100) }
    association :category
    association :vendor, factory: [:user, :vendor]
    
    trait :featured do
      is_featured { true }
    end

    trait :out_of_stock do
      stock_quantity { 0 }
    end

    trait :low_stock do
      stock_quantity { 5 }
      low_stock_threshold { 10 }
    end
  end
end
```

### Controller Tests
```ruby
# spec/requests/api/v1/auth_spec.rb
require 'rails_helper'

RSpec.describe 'Api::V1::Auth', type: :request do
  describe 'POST /api/v1/auth/register' do
    let(:valid_params) do
      {
        user: {
          email: 'test@example.com',
          password: 'password123',
          first_name: 'John',
          last_name: 'Doe'
        }
      }
    end

    context 'with valid parameters' do
      it 'creates a new user' do
        expect {
          post '/api/v1/auth/register', params: valid_params
        }.to change(User, :count).by(1)

        expect(response).to have_http_status(:created)
        expect(json_response['success']).to be true
        expect(json_response['data']['user']['email']).to eq('test@example.com')
        expect(json_response['data']['access_token']).to be_present
      end
    end

    context 'with invalid parameters' do
      let(:invalid_params) { { user: { email: 'invalid-email' } } }

      it 'returns validation errors' do
        post '/api/v1/auth/register', params: invalid_params

        expect(response).to have_http_status(:unprocessable_entity)
        expect(json_response['success']).to be false
        expect(json_response['errors']).to be_present
      end
    end
  end

  describe 'POST /api/v1/auth/login' do
    let(:user) { create(:user, email: 'test@example.com', password: 'password123') }

    context 'with valid credentials' do
      it 'returns access token' do
        post '/api/v1/auth/login', params: { email: 'test@example.com', password: 'password123' }

        expect(response).to have_http_status(:ok)
        expect(json_response['success']).to be true
        expect(json_response['data']['access_token']).to be_present
        expect(json_response['data']['user']['email']).to eq('test@example.com')
      end
    end

    context 'with invalid credentials' do
      it 'returns unauthorized' do
        post '/api/v1/auth/login', params: { email: 'test@example.com', password: 'wrongpassword' }

        expect(response).to have_http_status(:unauthorized)
        expect(json_response['success']).to be false
      end
    end
  end

  private

  def json_response
    JSON.parse(response.body)
  end
end
```

---

## 🚀 Deployment Requirements

### Environment Configuration
```ruby
# config/environments/production.rb
Rails.application.configure do
  config.cache_classes = true
  config.eager_load = true
  config.consider_all_requests_local = false
  config.public_file_server.enabled = ENV['RAILS_SERVE_STATIC_FILES'].present?
  config.log_level = :info
  config.log_tags = [ :request_id ]
  config.action_mailer.perform_caching = false
  config.i18n.fallbacks = true
  config.active_support.deprecation = :notify
  config.log_formatter = ::Logger::Formatter.new
  config.active_record.dump_schema_after_migration = false

  # Redis configuration
  config.cache_store = :redis_cache_store, {
    url: ENV['REDIS_URL'],
    expires_in: 1.hour
  }

  # Database configuration
  config.active_record.database_url = ENV['DATABASE_URL']

  # Mail configuration
  config.action_mailer.default_url_options = { host: ENV['APP_HOST'] }
  config.action_mailer.delivery_method = :smtp
  config.action_mailer.smtp_settings = {
    address: ENV['SMTP_ADDRESS'],
    port: ENV['SMTP_PORT'],
    domain: ENV['SMTP_DOMAIN'],
    user_name: ENV['SMTP_USERNAME'],
    password: ENV['SMTP_PASSWORD'],
    authentication: 'plain',
    enable_starttls_auto: true
  }
end
```

### Docker Configuration
```dockerfile
FROM ruby:3.2-alpine

# Install dependencies
RUN apk add --no-cache \
    build-base \
    postgresql-dev \
    tzdata \
    curl

WORKDIR /app

# Install gems
COPY Gemfile Gemfile.lock ./
RUN bundle config set --local without 'development test' && \
    bundle install --jobs 4 --retry 3

# Copy application code
COPY . .

# Precompile assets
RUN bundle exec rails assets:precompile RAILS_ENV=production

# Create non-root user
RUN addgroup -g 1001 -S rails && \
    adduser -S rails -u 1001
USER rails

EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0", "-e", "production"]
```

---

## 📚 Learning Resources

### Essential Reading
- [Rails Guides](https://guides.rubyonrails.org/) - Framework fundamentals
- [Active Record Documentation](https://guides.rubyonrails.org/active_record_basics/) - ORM
- [Rails API Documentation](https://guides.rubyonrails.org/api_app/) - API development
- [RSpec Documentation](https://rspec.info/documentation/) - Testing framework

### Udemy Courses (Focus Sections)
- **Complete Ruby on Rails Developer Course** - Sections 1-8 (Fundamentals, Models, Controllers)
- **Rails API Development** - Sections 2-6 (JSON APIs, Authentication, Testing)

### Practice Projects
- Build a simple blog API with Rails
- Create a todo application with user authentication
- Implement a basic e-commerce API

---

## ✅ Success Criteria

### Week 1 Goals
- [ ] Set up Rails API application with PostgreSQL
- [ ] Implement user authentication system
- [ ] Create product and category models
- [ ] Build basic CRUD operations
- [ ] Write unit tests for models

### Week 2 Goals
- [ ] Implement shopping cart functionality
- [ ] Create order management system
- [ ] Add comprehensive API documentation
- [ ] Implement caching with Redis
- [ ] Deploy to Heroku/AWS
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional Rails API
- [ ] Comprehensive API documentation
- [ ] 80%+ test coverage with RSpec
- [ ] Deployed application with live demo
- [ ] GitHub repository with clean commit history
- [ ] README with setup instructions

---

## 🔗 Related Projects
- **Next Project**: [Project 2: Data Processing Platform](../Backend_Intensive/Project_2_Data_Processing_Platform/)
- **Training Plan**: [Backend Intensive Plan](../../Intensive_Plans/Training_Plan_Backend_Intensive.html/)
- **Main README**: [Repository Overview](../../README.html)
