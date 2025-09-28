# Project 1: E-commerce Platform
*Full Stack Intensive Plan - Weeks 1-2*

📋 **[Back to Training Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html)** | 🏠 **[Back to README](../../README.html)**

## 🎯 Project Overview

**Duration**: 2 weeks (Weeks 1-2)  
**Tech Stack**: React + TypeScript + Rails API + PostgreSQL + Redis  
**Focus**: Full-stack development, frontend-backend integration, and modern web applications  

### Learning Objectives
- Master React with TypeScript for modern frontend development
- Build Rails API backend with authentication
- Implement frontend-backend communication
- Handle state management and data flow
- Create responsive and accessible user interfaces

---

## 📊 Database Schema Design (Rails)

### Users Table
```ruby
class CreateUsers < ActiveRecord::Migration[7.0]
  def change
    create_table :users do |t|
      t.string :email, null: false, index: { unique: true }
      t.string :password_digest, null: false
      t.string :first_name, null: false
      t.string :last_name, null: false
      t.string :phone_number
      t.text :shipping_address
      t.text :billing_address
      t.boolean :email_verified, default: false
      t.boolean :is_active, default: true
      t.datetime :last_login_at
      t.timestamps
    end
  end
end
```

### Products Table
```ruby
class CreateProducts < ActiveRecord::Migration[7.0]
  def change
    create_table :products do |t|
      t.string :name, null: false
      t.string :slug, null: false, index: { unique: true }
      t.text :description
      t.decimal :price, precision: 10, scale: 2, null: false
      t.decimal :compare_price, precision: 10, scale: 2
      t.integer :stock_quantity, default: 0
      t.string :sku, null: false, index: { unique: true }
      t.references :category, null: false, foreign_key: true
      t.boolean :is_active, default: true
      t.boolean :is_featured, default: false
      t.string :image_url
      t.text :tags, array: true, default: []
      t.timestamps
    end

    add_index :products, [:category_id, :is_active]
    add_index :products, [:price, :is_active]
    add_index :products, :tags, using: 'gin'
  end
end
```

### Orders Table
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
      t.decimal :total_amount, precision: 10, scale: 2, null: false
      t.json :shipping_address, null: false
      t.json :billing_address, null: false
      t.string :payment_method
      t.string :payment_status, default: 'pending'
      t.timestamps
    end

    add_index :orders, [:user_id, :created_at]
    add_index :orders, [:status, :created_at]
  end
end
```

---

## 🛠️ Frontend Components (React + TypeScript)

### Product List Component
```typescript
// components/ProductList.tsx
import React, { useState, useEffect } from 'react';
import { Product } from '../types/Product';

interface ProductListProps {
  categoryId?: string;
  searchQuery?: string;
}

const ProductList: React.FC<ProductListProps> = ({ categoryId, searchQuery }) => {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [filters, setFilters] = useState({
    minPrice: '',
    maxPrice: '',
    sortBy: 'created_at',
    sortOrder: 'desc'
  });

  useEffect(() => {
    fetchProducts();
  }, [categoryId, searchQuery, filters]);

  const fetchProducts = async () => {
    try {
      setLoading(true);
      const params = new URLSearchParams();
      
      if (categoryId) params.append('category_id', categoryId);
      if (searchQuery) params.append('search', searchQuery);
      if (filters.minPrice) params.append('min_price', filters.minPrice);
      if (filters.maxPrice) params.append('max_price', filters.maxPrice);
      params.append('sort_by', filters.sortBy);
      params.append('sort_order', filters.sortOrder);

      const response = await fetch(`/api/v1/products?${params}`);
      
      if (!response.ok) {
        throw new Error('Failed to fetch products');
      }

      const data = await response.json();
      setProducts(data.products);
    } catch (err) {
      setError(err instanceof Error ? err.message : 'An error occurred');
    } finally {
      setLoading(false);
    }
  };

  const handleAddToCart = async (productId: string) => {
    try {
      const response = await fetch('/api/v1/cart/items', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify({ product_id: productId, quantity: 1 })
      });

      if (response.ok) {
        // Show success message or update cart state
        console.log('Product added to cart');
      }
    } catch (error) {
      console.error('Error adding to cart:', error);
    }
  };

  if (loading) return <div className="loading">Loading products...</div>;
  if (error) return <div className="error">Error: {error}</div>;

  return (
    <div className="product-list">
      <div className="filters">
        <input
          type="number"
          placeholder="Min Price"
          value={filters.minPrice}
          onChange={(e) => setFilters({...filters, minPrice: e.target.value})}
        />
        <input
          type="number"
          placeholder="Max Price"
          value={filters.maxPrice}
          onChange={(e) => setFilters({...filters, maxPrice: e.target.value})}
        />
        <select
          value={filters.sortBy}
          onChange={(e) => setFilters({...filters, sortBy: e.target.value})}
        >
          <option value="created_at">Newest</option>
          <option value="price">Price</option>
          <option value="name">Name</option>
        </select>
      </div>

      <div className="products-grid">
        {products.map((product) => (
          <div key={product.id} className="product-card">
            <img src={product.image_url} alt={product.name} />
            <h3>{product.name}</h3>
            <p className="price">
              ${product.price}
              {product.compare_price && (
                <span className="compare-price">${product.compare_price}</span>
              )}
            </p>
            <button 
              onClick={() => handleAddToCart(product.id)}
              disabled={product.stock_quantity === 0}
            >
              {product.stock_quantity === 0 ? 'Out of Stock' : 'Add to Cart'}
            </button>
          </div>
        ))}
      </div>
    </div>
  );
};

export default ProductList;
```

### Shopping Cart Component
```typescript
// components/ShoppingCart.tsx
import React, { useState, useEffect } from 'react';
import { CartItem } from '../types/Cart';

const ShoppingCart: React.FC = () => {
  const [cartItems, setCartItems] = useState<CartItem[]>([]);
  const [loading, setLoading] = useState(true);
  const [isOpen, setIsOpen] = useState(false);

  useEffect(() => {
    fetchCartItems();
  }, []);

  const fetchCartItems = async () => {
    try {
      const response = await fetch('/api/v1/cart', {
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        const data = await response.json();
        setCartItems(data.items);
      }
    } catch (error) {
      console.error('Error fetching cart:', error);
    } finally {
      setLoading(false);
    }
  };

  const updateQuantity = async (itemId: string, quantity: number) => {
    try {
      const response = await fetch(`/api/v1/cart/items/${itemId}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        },
        body: JSON.stringify({ quantity })
      });

      if (response.ok) {
        setCartItems(items => 
          items.map(item => 
            item.id === itemId ? { ...item, quantity } : item
          )
        );
      }
    } catch (error) {
      console.error('Error updating quantity:', error);
    }
  };

  const removeItem = async (itemId: string) => {
    try {
      const response = await fetch(`/api/v1/cart/items/${itemId}`, {
        method: 'DELETE',
        headers: {
          'Authorization': `Bearer ${localStorage.getItem('token')}`
        }
      });

      if (response.ok) {
        setCartItems(items => items.filter(item => item.id !== itemId));
      }
    } catch (error) {
      console.error('Error removing item:', error);
    }
  };

  const calculateTotal = () => {
    return cartItems.reduce((total, item) => total + (item.unit_price * item.quantity), 0);
  };

  if (loading) return <div>Loading cart...</div>;

  return (
    <div className={`shopping-cart ${isOpen ? 'open' : ''}`}>
      <button 
        className="cart-toggle"
        onClick={() => setIsOpen(!isOpen)}
      >
        Cart ({cartItems.length})
      </button>

      <div className="cart-content">
        <h2>Shopping Cart</h2>
        
        {cartItems.length === 0 ? (
          <p>Your cart is empty</p>
        ) : (
          <>
            {cartItems.map((item) => (
              <div key={item.id} className="cart-item">
                <img src={item.product.image_url} alt={item.product.name} />
                <div className="item-details">
                  <h4>{item.product.name}</h4>
                  <p>${item.unit_price}</p>
                  <div className="quantity-controls">
                    <button 
                      onClick={() => updateQuantity(item.id, item.quantity - 1)}
                      disabled={item.quantity <= 1}
                    >
                      -
                    </button>
                    <span>{item.quantity}</span>
                    <button 
                      onClick={() => updateQuantity(item.id, item.quantity + 1)}
                      disabled={item.quantity >= item.product.stock_quantity}
                    >
                      +
                    </button>
                  </div>
                  <button 
                    onClick={() => removeItem(item.id)}
                    className="remove-item"
                  >
                    Remove
                  </button>
                </div>
              </div>
            ))}
            
            <div className="cart-summary">
              <h3>Total: ${calculateTotal().toFixed(2)}</h3>
              <button className="checkout-btn">
                Proceed to Checkout
              </button>
            </div>
          </>
        )}
      </div>
    </div>
  );
};

export default ShoppingCart;
```

---

## 🛠️ Backend API (Rails)

### Products Controller
```ruby
# app/controllers/api/v1/products_controller.rb
class Api::V1::ProductsController < ApplicationController
  before_action :authenticate_user!, except: [:index, :show]
  before_action :set_product, only: [:show, :update, :destroy]

  def index
    @products = Product.active.includes(:category)
                      .page(params[:page])
                      .per(params[:per_page] || 20)

    # Apply filters
    @products = @products.by_category(params[:category_id]) if params[:category_id]
    @products = @products.price_range(params[:min_price], params[:max_price]) if params[:min_price] && params[:max_price]
    @products = @products.search(params[:search]) if params[:search]

    # Apply sorting
    case params[:sort_by]
    when 'price'
      @products = @products.order(price: params[:sort_order] || :asc)
    when 'name'
      @products = @products.order(name: params[:sort_order] || :asc)
    else
      @products = @products.order(created_at: params[:sort_order] || :desc)
    end

    render json: {
      products: @products.map { |product| product_response(product) },
      pagination: pagination_response(@products)
    }
  end

  def show
    @product.increment!(:views_count)
    
    render json: {
      product: detailed_product_response(@product)
    }
  end

  private

  def set_product
    @product = Product.find(params[:id])
  end

  def product_response(product)
    {
      id: product.id,
      name: product.name,
      slug: product.slug,
      price: product.price,
      compare_price: product.compare_price,
      stock_quantity: product.stock_quantity,
      image_url: product.image_url,
      category: {
        id: product.category.id,
        name: product.category.name
      },
      created_at: product.created_at
    }
  end

  def detailed_product_response(product)
    product_response(product).merge({
      description: product.description,
      sku: product.sku,
      tags: product.tags,
      views_count: product.views_count
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

### Cart Controller
```ruby
# app/controllers/api/v1/cart_controller.rb
class Api::V1::CartController < ApplicationController
  before_action :authenticate_user!

  def show
    @cart_items = current_user.cart_items.includes(:product)
    
    render json: {
      items: @cart_items.map { |item| cart_item_response(item) },
      summary: {
        item_count: @cart_items.sum(:quantity),
        subtotal: @cart_items.sum(:total_price),
        tax: calculate_tax(@cart_items.sum(:total_price)),
        total: calculate_total(@cart_items.sum(:total_price))
      }
    }
  end

  def add_item
    @product = Product.find(params[:product_id])
    @cart_item = current_user.cart_items.find_by(product: @product)

    if @cart_item
      @cart_item.increment!(:quantity)
    else
      @cart_item = current_user.cart_items.create!(
        product: @product,
        quantity: 1,
        unit_price: @product.price,
        total_price: @product.price
      )
    end

    render json: {
      item: cart_item_response(@cart_item)
    }
  end

  def update_item
    @cart_item = current_user.cart_items.find(params[:id])
    @cart_item.update!(
      quantity: params[:quantity],
      total_price: @cart_item.unit_price * params[:quantity]
    )

    render json: {
      item: cart_item_response(@cart_item)
    }
  end

  def remove_item
    @cart_item = current_user.cart_items.find(params[:id])
    @cart_item.destroy!

    render json: { message: 'Item removed from cart' }
  end

  private

  def cart_item_response(item)
    {
      id: item.id,
      product: {
        id: item.product.id,
        name: item.product.name,
        image_url: item.product.image_url,
        stock_quantity: item.product.stock_quantity
      },
      quantity: item.quantity,
      unit_price: item.unit_price,
      total_price: item.total_price,
      created_at: item.created_at
    }
  end

  def calculate_tax(subtotal)
    # Simple 8.5% tax calculation
    (subtotal * 0.085).round(2)
  end

  def calculate_total(subtotal)
    subtotal + calculate_tax(subtotal)
  end
end
```

---

## 🧪 Testing Requirements

### Frontend Testing (Jest + React Testing Library)
```typescript
// __tests__/ProductList.test.tsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import ProductList from '../components/ProductList';

// Mock fetch
global.fetch = jest.fn();

describe('ProductList', () => {
  beforeEach(() => {
    (fetch as jest.Mock).mockClear();
  });

  test('renders product list', async () => {
    const mockProducts = [
      {
        id: '1',
        name: 'Test Product',
        price: 99.99,
        image_url: 'test.jpg',
        stock_quantity: 10
      }
    ];

    (fetch as jest.Mock).mockResolvedValueOnce({
      ok: true,
      json: async () => ({ products: mockProducts })
    });

    render(<ProductList />);

    await waitFor(() => {
      expect(screen.getByText('Test Product')).toBeInTheDocument();
      expect(screen.getByText('$99.99')).toBeInTheDocument();
    });
  });

  test('handles add to cart', async () => {
    const mockProducts = [
      {
        id: '1',
        name: 'Test Product',
        price: 99.99,
        image_url: 'test.jpg',
        stock_quantity: 10
      }
    ];

    (fetch as jest.Mock)
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({ products: mockProducts })
      })
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({})
      });

    // Mock localStorage
    Object.defineProperty(window, 'localStorage', {
      value: {
        getItem: jest.fn(() => 'mock-token')
      }
    });

    render(<ProductList />);

    await waitFor(() => {
      expect(screen.getByText('Add to Cart')).toBeInTheDocument();
    });

    fireEvent.click(screen.getByText('Add to Cart'));

    expect(fetch).toHaveBeenCalledWith('/api/v1/cart/items', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer mock-token'
      },
      body: JSON.stringify({ product_id: '1', quantity: 1 })
    });
  });
});
```

### Backend Testing (RSpec)
```ruby
# spec/requests/api/v1/products_spec.rb
require 'rails_helper'

RSpec.describe 'Api::V1::Products', type: :request do
  let(:user) { create(:user) }
  let(:category) { create(:category) }
  let(:products) { create_list(:product, 3, category: category) }

  describe 'GET /api/v1/products' do
    it 'returns products list' do
      products

      get '/api/v1/products'

      expect(response).to have_http_status(:ok)
      expect(json_response['products']).to be_present
      expect(json_response['products'].length).to eq(3)
    end

    it 'filters by category' do
      products
      other_category = create(:category)
      other_product = create(:product, category: other_category)

      get '/api/v1/products', params: { category_id: category.id }

      expect(response).to have_http_status(:ok)
      expect(json_response['products'].length).to eq(3)
      expect(json_response['products'].all? { |p| p['category']['id'] == category.id }).to be true
    end
  end

  describe 'GET /api/v1/products/:id' do
    it 'returns product details' do
      product = products.first

      get "/api/v1/products/#{product.id}"

      expect(response).to have_http_status(:ok)
      expect(json_response['product']['id']).to eq(product.id)
      expect(json_response['product']['name']).to eq(product.name)
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

### Frontend Deployment (Vercel/Netlify)
```json
// package.json
{
  "name": "ecommerce-frontend",
  "version": "1.0.0",
  "scripts": {
    "build": "react-scripts build",
    "start": "react-scripts start",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.8.0",
    "axios": "^1.3.0",
    "@types/react": "^18.0.0",
    "@types/react-dom": "^18.0.0",
    "typescript": "^4.9.0"
  }
}
```

### Backend Deployment (Heroku/Railway)
```ruby
# Gemfile
source 'https://rubygems.org'
ruby '3.2.0'

gem 'rails', '~> 7.0'
gem 'pg', '~> 1.1'
gem 'redis', '~> 4.0'
gem 'puma', '~> 5.0'
gem 'rack-cors'
gem 'jwt'
gem 'bcrypt'
gem 'kaminari'

group :development, :test do
  gem 'rspec-rails'
  gem 'factory_bot_rails'
  gem 'faker'
end

group :production do
  gem 'rails_12factor'
end
```

---

## 📚 Learning Resources

### Essential Reading
- [React Documentation](https://reactjs.org/docs/) - Frontend framework
- [TypeScript Handbook](https://www.typescriptlang.org/docs/) - Type safety
- [Rails Guides](https://guides.rubyonrails.org/) - Backend framework
- [React Testing Library](https://testing-library.com/docs/react-testing-library/intro/) - Frontend testing

### Udemy Courses (Focus Sections)
- **React Complete Guide** - Sections 1-8 (Components, Hooks, State management)
- **TypeScript Fundamentals** - Sections 1-6 (Types, Interfaces, Generics)
- **Rails API Development** - Sections 2-6 (JSON APIs, Authentication)

### Practice Projects
- Build a simple todo app with React and Rails
- Create a basic blog with comments
- Implement user authentication flow

---

## ✅ Success Criteria

### Week 1 Goals
- [ ] Set up React frontend with TypeScript
- [ ] Set up Rails API backend
- [ ] Implement user authentication
- [ ] Create product listing and detail pages
- [ ] Build basic shopping cart functionality

### Week 2 Goals
- [ ] Implement checkout process
- [ ] Add order management
- [ ] Create responsive design
- [ ] Add comprehensive testing
- [ ] Deploy frontend and backend
- [ ] Achieve 80%+ test coverage

### Final Deliverables
- [ ] Fully functional e-commerce platform
- [ ] Responsive frontend with TypeScript
- [ ] RESTful Rails API backend
- [ ] User authentication and authorization
- [ ] Shopping cart and checkout process
- [ ] 80%+ test coverage on both frontend and backend
- [ ] Deployed applications with live demos

---

## 🔗 Related Projects
- **Next Project**: [Project 2: Social Media Dashboard](../Full_Stack/Project_2_Social_Media_Dashboard.html)
- **Training Plan**: [Full Stack Intensive Plan](../../Intensive_Plans/Training_Plan_Full_Stack_Intensive.html)
- **Main README**: [Repository Overview](../../README.html)
