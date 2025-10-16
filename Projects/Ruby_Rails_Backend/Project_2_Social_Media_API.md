# Project 2: Social Media API
*Rails API with real-time features, user relationships, and content management*

## Project Overview

Build a production-ready social media API using Rails API mode, showcasing RESTful design, real-time features with Action Cable, JWT authentication, and efficient data handling. This project demonstrates expertise in building scalable APIs with complex data relationships.

## Technology Stack

- **Framework**: Rails 7.1+ (API mode)
- **Language**: Ruby 3.2+
- **Database**: PostgreSQL 15+
- **Cache**: Redis 7+
- **Authentication**: JWT
- **Real-time**: Action Cable + WebSockets
- **File Storage**: Active Storage + AWS S3
- **Testing**: RSpec + FactoryBot
- **Background Jobs**: Sidekiq
- **API Docs**: rswag (OpenAPI)
- **Deployment**: Docker + AWS

## Core Features

### 1. User Management
- JWT-based authentication
- User profiles with bio and avatar
- User settings and preferences
- Email verification
- Password reset
- Account deactivation

### 2. User Relationships
- Follow/unfollow users
- Followers and following lists
- Block users
- Mutual friends detection
- Follower recommendations

### 3. Content Management
- Create, edit, delete posts
- Post visibility (public, followers, private)
- Rich media support (images, videos)
- Hashtags and mentions
- Post scheduling
- Draft posts

### 4. Engagement Features
- Like/unlike posts
- Comment on posts
- Reply to comments
- Share/repost content
- Save/bookmark posts
- Post reactions (emoji)

### 5. Real-time Features
- Live notifications
- Real-time feed updates
- Online status tracking
- Typing indicators
- Read receipts

### 6. Feed & Discovery
- Personalized feed algorithm
- Trending posts and hashtags
- User search
- Content discovery
- Hashtag feeds

## API Endpoints

### Authentication
```
POST /api/v1/auth/register
POST /api/v1/auth/login
POST /api/v1/auth/refresh
POST /api/v1/auth/logout
POST /api/v1/auth/forgot-password
POST /api/v1/auth/reset-password
GET /api/v1/auth/me
```

### Users
```
GET /api/v1/users
GET /api/v1/users/:id
PUT /api/v1/users/:id
DELETE /api/v1/users/:id
GET /api/v1/users/:id/followers
GET /api/v1/users/:id/following
POST /api/v1/users/:id/follow
DELETE /api/v1/users/:id/unfollow
POST /api/v1/users/:id/block
DELETE /api/v1/users/:id/unblock
```

### Posts
```
GET /api/v1/posts
POST /api/v1/posts
GET /api/v1/posts/:id
PUT /api/v1/posts/:id
DELETE /api/v1/posts/:id
GET /api/v1/posts/feed
GET /api/v1/posts/trending
POST /api/v1/posts/:id/like
DELETE /api/v1/posts/:id/unlike
POST /api/v1/posts/:id/share
```

### Comments
```
GET /api/v1/posts/:post_id/comments
POST /api/v1/posts/:post_id/comments
PUT /api/v1/comments/:id
DELETE /api/v1/comments/:id
POST /api/v1/comments/:id/like
```

### Notifications
```
GET /api/v1/notifications
PUT /api/v1/notifications/:id/read
PUT /api/v1/notifications/mark-all-read
DELETE /api/v1/notifications/:id
```

## Database Schema

### Users Table
```ruby
create_table :users do |t|
  t.string :email, null: false
  t.string :username, null: false
  t.string :password_digest, null: false
  t.string :full_name
  t.text :bio
  t.string :location
  t.string :website
  t.boolean :verified, default: false
  t.boolean :active, default: true
  t.datetime :last_seen_at
  t.timestamps
end

add_index :users, :email, unique: true
add_index :users, :username, unique: true
```

### Posts Table
```ruby
create_table :posts do |t|
  t.references :user, foreign_key: true, null: false
  t.text :content
  t.string :visibility, default: 'public'
  t.integer :likes_count, default: 0
  t.integer :comments_count, default: 0
  t.integer :shares_count, default: 0
  t.datetime :published_at
  t.timestamps
end

add_index :posts, :user_id
add_index :posts, :visibility
add_index :posts, :published_at
```

### Relationships Table
```ruby
create_table :relationships do |t|
  t.references :follower, foreign_key: { to_table: :users }, null: false
  t.references :followed, foreign_key: { to_table: :users }, null: false
  t.timestamps
end

add_index :relationships, [:follower_id, :followed_id], unique: true
```

## Implementation Guide

### Week 3: Foundation & Authentication

#### Day 1-2: Rails API Setup
```bash
rails new social_media_api --api --database=postgresql
cd social_media_api

# Add gems
# Gemfile
gem 'bcrypt'
gem 'jwt'
gem 'rack-cors'
gem 'active_model_serializers'
gem 'redis'
gem 'sidekiq'

group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
  gem 'rswag'
end

bundle install
rails db:create
```

#### Day 3-5: JWT Authentication
```ruby
# app/models/user.rb
class User < ApplicationRecord
  has_secure_password
  
  has_many :posts, dependent: :destroy
  has_many :active_relationships, class_name: 'Relationship',
                                  foreign_key: 'follower_id',
                                  dependent: :destroy
  has_many :passive_relationships, class_name: 'Relationship',
                                   foreign_key: 'followed_id',
                                   dependent: :destroy
  has_many :following, through: :active_relationships, source: :followed
  has_many :followers, through: :passive_relationships, source: :follower
  
  validates :email, presence: true, uniqueness: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :username, presence: true, uniqueness: true, length: { minimum: 3, maximum: 30 }
  validates :password, length: { minimum: 6 }, if: -> { password.present? }
  
  def follow(other_user)
    following << other_user unless self == other_user || following?(other_user)
  end
  
  def unfollow(other_user)
    following.delete(other_user)
  end
  
  def following?(other_user)
    following.include?(other_user)
  end
end

# lib/json_web_token.rb
class JsonWebToken
  SECRET_KEY = Rails.application.credentials.secret_key_base
  
  def self.encode(payload, exp = 24.hours.from_now)
    payload[:exp] = exp.to_i
    JWT.encode(payload, SECRET_KEY)
  end
  
  def self.decode(token)
    body = JWT.decode(token, SECRET_KEY)[0]
    HashWithIndifferentAccess.new body
  rescue JWT::DecodeError => e
    nil
  end
end

# app/controllers/api/v1/auth_controller.rb
class Api::V1::AuthController < ApplicationController
  skip_before_action :authenticate_request, only: [:login, :register]
  
  def register
    user = User.new(user_params)
    if user.save
      token = JsonWebToken.encode(user_id: user.id)
      render json: { token: token, user: UserSerializer.new(user) }, status: :created
    else
      render json: { errors: user.errors.full_messages }, status: :unprocessable_entity
    end
  end
  
  def login
    user = User.find_by(email: params[:email])
    if user&.authenticate(params[:password])
      token = JsonWebToken.encode(user_id: user.id)
      render json: { token: token, user: UserSerializer.new(user) }
    else
      render json: { error: 'Invalid credentials' }, status: :unauthorized
    end
  end
  
  private
  
  def user_params
    params.require(:user).permit(:email, :username, :password, :full_name)
  end
end

# app/controllers/application_controller.rb
class ApplicationController < ActionController::API
  before_action :authenticate_request
  
  attr_reader :current_user
  
  private
  
  def authenticate_request
    header = request.headers['Authorization']
    header = header.split(' ').last if header
    
    begin
      decoded = JsonWebToken.decode(header)
      @current_user = User.find(decoded[:user_id])
    rescue ActiveRecord::RecordNotFound, JWT::DecodeError => e
      render json: { errors: 'Unauthorized' }, status: :unauthorized
    end
  end
end
```

#### Day 6-7: User Relationships
```ruby
# app/models/relationship.rb
class Relationship < ApplicationRecord
  belongs_to :follower, class_name: 'User'
  belongs_to :followed, class_name: 'User'
  
  validates :follower_id, presence: true
  validates :followed_id, presence: true
  validates :follower_id, uniqueness: { scope: :followed_id }
  validate :cannot_follow_self
  
  private
  
  def cannot_follow_self
    errors.add(:follower_id, "can't follow yourself") if follower_id == followed_id
  end
end

# app/controllers/api/v1/relationships_controller.rb
class Api::V1::RelationshipsController < ApplicationController
  def create
    user = User.find(params[:user_id])
    current_user.follow(user)
    
    # Send notification
    NotificationJob.perform_later(user.id, 'new_follower', current_user.id)
    
    render json: { message: 'Successfully followed user' }, status: :created
  end
  
  def destroy
    user = User.find(params[:user_id])
    current_user.unfollow(user)
    render json: { message: 'Successfully unfollowed user' }
  end
  
  def followers
    user = User.find(params[:user_id])
    followers = user.followers.page(params[:page])
    render json: followers, each_serializer: UserSerializer
  end
  
  def following
    user = User.find(params[:user_id])
    following = user.following.page(params[:page])
    render json: following, each_serializer: UserSerializer
  end
end
```

### Week 4: Posts, Real-time & Advanced Features

#### Day 8-10: Post Management
```ruby
# app/models/post.rb
class Post < ApplicationRecord
  belongs_to :user
  has_many :comments, dependent: :destroy
  has_many :likes, dependent: :destroy
  has_many :liked_by_users, through: :likes, source: :user
  has_many_attached :media
  
  validates :content, presence: true, length: { maximum: 500 }
  validates :visibility, inclusion: { in: %w[public followers private] }
  
  scope :published, -> { where('published_at <= ?', Time.current) }
  scope :public_posts, -> { where(visibility: 'public') }
  scope :for_user_feed, ->(user) {
    where(user_id: user.following.pluck(:id) + [user.id])
    .or(where(visibility: 'public'))
    .published
  }
  
  before_save :extract_hashtags
  
  def liked_by?(user)
    liked_by_users.include?(user)
  end
  
  private
  
  def extract_hashtags
    self.hashtags = content.scan(/#\w+/).map { |tag| tag[1..-1].downcase }.uniq
  end
end

# app/controllers/api/v1/posts_controller.rb
class Api::V1::PostsController < ApplicationController
  def index
    posts = Post.public_posts.published.includes(:user).page(params[:page])
    render json: posts, each_serializer: PostSerializer
  end
  
  def create
    post = current_user.posts.build(post_params)
    if post.save
      BroadcastPostJob.perform_later(post.id)
      render json: post, serializer: PostSerializer, status: :created
    else
      render json: { errors: post.errors.full_messages }, status: :unprocessable_entity
    end
  end
  
  def feed
    posts = Post.for_user_feed(current_user)
                .includes(:user, :comments)
                .order(published_at: :desc)
                .page(params[:page])
    render json: posts, each_serializer: PostSerializer
  end
  
  def trending
    posts = Post.public_posts
                .where('created_at > ?', 24.hours.ago)
                .order(likes_count: :desc, comments_count: :desc)
                .limit(20)
    render json: posts, each_serializer: PostSerializer
  end
  
  private
  
  def post_params
    params.require(:post).permit(:content, :visibility, media: [])
  end
end
```

#### Day 11-12: Real-time with Action Cable
```ruby
# app/channels/notifications_channel.rb
class NotificationsChannel < ApplicationCable::Channel
  def subscribed
    stream_for current_user
  end
  
  def unsubscribed
    stop_all_streams
  end
end

# app/channels/feed_channel.rb
class FeedChannel < ApplicationCable::Channel
  def subscribed
    stream_from "feed_#{current_user.id}"
  end
  
  def unsubscribed
    stop_all_streams
  end
end

# app/jobs/broadcast_post_job.rb
class BroadcastPostJob < ApplicationJob
  queue_as :default
  
  def perform(post_id)
    post = Post.find(post_id)
    user = post.user
    
    # Broadcast to all followers
    user.followers.each do |follower|
      ActionCable.server.broadcast(
        "feed_#{follower.id}",
        {
          type: 'new_post',
          post: PostSerializer.new(post).as_json
        }
      )
    end
  end
end

# app/jobs/notification_job.rb
class NotificationJob < ApplicationJob
  queue_as :default
  
  def perform(user_id, notification_type, actor_id, notifiable_id = nil)
    user = User.find(user_id)
    notification = Notification.create!(
      user: user,
      notification_type: notification_type,
      actor_id: actor_id,
      notifiable_id: notifiable_id
    )
    
    NotificationsChannel.broadcast_to(
      user,
      NotificationSerializer.new(notification).as_json
    )
  end
end
```

#### Day 13-14: Testing & Documentation
```ruby
# spec/models/user_spec.rb
require 'rails_helper'

RSpec.describe User, type: :model do
  describe 'validations' do
    it { should validate_presence_of(:email) }
    it { should validate_presence_of(:username) }
    it { should validate_uniqueness_of(:email).case_insensitive }
    it { should validate_uniqueness_of(:username) }
  end
  
  describe 'associations' do
    it { should have_many(:posts).dependent(:destroy) }
    it { should have_many(:following) }
    it { should have_many(:followers) }
  end
  
  describe '#follow' do
    let(:user) { create(:user) }
    let(:other_user) { create(:user) }
    
    it 'follows another user' do
      expect {
        user.follow(other_user)
      }.to change(user.following, :count).by(1)
    end
    
    it 'does not follow the same user twice' do
      user.follow(other_user)
      expect {
        user.follow(other_user)
      }.not_to change(user.following, :count)
    end
  end
end

# spec/requests/api/v1/posts_spec.rb
require 'rails_helper'

RSpec.describe 'Api::V1::Posts', type: :request do
  let(:user) { create(:user) }
  let(:token) { JsonWebToken.encode(user_id: user.id) }
  let(:headers) { { 'Authorization' => "Bearer #{token}" } }
  
  describe 'GET /api/v1/posts' do
    it 'returns public posts' do
      create_list(:post, 3, visibility: 'public')
      get '/api/v1/posts', headers: headers
      
      expect(response).to have_http_status(:success)
      expect(JSON.parse(response.body).size).to eq(3)
    end
  end
  
  describe 'POST /api/v1/posts' do
    it 'creates a new post' do
      post_params = { post: { content: 'Test post', visibility: 'public' } }
      
      expect {
        post '/api/v1/posts', params: post_params, headers: headers
      }.to change(Post, :count).by(1)
      
      expect(response).to have_http_status(:created)
    end
  end
end
```

## Serializers

```ruby
# app/serializers/user_serializer.rb
class UserSerializer < ActiveModel::Serializer
  attributes :id, :username, :full_name, :bio, :avatar_url, :followers_count, :following_count
  
  def avatar_url
    object.avatar.attached? ? Rails.application.routes.url_helpers.rails_blob_url(object.avatar, only_path: true) : nil
  end
  
  def followers_count
    object.followers.count
  end
  
  def following_count
    object.following.count
  end
end

# app/serializers/post_serializer.rb
class PostSerializer < ActiveModel::Serializer
  attributes :id, :content, :visibility, :likes_count, :comments_count, :created_at, :media_urls
  belongs_to :user
  
  def media_urls
    object.media.map { |file| Rails.application.routes.url_helpers.rails_blob_url(file, only_path: true) }
  end
end
```

## Performance Optimization

```ruby
# config/initializers/redis.rb
$redis = Redis.new(url: ENV['REDIS_URL'] || 'redis://localhost:6379/1')

# app/models/concerns/cacheable.rb
module Cacheable
  extend ActiveSupport::Concern
  
  def cache_key_with_version
    "#{super}-#{cache_version}"
  end
  
  def cache_version
    updated_at.to_i
  end
end

# Feed caching
class FeedService
  def self.get_user_feed(user, page = 1)
    cache_key = "user_feed:#{user.id}:page:#{page}"
    
    Rails.cache.fetch(cache_key, expires_in: 5.minutes) do
      Post.for_user_feed(user)
          .includes(:user, :comments)
          .order(published_at: :desc)
          .page(page)
          .map { |post| PostSerializer.new(post).as_json }
    end
  end
end
```

## Success Criteria

- [ ] Complete social media API functionality
- [ ] Real-time features with Action Cable working
- [ ] 90%+ test coverage with RSpec
- [ ] JWT authentication implemented
- [ ] Feed algorithm optimized
- [ ] Production deployment with WebSocket support
- [ ] Comprehensive API documentation

---

📋 **[Back to Projects Overview](../Projects/Ruby_Rails_Backend/)** | 🏠 **[Back to Training Plan](../Intensive_Plans/Training_Plan_Ruby_Rails_Backend_Intensive.html)** | 🏠 **[Back to README](../README.html)**

### Related Projects
- **[Project 1: E-commerce Platform](./Project_1_Rails_E_Commerce.html)** - Full Rails application with Stripe
- **[Project 3: Microservices Platform](./Project_3_Microservices_Platform.html)** - Rails microservices architecture
- **[Project 4: Production System](./Project_4_Production_System.html)** - Full DevOps pipeline

