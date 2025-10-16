# Project 1: Rails E-commerce Platform
*Full-featured e-commerce platform with Rails, PostgreSQL, and Stripe*

## Project Overview

Build a production-ready e-commerce platform using Ruby on Rails, showcasing MVC architecture, authentication, payment processing, and admin management. This project demonstrates your expertise in building scalable, secure web applications with comprehensive testing.

## Technology Stack

- **Framework**: Ruby on Rails 7.1+
- **Language**: Ruby 3.2+
- **Database**: PostgreSQL 15+
- **Authentication**: Devise
- **Authorization**: Pundit
- **Payment**: Stripe
- **Admin**: ActiveAdmin
- **Testing**: RSpec + FactoryBot
- **Background Jobs**: Sidekiq
- **Deployment**: Docker + Heroku/AWS

## Core Features

### 1. User Management
- User registration and authentication (Devise)
- User profiles and preferences
- Password reset functionality
- Email verification
- Social authentication (OAuth)
- User roles (customer, admin)

### 2. Product Catalog
- Product CRUD operations
- Product categories and variants
- Product search and filtering
- Product images (Active Storage)
- Inventory management
- Product reviews and ratings

### 3. Shopping Cart & Orders
- Shopping cart management (session-based)
- Order creation and processing
- Order status tracking
- Order history
- Order cancellation and refunds
- Email notifications

### 4. Payment Processing
- Payment method management
- Stripe integration
- Secure checkout process
- Payment confirmation
- Refund processing
- Invoice generation

### 5. Admin Features
- ActiveAdmin dashboard
- User management
- Product management
- Order management
- Analytics and reporting
- Content management

## Rails Routes

### Authentication (Devise)
```ruby
# config/routes.rb
devise_for :users, controllers: {
  registrations: 'users/registrations',
  sessions: 'users/sessions'
}
```

### Products
```ruby
resources :products do
  collection do
    get :search
  end
  member do
    post :add_to_cart
  end
  resources :reviews, only: [:create, :destroy]
end

resources :categories, only: [:index, :show]
```

### Cart & Orders
```ruby
resource :cart, only: [:show, :update, :destroy] do
  post :add_item
  delete :remove_item
end

resources :orders do
  member do
    post :cancel
    get :invoice
  end
end
```

### Payments
```ruby
namespace :payments do
  post :create_intent
  post :confirm
  post :refund
end
```

### Admin
```ruby
ActiveAdmin.routes(self)
```

## Database Schema

### Users Table
```ruby
# db/migrate/xxx_devise_create_users.rb
create_table :users do |t|
  t.string :email, null: false, default: ""
  t.string :encrypted_password, null: false, default: ""
  t.string :reset_password_token
  t.datetime :reset_password_sent_at
  t.datetime :remember_created_at
  t.string :first_name
  t.string :last_name
  t.string :role, default: "customer"
  t.timestamps
end

add_index :users, :email, unique: true
add_index :users, :reset_password_token, unique: true
```

### Products Table
```ruby
create_table :products do |t|
  t.string :name, null: false
  t.text :description
  t.decimal :price, precision: 10, scale: 2, null: false
  t.references :category, foreign_key: true
  t.integer :stock_quantity, default: 0
  t.boolean :active, default: true
  t.string :sku
  t.timestamps
end

add_index :products, :sku, unique: true
add_index :products, :active
```

### Orders Table
```ruby
create_table :orders do |t|
  t.references :user, foreign_key: true, null: false
  t.decimal :total_amount, precision: 10, scale: 2, null: false
  t.string :status, default: "pending"
  t.jsonb :shipping_address
  t.jsonb :billing_address
  t.string :stripe_payment_intent_id
  t.timestamps
end

add_index :orders, :user_id
add_index :orders, :status
```

### Order Items Table
```ruby
create_table :order_items do |t|
  t.references :order, foreign_key: true, null: false
  t.references :product, foreign_key: true, null: false
  t.integer :quantity, null: false
  t.decimal :price, precision: 10, scale: 2, null: false
  t.timestamps
end
```

## Implementation Guide

### Week 1: Foundation & Core Features

#### Day 1-2: Project Setup
1. **Create Rails Application**
   ```bash
   rails new ecommerce_platform --database=postgresql
   cd ecommerce_platform
   bundle install
   rails db:create
   ```

2. **Install Core Gems**
   ```ruby
   # Gemfile
   gem 'devise'
   gem 'pundit'
   gem 'activeadmin'
   gem 'stripe'
   gem 'sidekiq'
   gem 'image_processing'
   
   group :development, :test do
     gem 'rspec-rails'
     gem 'factory_bot_rails'
     gem 'faker'
   end
   ```

3. **Configure Database**
   ```bash
   rails db:create
   rails db:migrate
   ```

#### Day 3-5: Authentication & Authorization
1. **Install Devise**
   ```bash
   rails generate devise:install
   rails generate devise User
   rails generate migration AddFieldsToUsers first_name:string last_name:string role:string
   rails db:migrate
   ```

2. **User Model**
   ```ruby
   # app/models/user.rb
   class User < ApplicationRecord
     devise :database_authenticatable, :registerable,
            :recoverable, :rememberable, :validatable
     
     has_many :orders, dependent: :destroy
     
     enum role: { customer: 'customer', admin: 'admin' }
     
     def full_name
       "#{first_name} #{last_name}"
     end
   end
   ```

3. **Authorization with Pundit**
   ```bash
   rails generate pundit:install
   rails generate pundit:policy Product
   ```

#### Day 6-7: Product Management
1. **Generate Product Models**
   ```bash
   rails generate model Category name:string description:text
   rails generate model Product name:string description:text price:decimal category:references stock_quantity:integer active:boolean sku:string
   rails db:migrate
   ```

2. **Product Model**
   ```ruby
   # app/models/product.rb
   class Product < ApplicationRecord
     belongs_to :category
     has_many :order_items
     has_many_attached :images
     has_many :reviews, dependent: :destroy
     
     validates :name, :price, presence: true
     validates :price, numericality: { greater_than: 0 }
     validates :sku, uniqueness: true
     
     scope :active, -> { where(active: true) }
     scope :in_stock, -> { where('stock_quantity > ?', 0) }
     
     def in_stock?
       stock_quantity > 0
     end
   end
   ```

3. **Products Controller**
   ```ruby
   # app/controllers/products_controller.rb
   class ProductsController < ApplicationController
     before_action :set_product, only: [:show, :edit, :update, :destroy]
     
     def index
       @products = Product.active.includes(:category).page(params[:page])
     end
     
     def show
       @reviews = @product.reviews.includes(:user).order(created_at: :desc)
     end
     
     def search
       @products = Product.where('name ILIKE ?', "%#{params[:q]}%")
                         .active
                         .page(params[:page])
     end
     
     private
     
     def set_product
       @product = Product.find(params[:id])
     end
   end
   ```

### Week 2: Shopping Cart, Orders & Payment

#### Day 8-10: Shopping Cart
1. **Cart Service**
   ```ruby
   # app/services/cart_service.rb
   class CartService
     def initialize(session)
       @session = session
       @session[:cart] ||= {}
     end
     
     def add_item(product_id, quantity = 1)
       product_id = product_id.to_s
       @session[:cart][product_id] ||= 0
       @session[:cart][product_id] += quantity
     end
     
     def remove_item(product_id)
       @session[:cart].delete(product_id.to_s)
     end
     
     def items
       product_ids = @session[:cart].keys
       products = Product.find(product_ids)
       products.map do |product|
         {
           product: product,
           quantity: @session[:cart][product.id.to_s]
         }
       end
     end
     
     def total
       items.sum { |item| item[:product].price * item[:quantity] }
     end
     
     def clear
       @session[:cart] = {}
     end
   end
   ```

2. **Cart Controller**
   ```ruby
   # app/controllers/carts_controller.rb
   class CartsController < ApplicationController
     before_action :authenticate_user!
     
     def show
       @cart = CartService.new(session)
       @cart_items = @cart.items
     end
     
     def add_item
       @cart = CartService.new(session)
       @cart.add_item(params[:product_id], params[:quantity].to_i)
       redirect_to cart_path, notice: 'Product added to cart'
     end
     
     def remove_item
       @cart = CartService.new(session)
       @cart.remove_item(params[:product_id])
       redirect_to cart_path, notice: 'Product removed from cart'
     end
   end
   ```

#### Day 11-12: Order Processing
1. **Generate Order Models**
   ```bash
   rails generate model Order user:references total_amount:decimal status:string shipping_address:jsonb billing_address:jsonb stripe_payment_intent_id:string
   rails generate model OrderItem order:references product:references quantity:integer price:decimal
   rails db:migrate
   ```

2. **Order Model**
   ```ruby
   # app/models/order.rb
   class Order < ApplicationRecord
     belongs_to :user
     has_many :order_items, dependent: :destroy
     has_many :products, through: :order_items
     
     enum status: {
       pending: 'pending',
       paid: 'paid',
       processing: 'processing',
       shipped: 'shipped',
       delivered: 'delivered',
       cancelled: 'cancelled'
     }
     
     validates :total_amount, presence: true, numericality: { greater_than: 0 }
     
     def calculate_total
       order_items.sum { |item| item.price * item.quantity }
     end
   end
   ```

3. **Order Service**
   ```ruby
   # app/services/order_service.rb
   class OrderService
     def self.create_from_cart(user, cart_service, addresses)
       Order.transaction do
         order = user.orders.create!(
           total_amount: cart_service.total,
           shipping_address: addresses[:shipping],
           billing_address: addresses[:billing],
           status: 'pending'
         )
         
         cart_service.items.each do |item|
           order.order_items.create!(
             product: item[:product],
             quantity: item[:quantity],
             price: item[:product].price
           )
         end
         
         cart_service.clear
         order
       end
     end
   end
   ```

#### Day 13-14: Stripe Payment Integration
1. **Configure Stripe**
   ```ruby
   # config/initializers/stripe.rb
   Stripe.api_key = Rails.application.credentials.dig(:stripe, :secret_key)
   ```

2. **Payment Service**
   ```ruby
   # app/services/payment_service.rb
   class PaymentService
     def self.create_payment_intent(order)
       Stripe::PaymentIntent.create(
         amount: (order.total_amount * 100).to_i,
         currency: 'usd',
         metadata: { order_id: order.id }
       )
     end
     
     def self.confirm_payment(order, payment_intent_id)
       payment_intent = Stripe::PaymentIntent.retrieve(payment_intent_id)
       
       if payment_intent.status == 'succeeded'
         order.update!(
           status: 'paid',
           stripe_payment_intent_id: payment_intent_id
         )
         OrderMailer.confirmation(order).deliver_later
         true
       else
         false
       end
     end
     
     def self.refund_payment(order)
       return false unless order.stripe_payment_intent_id
       
       refund = Stripe::Refund.create(
         payment_intent: order.stripe_payment_intent_id
       )
       
       if refund.status == 'succeeded'
         order.update!(status: 'cancelled')
         OrderMailer.refund_confirmation(order).deliver_later
         true
       else
         false
       end
     end
   end
   ```

3. **Payments Controller**
   ```ruby
   # app/controllers/payments_controller.rb
   class PaymentsController < ApplicationController
     before_action :authenticate_user!
     
     def create_intent
       @order = current_user.orders.find(params[:order_id])
       @payment_intent = PaymentService.create_payment_intent(@order)
       
       render json: { client_secret: @payment_intent.client_secret }
     end
     
     def confirm
       @order = current_user.orders.find(params[:order_id])
       
       if PaymentService.confirm_payment(@order, params[:payment_intent_id])
         redirect_to order_path(@order), notice: 'Payment successful!'
       else
         redirect_to order_path(@order), alert: 'Payment failed'
       end
     end
   end
   ```

## Testing Requirements

### Test Coverage
- **Unit Tests**: 90%+ coverage
- **Integration Tests**: All critical user flows
- **System Tests**: Complete checkout process
- **Model Tests**: All validations and associations

### RSpec Test Examples
```ruby
# spec/models/product_spec.rb
require 'rails_helper'

RSpec.describe Product, type: :model do
  describe 'validations' do
    it { should validate_presence_of(:name) }
    it { should validate_presence_of(:price) }
    it { should validate_numericality_of(:price).is_greater_than(0) }
  end
  
  describe 'associations' do
    it { should belong_to(:category) }
    it { should have_many(:order_items) }
  end
  
  describe '#in_stock?' do
    it 'returns true when stock_quantity > 0' do
      product = create(:product, stock_quantity: 5)
      expect(product.in_stock?).to be true
    end
  end
end

# spec/services/order_service_spec.rb
require 'rails_helper'

RSpec.describe OrderService do
  describe '.create_from_cart' do
    let(:user) { create(:user) }
    let(:product) { create(:product, price: 10.00) }
    let(:cart) { instance_double(CartService) }
    
    before do
      allow(cart).to receive(:items).and_return([
        { product: product, quantity: 2 }
      ])
      allow(cart).to receive(:total).and_return(20.00)
      allow(cart).to receive(:clear)
    end
    
    it 'creates an order with items' do
      order = OrderService.create_from_cart(user, cart, {
        shipping: { address: '123 Main St' },
        billing: { address: '123 Main St' }
      })
      
      expect(order).to be_persisted
      expect(order.order_items.count).to eq(1)
      expect(order.total_amount).to eq(20.00)
    end
  end
end

# spec/requests/products_spec.rb
require 'rails_helper'

RSpec.describe 'Products', type: :request do
  describe 'GET /products' do
    it 'returns a successful response' do
      create_list(:product, 3)
      get products_path
      expect(response).to have_http_status(:success)
    end
  end
  
  describe 'GET /products/:id' do
    it 'returns product details' do
      product = create(:product)
      get product_path(product)
      expect(response).to have_http_status(:success)
      expect(response.body).to include(product.name)
    end
  end
end
```

## ActiveAdmin Setup

```ruby
# app/admin/products.rb
ActiveAdmin.register Product do
  permit_params :name, :description, :price, :category_id, :stock_quantity, :active, :sku
  
  index do
    selectable_column
    id_column
    column :name
    column :category
    column :price
    column :stock_quantity
    column :active
    actions
  end
  
  filter :name
  filter :category
  filter :active
  filter :created_at
  
  form do |f|
    f.inputs do
      f.input :name
      f.input :description
      f.input :price
      f.input :category
      f.input :stock_quantity
      f.input :sku
      f.input :active
    end
    f.actions
  end
end
```

## Background Jobs

```ruby
# app/jobs/order_confirmation_job.rb
class OrderConfirmationJob < ApplicationJob
  queue_as :default
  
  def perform(order_id)
    order = Order.find(order_id)
    OrderMailer.confirmation(order).deliver_now
  end
end

# app/mailers/order_mailer.rb
class OrderMailer < ApplicationMailer
  def confirmation(order)
    @order = order
    @user = order.user
    mail(to: @user.email, subject: "Order Confirmation ##{@order.id}")
  end
  
  def refund_confirmation(order)
    @order = order
    @user = order.user
    mail(to: @user.email, subject: "Refund Confirmed for Order ##{@order.id}")
  end
end
```

## Performance Requirements

### Response Times
- **Page Load**: < 500ms average
- **Database Queries**: < 100ms average
- **API Endpoints**: < 200ms average

### Scalability
- Handle 1000+ concurrent users
- Support 10,000+ products
- Process 100+ orders per hour

## Deployment Guide

### Docker Setup
```dockerfile
# Dockerfile
FROM ruby:3.2

WORKDIR /app

COPY Gemfile Gemfile.lock ./
RUN bundle install

COPY . .

CMD ["rails", "server", "-b", "0.0.0.0"]
```

### Heroku Deployment
```bash
heroku create ecommerce-platform
heroku addons:create heroku-postgresql:hobby-dev
heroku addons:create heroku-redis:hobby-dev
git push heroku main
heroku run rails db:migrate
```

## Success Criteria

- [ ] Complete e-commerce functionality
- [ ] Stripe payment integration working
- [ ] 90%+ test coverage with RSpec
- [ ] ActiveAdmin dashboard functional
- [ ] Production deployment on Heroku/AWS
- [ ] Comprehensive documentation
- [ ] Security best practices implemented

## Portfolio Showcase

### GitHub README
- Project overview and features
- Screenshots of key pages
- Setup and installation instructions
- Testing and deployment guides
- Tech stack and architecture

### Live Demo
- Deployed application URL
- Demo credentials
- Feature walkthrough video

---

📋 **[Back to Projects Overview](../Projects/Ruby_Rails_Backend/)** | 🏠 **[Back to Training Plan](../Intensive_Plans/Training_Plan_Ruby_Rails_Backend_Intensive.html)** | 🏠 **[Back to README](../README.html)**

### Related Projects
- **[Project 2: Social Media API](./Project_2_Social_Media_API.html)** - Rails API with real-time features
- **[Project 3: Microservices Platform](./Project_3_Microservices_Platform.html)** - Rails microservices architecture
- **[Project 4: Production System](./Project_4_Production_System.html)** - Full DevOps pipeline

