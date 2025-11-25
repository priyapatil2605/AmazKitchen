# AmazKitchen - Food Delivery Platform

A full-stack food delivery application built with React, TypeScript, Supabase, and Razorpay payment integration. This platform enables users to browse menus, place orders, track deliveries, leave reviews, and manage their diet plans with expert nutritionists.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Technology Stack](#technology-stack)
- [Key Features](#key-features)
- [Architecture](#architecture)
- [Database Schema](#database-schema)
- [Frontend Structure](#frontend-structure)
- [Backend (Supabase)](#backend-supabase)
- [Authentication System](#authentication-system)
- [Payment Integration](#payment-integration)
- [Setup & Installation](#setup--installation)
- [Environment Variables](#environment-variables)
- [API Documentation](#api-documentation)
- [Complete SQL Schema](#complete-sql-schema)
- [Security & RLS Policies](#security--rls-policies)
- [Deployment](#deployment)

---

## 🎯 Overview

**AmazKitchen** is a modern food delivery platform that connects customers with expert chefs and nutritionists. The application features:

- **Customer Portal**: Browse menu, place orders, track deliveries, manage profile
- **Menu Management**: Categorized food items with nutritional information
- **Order System**: Real-time order tracking with status updates
- **Payment Gateway**: Secure Razorpay integration for online payments
- **Review System**: Customers can rate and review menu items
- **Diet Planning**: Integration with nutritionists for personalized meal plans
- **Chef Profiles**: Transparent information about chefs and their specialties

---

## 🛠 Technology Stack

### Frontend
- **React 18.3.1** - UI library
- **TypeScript** - Type safety
- **Vite** - Build tool and dev server
- **React Router DOM 6.30.1** - Routing
- **Tailwind CSS** - Utility-first CSS framework
- **shadcn/ui** - Re-usable component library
- **Framer Motion 12.23.24** - Animation library
- **TanStack Query 5.83.0** - Data fetching and caching
- **Lucide React** - Icon library
- **React Hook Form 7.61.1** - Form management
- **Zod 3.25.76** - Schema validation
- **Sonner** - Toast notifications

### Backend
- **Supabase** - Backend as a Service (BaaS)
  - PostgreSQL Database
  - Authentication (JWT-based)
  - Row Level Security (RLS)
  - Edge Functions (Deno runtime)
  - Real-time subscriptions
- **Deno** - Edge function runtime

### Payment Gateway
- **Razorpay** - Payment processing

### Deployment
- **Lovable** - Hosting platform
- **Supabase Cloud** - Database and auth hosting

---

## ✨ Key Features

### User Features
1. **Authentication**
   - Email/password signup and login
   - Automatic profile creation on signup
   - Session management with JWT tokens
   - Secure password reset

2. **Menu Browsing**
   - Browse by categories (appetizer, main course, dessert, beverage, salad, soup)
   - View nutritional information (calories)
   - Filter by availability
   - Chef information for each dish
   - High-quality food images

3. **Shopping Cart**
   - Add/remove items
   - Update quantities
   - Local storage persistence
   - Real-time price calculation
   - Stock availability check

4. **Order Management**
   - Place orders with delivery address
   - Real-time order tracking
   - Order status updates (pending → confirmed → preparing → ready → out for delivery → delivered)
   - Delivery time estimates
   - Order history

5. **Payment Processing**
   - Multiple payment modes (card, UPI, wallet, COD, online)
   - Secure Razorpay integration
   - Payment verification
   - Transaction history
   - Payment status tracking

6. **Review System**
   - Rate menu items (1-5 stars)
   - Write detailed reviews
   - Edit/delete own reviews
   - View all reviews for items

7. **Diet Planning**
   - Link with nutritionists
   - Personalized meal plans
   - Macro tracking (protein, carbs, fat)
   - Calorie monitoring

### Admin/Chef Features
- Chef profiles with specialties
- Experience and availability information
- Menu item management

### System Features
- **Automatic Stock Management**: Inventory updated on order placement/cancellation
- **Real-time Updates**: Order status changes reflected immediately
- **Security**: Row Level Security policies for data protection
- **PII Protection**: Sensitive data hidden from public views
- **Responsive Design**: Mobile-first approach

---

## 🏗 Architecture

### System Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT (React App)                      │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Home       │  │    Menu      │  │    Cart      │     │
│  │   Page       │  │    Page      │  │    Page      │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Orders     │  │  Dashboard   │  │ Track Order  │     │
│  │   Page       │  │    Page      │  │    Page      │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │   Auth       │  │   Not Found  │                        │
│  │   Page       │  │    Page      │                        │
│  └──────────────┘  └──────────────┘                        │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ HTTP/WebSocket
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                  SUPABASE BACKEND                           │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │            PostgreSQL Database                     │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐          │   │
│  │  │customers │ │menu_items│ │  orders  │          │   │
│  │  └──────────┘ └──────────┘ └──────────┘          │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐          │   │
│  │  │  chefs   │ │nutritio..│ │ payments │          │   │
│  │  └──────────┘ └──────────┘ └──────────┘          │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐          │   │
│  │  │order_item│ │diet_plans│ │ reviews  │          │   │
│  │  └──────────┘ └──────────┘ └──────────┘          │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │         Authentication (auth.users)                │   │
│  │  - JWT token management                            │   │
│  │  - Email verification                              │   │
│  │  - Session handling                                │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │            Edge Functions (Deno)                   │   │
│  │  ┌──────────────────────────────────────────┐     │   │
│  │  │  create-razorpay-order                   │     │   │
│  │  │  - Validates cart items                  │     │   │
│  │  │  - Checks stock availability             │     │   │
│  │  │  - Creates Razorpay order                │     │   │
│  │  └──────────────────────────────────────────┘     │   │
│  │  ┌──────────────────────────────────────────┐     │   │
│  │  │  verify-razorpay-payment                 │     │   │
│  │  │  - Verifies payment signature            │     │   │
│  │  │  - Updates payment status                │     │   │
│  │  └──────────────────────────────────────────┘     │   │
│  └────────────────────────────────────────────────────┘   │
│                                                             │
│  ┌────────────────────────────────────────────────────┐   │
│  │      Row Level Security (RLS) Policies             │   │
│  │  - User can only view/edit own data               │   │
│  │  - Public data accessible to all                  │   │
│  │  - PII protection for sensitive info              │   │
│  └────────────────────────────────────────────────────┘   │
└─────────────────────┬───────────────────────────────────────┘
                      │
                      │ API Calls
                      │
┌─────────────────────▼───────────────────────────────────────┐
│                 RAZORPAY GATEWAY                            │
│  - Payment order creation                                   │
│  - Payment verification                                     │
│  - Transaction management                                   │
└─────────────────────────────────────────────────────────────┘
```

### Data Flow

1. **User Registration**:
   ```
   User → Auth Page → Supabase Auth → Database Trigger → Create Customer Profile
   ```

2. **Order Placement**:
   ```
   User → Cart → Create Order → Edge Function → Validate Stock → Razorpay Order → Payment → Update Database
   ```

3. **Real-time Updates**:
   ```
   Database Change → Realtime Subscription → Client Update → UI Refresh
   ```

---

## 🗄 Database Schema

### Entity Relationship Diagram

```
┌─────────────────┐
│   auth.users    │
│   (Supabase)    │
└────────┬────────┘
         │ 1
         │ has
         │ 1
┌────────▼────────┐      ┌─────────────────┐
│   customers     │──────│  nutritionists  │
│  - customer_id  │  n:1 │ - nutritionist_id│
│  - auth_user_id │      │ - name          │
│  - name         │      │ - qualification │
│  - email        │      └─────────────────┘
│  - phone        │
│  - address      │      ┌─────────────────┐
│  - diet_type    │──────│   diet_plans    │
└────────┬────────┘ 1:n  │ - plan_id       │
         │               │ - customer_id   │
         │ 1             │ - meal_type     │
         │ has           │ - calories      │
         │ n             │ - protein       │
┌────────▼────────┐      │ - carbs         │
│     orders      │      │ - fat           │
│  - order_id     │      └─────────────────┘
│  - customer_id  │
│  - total_price  │      ┌─────────────────┐
│  - status       │──────│    payments     │
│  - payment_status│ 1:n │ - payment_id    │
│  - delivery_addr│      │ - order_id      │
└────────┬────────┘      │ - amount        │
         │ 1             │ - payment_mode  │
         │ contains      │ - transaction_id│
         │ n             └─────────────────┘
┌────────▼────────┐
│  order_items    │      ┌─────────────────┐
│ - order_item_id │──────│   menu_items    │
│ - order_id      │  n:1 │ - item_id       │
│ - item_id       │      │ - item_name     │
│ - quantity      │      │ - category      │
│ - subtotal      │      │ - price         │
└─────────────────┘      │ - chef_id       │
                         │ - stock_quantity│
┌─────────────────┐      │ - calories      │
│    reviews      │──────│ - is_available  │
│ - review_id     │  n:1 └────────┬────────┘
│ - customer_id   │               │ n:1
│ - item_id       │               │
│ - rating        │      ┌────────▼────────┐
│ - comment       │      │     chefs       │
└─────────────────┘      │ - chef_id       │
                         │ - name          │
                         │ - specialty     │
                         │ - experience    │
                         │ - availability  │
                         └─────────────────┘
```

### Core Tables

1. **customers** - User profiles
   - Links to auth.users via auth_user_id
   - Personal information (name, phone, address)
   - Diet preferences
   - Optional nutritionist assignment

2. **menu_items** - Food inventory
   - Item details (name, description, price)
   - Nutritional info (calories)
   - Chef assignment
   - Stock management
   - Availability status
   - Images

3. **orders** - Order management
   - Customer reference
   - Pricing and status
   - Payment tracking
   - Delivery information
   - Timestamps

4. **order_items** - Order line items
   - Links orders to menu items
   - Quantity and subtotal
   - Triggers stock updates

5. **payments** - Payment records
   - Transaction details
   - Payment modes and status
   - Provider information
   - Card/UPI details

6. **reviews** - Customer feedback
   - Rating (1-5 stars)
   - Comments
   - Links to customer and menu item

7. **chefs** - Chef profiles
   - Personal and professional info
   - Specialties
   - Experience and availability

8. **nutritionists** - Nutritionist profiles
   - Qualifications and specializations
   - Contact information

9. **diet_plans** - Personalized nutrition
   - Meal types
   - Macro breakdown
   - Nutritionist recommendations

### Enums

```sql
-- User diet preferences
diet_type: 'vegetarian' | 'vegan' | 'keto' | 'paleo' | 'mediterranean' | 'none'

-- Meal categories
meal_type: 'breakfast' | 'lunch' | 'dinner' | 'snack'

-- Menu categories
menu_category: 'appetizer' | 'main_course' | 'dessert' | 'beverage' | 'salad' | 'soup'

-- Order progression
order_status: 'pending' | 'confirmed' | 'preparing' | 'ready' | 'out_for_delivery' | 'delivered' | 'cancelled'

-- Payment tracking
payment_status: 'pending' | 'completed' | 'failed' | 'refunded'

-- Payment methods
payment_mode: 'cash' | 'card' | 'upi' | 'wallet' | 'online' | 'cod'

-- Delivery tracking
delivery_status: 'pending' | 'dispatched' | 'in_transit' | 'delivered'
```

---

## 📁 Frontend Structure

```
src/
├── components/
│   ├── ui/                    # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── form.tsx
│   │   ├── input.tsx
│   │   └── ... (40+ components)
│   ├── Navbar.tsx             # Main navigation
│   ├── NavLink.tsx            # Navigation link component
│   ├── PaymentModal.tsx       # Razorpay integration
│   └── ReviewsSection.tsx     # Product reviews
│
├── contexts/
│   ├── AuthContext.tsx        # Authentication state
│   └── CartContext.tsx        # Shopping cart state
│
├── hooks/
│   ├── use-mobile.tsx         # Mobile detection
│   └── use-toast.ts           # Toast notifications
│
├── integrations/
│   └── supabase/
│       ├── client.ts          # Supabase client config
│       └── types.ts           # Auto-generated types
│
├── lib/
│   └── utils.ts               # Utility functions
│
├── pages/
│   ├── Auth.tsx               # Login/Signup
│   ├── Cart.tsx               # Shopping cart
│   ├── Dashboard.tsx          # User dashboard
│   ├── Home.tsx               # Landing page
│   ├── Menu.tsx               # Menu browsing
│   ├── NotFound.tsx           # 404 page
│   ├── Orders.tsx             # Order history
│   └── TrackOrder.tsx         # Order tracking
│
├── App.tsx                    # Root component
├── main.tsx                   # Entry point
├── index.css                  # Global styles
└── vite-env.d.ts             # Type definitions
```

### Key Components

#### Authentication Context
```typescript
// Provides authentication state and methods throughout the app
interface AuthContextType {
  user: User | null;
  session: Session | null;
  signUp: (email, password, name) => Promise<{error}>;
  signIn: (email, password) => Promise<{error}>;
  signOut: () => Promise<void>;
  loading: boolean;
}
```

#### Cart Context
```typescript
// Manages shopping cart state
interface CartContextType {
  cart: CartItem[];
  addToCart: (item: CartItem) => void;
  removeFromCart: (item_id: string) => void;
  updateQuantity: (item_id: string, quantity: number) => void;
  clearCart: () => void;
  getTotalPrice: () => number;
  getTotalItems: () => number;
}
```

---

## 🔧 Backend (Supabase)

### Edge Functions

#### 1. create-razorpay-order

**Purpose**: Creates a Razorpay order after validating cart items and stock availability.

**Location**: `supabase/functions/create-razorpay-order/index.ts`

**Authentication**: Required (JWT verification enabled)

**Request**:
```typescript
POST /functions/v1/create-razorpay-order
Authorization: Bearer <jwt_token>

Body:
{
  items: [
    {
      item_id: string,
      quantity: number
    }
  ],
  delivery_address: string,
  notes?: string
}
```

**Process**:
1. Authenticates user from JWT
2. Validates request body with Zod schema
3. Fetches item prices from database
4. Validates stock availability
5. Calculates total amount
6. Creates Razorpay order (currently simulated)
7. Returns order details

**Response**:
```typescript
{
  id: string,              // Razorpay order ID
  entity: "order",
  amount: number,          // Amount in paise (₹1 = 100 paise)
  amount_paid: number,
  amount_due: number,
  currency: "INR",
  receipt: string,
  status: "created",
  attempts: number,
  notes: {},
  created_at: number       // Unix timestamp
}
```

#### 2. verify-razorpay-payment

**Purpose**: Verifies Razorpay payment signature and processes payment.

**Location**: `supabase/functions/verify-razorpay-payment/index.ts`

**Authentication**: Required (JWT verification enabled)

**Request**:
```typescript
POST /functions/v1/verify-razorpay-payment
Authorization: Bearer <jwt_token>

Body:
{
  razorpay_order_id: string,
  razorpay_payment_id: string,
  razorpay_signature: string
}
```

**Process**:
1. Authenticates user from JWT
2. Validates request body
3. Verifies payment signature using HMAC SHA256
4. Returns verification status

**Response**:
```typescript
{
  success: boolean,
  verified: boolean,
  payment_id: string,
  order_id: string,
  user_id: string
}
```

### Database Functions

#### 1. update_menu_stock()

**Purpose**: Automatically updates menu item stock when orders are placed or cancelled.

**Trigger**: AFTER INSERT OR DELETE on order_items

**Logic**:
- On INSERT: Decreases stock_quantity by order quantity
- On DELETE: Increases stock_quantity by order quantity

```sql
CREATE OR REPLACE FUNCTION update_menu_stock()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    UPDATE menu_items
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE item_id = NEW.item_id;
    RETURN NEW;
  ELSIF TG_OP = 'DELETE' THEN
    UPDATE menu_items
    SET stock_quantity = stock_quantity + OLD.quantity
    WHERE item_id = OLD.item_id;
    RETURN OLD;
  END IF;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

#### 2. handle_new_user()

**Purpose**: Automatically creates customer profile when user signs up.

**Trigger**: AFTER INSERT on auth.users

```sql
CREATE OR REPLACE FUNCTION handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO customers (auth_user_id, name, email)
  VALUES (
    NEW.id,
    COALESCE(NEW.raw_user_meta_data->>'name', 'User'),
    NEW.email
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;
```

### Database Views

#### 1. customer_order_summary

Aggregates customer order statistics.

```sql
CREATE VIEW customer_order_summary AS
SELECT 
  c.customer_id,
  c.name AS customer_name,
  c.email,
  c.phone,
  c.diet_type,
  COUNT(DISTINCT o.order_id) AS total_orders,
  COALESCE(SUM(o.total_price), 0) AS total_spent,
  MAX(o.order_date) AS last_order_date
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.email, c.phone, c.diet_type;
```

#### 2. chefs_public

Public view of chef information (PII protected).

```sql
CREATE VIEW chefs_public AS
SELECT 
  chef_id,
  name,
  specialty,
  experience_years,
  availability,
  created_at
FROM chefs;
```

#### 3. nutritionists_public

Public view of nutritionist information (PII protected).

```sql
CREATE VIEW nutritionists_public AS
SELECT 
  nutritionist_id,
  name,
  specialization,
  qualification,
  created_at
FROM nutritionists;
```

---

## 🔐 Authentication System

### Flow

1. **Sign Up**:
   ```
   User submits email, password, name
   → Supabase Auth creates user
   → Email verification sent
   → Trigger creates customer profile
   → JWT token issued
   → Redirect to home
   ```

2. **Sign In**:
   ```
   User submits email, password
   → Supabase Auth validates credentials
   → JWT token issued
   → Session stored in localStorage
   → Redirect to home
   ```

3. **Session Management**:
   - JWT tokens stored in localStorage
   - Auto-refresh enabled
   - Session persists across page reloads
   - Auth state listener updates UI

4. **Sign Out**:
   ```
   User clicks sign out
   → Supabase Auth destroys session
   → Clear localStorage
   → Redirect to home
   ```

### Protected Routes

Routes requiring authentication redirect to `/auth` if user is not logged in:
- `/cart` - Shopping cart
- `/orders` - Order history
- `/dashboard` - User dashboard
- `/track-order` - Order tracking

---

## 💳 Payment Integration

### Razorpay Flow

1. **Order Creation**:
   ```
   User clicks "Proceed to Pay"
   → Frontend validates cart
   → Call create-razorpay-order edge function
   → Edge function validates stock
   → Edge function creates Razorpay order
   → Returns order details to frontend
   ```

2. **Payment Processing**:
   ```
   Frontend receives Razorpay order
   → Open Razorpay modal
   → User completes payment
   → Razorpay callback with payment details
   → Frontend calls verify-razorpay-payment
   → Edge function verifies signature
   → Update order and payment status
   → Show success message
   ```

3. **Payment Verification**:
   - Uses HMAC SHA256 signature verification
   - Validates: order_id|payment_id with secret key
   - Prevents payment tampering
   - Ensures secure transactions

### Payment Modes Supported
- **Card** - Credit/Debit cards
- **UPI** - UPI IDs and apps
- **Wallet** - Digital wallets
- **Online** - Net banking
- **COD** - Cash on delivery

---

## 🚀 Setup & Installation

### Prerequisites
- Node.js 18+ and npm
- Supabase account
- Razorpay account (for payment integration)

### Local Development Setup

1. **Clone the repository**:
   ```bash
   git clone <repository-url>
   cd amazkitchen
   ```

2. **Install dependencies**:
   ```bash
   npm install
   ```

3. **Set up Supabase**:
   - Create a new Supabase project at https://supabase.com
   - Note your Project URL and Anon Key
   - Run all migrations from the [Complete SQL Schema](#complete-sql-schema) section

4. **Configure Razorpay**:
   - Create account at https://razorpay.com
   - Get API Key ID and Secret from Dashboard
   - Add to Supabase Edge Function secrets

5. **Environment variables** (for edge functions):
   ```bash
   # In Supabase Dashboard → Edge Functions → Settings
   RAZORPAY_KEY_ID=your_razorpay_key_id
   RAZORPAY_KEY_SECRET=your_razorpay_key_secret
   ```

6. **Update Supabase configuration**:
   ```typescript
   // src/integrations/supabase/client.ts
   const SUPABASE_URL = "your-project-url";
   const SUPABASE_PUBLISHABLE_KEY = "your-anon-key";
   ```

7. **Run development server**:
   ```bash
   npm run dev
   ```

8. **Access the application**:
   ```
   http://localhost:5173
   ```

### Deploying Edge Functions

```bash
# Install Supabase CLI
npm install -g supabase

# Login
supabase login

# Link project
supabase link --project-ref your-project-ref

# Deploy all functions
supabase functions deploy

# Or deploy specific function
supabase functions deploy create-razorpay-order
```

---

## 🔑 Environment Variables

### Frontend (embedded in code)
```typescript
SUPABASE_URL=https://homzxpzhhmgtlnnjmxop.supabase.co
SUPABASE_ANON_KEY=your_anon_key
```

### Backend (Supabase Secrets)
```bash
# Managed in Supabase Dashboard
SUPABASE_URL=your_project_url
SUPABASE_ANON_KEY=your_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
SUPABASE_DB_URL=your_db_connection_string
RAZORPAY_KEY_ID=your_razorpay_key_id
RAZORPAY_KEY_SECRET=your_razorpay_key_secret
```

---

## 📡 API Documentation

### Supabase Client Methods

#### Menu Items
```typescript
// Get all available menu items
const { data, error } = await supabase
  .from('menu_items')
  .select('*')
  .eq('is_available', true);

// Get item by category
const { data, error } = await supabase
  .from('menu_items')
  .select('*')
  .eq('category', 'main_course');
```

#### Orders
```typescript
// Create order
const { data, error } = await supabase
  .from('orders')
  .insert({
    customer_id: 'uuid',
    total_price: 100.00,
    delivery_address: 'Address',
    notes: 'Special instructions'
  });

// Get user orders
const { data, error } = await supabase
  .from('orders')
  .select(`
    *,
    order_items (
      *,
      menu_items (*)
    )
  `)
  .order('order_date', { ascending: false });
```

#### Reviews
```typescript
// Create review
const { data, error } = await supabase
  .from('reviews')
  .insert({
    customer_id: 'uuid',
    item_id: 'uuid',
    rating: 5,
    comment: 'Excellent!'
  });

// Get item reviews
const { data, error } = await supabase
  .from('reviews')
  .select(`
    *,
    customers (name)
  `)
  .eq('item_id', 'item_uuid');
```

### Edge Function Endpoints

#### Create Razorpay Order
```bash
curl -X POST \
  https://homzxpzhhmgtlnnjmxop.supabase.co/functions/v1/create-razorpay-order \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "items": [
      {"item_id": "uuid", "quantity": 2}
    ],
    "delivery_address": "123 Main St",
    "notes": "Ring doorbell"
  }'
```

#### Verify Payment
```bash
curl -X POST \
  https://homzxpzhhmgtlnnjmxop.supabase.co/functions/v1/verify-razorpay-payment \
  -H "Authorization: Bearer <jwt_token>" \
  -H "Content-Type: application/json" \
  -d '{
    "razorpay_order_id": "order_xxx",
    "razorpay_payment_id": "pay_xxx",
    "razorpay_signature": "signature_xxx"
  }'
```

---

## 🗃 Complete SQL Schema

### Migration 1: Initial Database Schema

```sql
-- Create enum types
CREATE TYPE diet_type AS ENUM ('vegetarian', 'vegan', 'keto', 'paleo', 'mediterranean', 'none');
CREATE TYPE meal_type AS ENUM ('breakfast', 'lunch', 'dinner', 'snack');
CREATE TYPE order_status AS ENUM ('pending', 'confirmed', 'preparing', 'ready', 'out_for_delivery', 'delivered', 'cancelled');
CREATE TYPE payment_status AS ENUM ('pending', 'completed', 'failed', 'refunded');
CREATE TYPE payment_mode AS ENUM ('cash', 'card', 'upi', 'wallet', 'online', 'cod');
CREATE TYPE delivery_status AS ENUM ('pending', 'dispatched', 'in_transit', 'delivered');
CREATE TYPE menu_category AS ENUM ('appetizer', 'main_course', 'dessert', 'beverage', 'salad', 'soup');

-- Nutritionists Table
CREATE TABLE public.nutritionists (
  nutritionist_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  qualification TEXT NOT NULL,
  specialization TEXT NOT NULL,
  phone TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  address TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Chefs Table
CREATE TABLE public.chefs (
  chef_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  age INTEGER CHECK (age >= 18),
  experience_years INTEGER CHECK (experience_years >= 0),
  specialty TEXT NOT NULL,
  address TEXT,
  phone TEXT NOT NULL,
  availability BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Customers Table
CREATE TABLE public.customers (
  customer_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  auth_user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  age INTEGER CHECK (age >= 0),
  gender TEXT,
  address TEXT,
  phone TEXT,
  email TEXT,
  nutritionist_id UUID REFERENCES public.nutritionists(nutritionist_id) ON DELETE SET NULL,
  diet_type diet_type DEFAULT 'none',
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now(),
  UNIQUE(auth_user_id)
);

-- Menu Items Table
CREATE TABLE public.menu_items (
  item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  item_name TEXT NOT NULL,
  category menu_category NOT NULL,
  calories INTEGER CHECK (calories >= 0),
  price DECIMAL(10, 2) NOT NULL CHECK (price >= 0),
  chef_id UUID REFERENCES public.chefs(chef_id) ON DELETE SET NULL,
  stock_quantity INTEGER DEFAULT 0 CHECK (stock_quantity >= 0),
  image_url TEXT,
  description TEXT,
  is_available BOOLEAN DEFAULT true,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Orders Table
CREATE TABLE public.orders (
  order_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES public.customers(customer_id) ON DELETE CASCADE NOT NULL,
  total_price DECIMAL(10, 2) NOT NULL CHECK (total_price >= 0),
  order_date TIMESTAMP WITH TIME ZONE DEFAULT now(),
  status order_status DEFAULT 'pending',
  payment_status payment_status DEFAULT 'pending',
  payment_mode payment_mode,
  delivery_status delivery_status DEFAULT 'pending',
  delivery_address TEXT,
  notes TEXT
);

-- Order Items Table
CREATE TABLE public.order_items (
  order_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES public.orders(order_id) ON DELETE CASCADE NOT NULL,
  item_id UUID REFERENCES public.menu_items(item_id) ON DELETE CASCADE NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0)
);

-- Diet Plan Table
CREATE TABLE public.diet_plans (
  plan_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id UUID REFERENCES public.customers(customer_id) ON DELETE CASCADE NOT NULL,
  nutritionist_id UUID REFERENCES public.nutritionists(nutritionist_id) ON DELETE SET NULL,
  meal_type meal_type NOT NULL,
  calories INTEGER CHECK (calories >= 0),
  protein DECIMAL(10, 2) CHECK (protein >= 0),
  carbs DECIMAL(10, 2) CHECK (carbs >= 0),
  fat DECIMAL(10, 2) CHECK (fat >= 0),
  remarks TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Enable Row Level Security
ALTER TABLE public.nutritionists ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.chefs ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.order_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.diet_plans ENABLE ROW LEVEL SECURITY;

-- RLS Policies for Menu Items
CREATE POLICY "Anyone can view menu items" ON public.menu_items FOR SELECT USING (true);

-- RLS Policies for Customers
CREATE POLICY "Users can view their own customer data" ON public.customers FOR SELECT USING (auth.uid() = auth_user_id);
CREATE POLICY "Users can update their own customer data" ON public.customers FOR UPDATE USING (auth.uid() = auth_user_id);
CREATE POLICY "Users can insert their own customer data" ON public.customers FOR INSERT WITH CHECK (auth.uid() = auth_user_id);

-- RLS Policies for Orders
CREATE POLICY "Users can view their own orders" ON public.orders FOR SELECT USING (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);
CREATE POLICY "Users can create their own orders" ON public.orders FOR INSERT WITH CHECK (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);

-- RLS Policies for Order Items
CREATE POLICY "Users can view their order items" ON public.order_items FOR SELECT USING (
  order_id IN (
    SELECT order_id FROM public.orders WHERE customer_id IN (
      SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid()
    )
  )
);
CREATE POLICY "Users can create order items" ON public.order_items FOR INSERT WITH CHECK (
  order_id IN (
    SELECT order_id FROM public.orders WHERE customer_id IN (
      SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid()
    )
  )
);

-- RLS Policies for Diet Plans
CREATE POLICY "Users can view their own diet plans" ON public.diet_plans FOR SELECT USING (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);

-- RLS Policies for Nutritionists
CREATE POLICY "Anyone can view nutritionists" ON public.nutritionists FOR SELECT USING (true);

-- RLS Policies for Chefs
CREATE POLICY "Anyone can view chefs" ON public.chefs FOR SELECT USING (true);

-- Stock Update Trigger
CREATE OR REPLACE FUNCTION update_menu_stock()
RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    UPDATE public.menu_items
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE item_id = NEW.item_id;
    RETURN NEW;
  ELSIF TG_OP = 'DELETE' THEN
    UPDATE public.menu_items
    SET stock_quantity = stock_quantity + OLD.quantity
    WHERE item_id = OLD.item_id;
    RETURN OLD;
  END IF;
  RETURN NULL;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER order_items_stock_trigger
AFTER INSERT OR DELETE ON public.order_items
FOR EACH ROW EXECUTE FUNCTION update_menu_stock();

-- Customer Order Summary View
CREATE OR REPLACE VIEW customer_order_summary AS
SELECT 
  c.customer_id,
  c.name AS customer_name,
  c.email,
  c.phone,
  c.diet_type,
  COUNT(DISTINCT o.order_id) AS total_orders,
  COALESCE(SUM(o.total_price), 0) AS total_spent,
  MAX(o.order_date) AS last_order_date
FROM public.customers c
LEFT JOIN public.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.email, c.phone, c.diet_type;

-- Auto-Create Customer Profile on Signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.customers (auth_user_id, name, email)
  VALUES (
    NEW.id,
    COALESCE(NEW.raw_user_meta_data->>'name', 'User'),
    NEW.email
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER SET search_path = public;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW EXECUTE FUNCTION public.handle_new_user();

-- Sample Data: Chefs
INSERT INTO public.chefs (name, age, experience_years, specialty, phone, availability) VALUES
('Marco Rossi', 45, 20, 'Italian Cuisine', '+1-555-0101', true),
('Mei Chen', 38, 15, 'Asian Fusion', '+1-555-0102', true),
('Pierre Dubois', 52, 25, 'French Pastry', '+1-555-0103', true),
('Carlos Martinez', 41, 18, 'Mexican Cuisine', '+1-555-0104', true);

-- Sample Data: Nutritionists
INSERT INTO public.nutritionists (name, qualification, specialization, phone, email) VALUES
('Dr. Sarah Johnson', 'PhD in Nutrition', 'Weight Management', '+1-555-0201', 'sarah.johnson@nutrition.com'),
('Dr. Raj Patel', 'MS in Clinical Nutrition', 'Sports Nutrition', '+1-555-0202', 'raj.patel@nutrition.com'),
('Dr. Emily White', 'RD, Certified Dietitian', 'Diabetes Care', '+1-555-0203', 'emily.white@nutrition.com');

-- Sample Data: Menu Items
INSERT INTO public.menu_items (item_name, category, calories, price, chef_id, stock_quantity, description, image_url) VALUES
('Margherita Pizza', 'main_course', 800, 14.99, (SELECT chef_id FROM public.chefs WHERE name = 'Marco Rossi' LIMIT 1), 50, 'Classic Italian pizza with fresh mozzarella and basil', 'https://images.unsplash.com/photo-1574071318508-1cdbab80d002'),
('Kung Pao Chicken', 'main_course', 650, 16.99, (SELECT chef_id FROM public.chefs WHERE name = 'Mei Chen' LIMIT 1), 40, 'Spicy Szechuan dish with peanuts and vegetables', 'https://images.unsplash.com/photo-1603073863670-58a6f88c9309'),
('Croissant', 'appetizer', 300, 4.99, (SELECT chef_id FROM public.chefs WHERE name = 'Pierre Dubois' LIMIT 1), 100, 'Buttery French pastry, baked fresh daily', 'https://images.unsplash.com/photo-1555507036-ab1f4038808a'),
('Tacos Al Pastor', 'main_course', 550, 12.99, (SELECT chef_id FROM public.chefs WHERE name = 'Carlos Martinez' LIMIT 1), 60, 'Traditional Mexican tacos with marinated pork', 'https://images.unsplash.com/photo-1565299585323-38d6b0865b47'),
('Caesar Salad', 'salad', 350, 9.99, (SELECT chef_id FROM public.chefs WHERE name = 'Marco Rossi' LIMIT 1), 80, 'Fresh romaine lettuce with parmesan and croutons', 'https://images.unsplash.com/photo-1546793665-c74683f339c1'),
('Chocolate Lava Cake', 'dessert', 450, 7.99, (SELECT chef_id FROM public.chefs WHERE name = 'Pierre Dubois' LIMIT 1), 30, 'Decadent chocolate cake with molten center', 'https://images.unsplash.com/photo-1563729784474-d77dbb933a9e'),
('Green Smoothie', 'beverage', 180, 6.99, (SELECT chef_id FROM public.chefs WHERE name = 'Mei Chen' LIMIT 1), 70, 'Healthy blend of spinach, banana, and mango', 'https://images.unsplash.com/photo-1610970881699-44a5587cabec'),
('Tom Yum Soup', 'soup', 280, 8.99, (SELECT chef_id FROM public.chefs WHERE name = 'Mei Chen' LIMIT 1), 45, 'Spicy and sour Thai soup with shrimp', 'https://images.unsplash.com/photo-1547592166-23ac45744acd');
```

### Migration 2: Payments Table

```sql
-- Payments Table
CREATE TABLE public.payments (
  payment_id UUID NOT NULL DEFAULT gen_random_uuid() PRIMARY KEY,
  order_id UUID NOT NULL REFERENCES public.orders(order_id) ON DELETE CASCADE,
  amount NUMERIC(10, 2) NOT NULL,
  payment_mode payment_mode NOT NULL,
  payment_status payment_status DEFAULT 'pending'::payment_status,
  transaction_id TEXT,
  payment_date TIMESTAMP WITH TIME ZONE DEFAULT now(),
  created_at TIMESTAMP WITH TIME ZONE DEFAULT now()
);

-- Enable RLS
ALTER TABLE public.payments ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Users can view their own payments"
ON public.payments
FOR SELECT
USING (
  order_id IN (
    SELECT order_id 
    FROM public.orders 
    WHERE customer_id IN (
      SELECT customer_id 
      FROM public.customers 
      WHERE auth_user_id = auth.uid()
    )
  )
);

CREATE POLICY "Users can create payments for their orders"
ON public.payments
FOR INSERT
WITH CHECK (
  order_id IN (
    SELECT order_id 
    FROM public.orders 
    WHERE customer_id IN (
      SELECT customer_id 
      FROM public.customers 
      WHERE auth_user_id = auth.uid()
    )
  )
);

-- Indexes
CREATE INDEX idx_payments_order_id ON public.payments(order_id);
CREATE INDEX idx_payments_status ON public.payments(payment_status);
```

### Migration 3: Delivery Tracking Fields

```sql
-- Add Delivery Tracking to Orders
ALTER TABLE public.orders 
ADD COLUMN IF NOT EXISTS estimated_delivery_time timestamp with time zone,
ADD COLUMN IF NOT EXISTS preparation_time integer DEFAULT 20,
ADD COLUMN IF NOT EXISTS actual_delivery_time timestamp with time zone;

-- Add Payment Details
ALTER TABLE public.payments
ADD COLUMN IF NOT EXISTS card_last_four text,
ADD COLUMN IF NOT EXISTS upi_id text,
ADD COLUMN IF NOT EXISTS payment_provider text;
```

### Migration 4: Reviews System

```sql
-- Reviews Table
CREATE TABLE IF NOT EXISTS public.reviews (
  review_id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  customer_id uuid NOT NULL,
  item_id uuid NOT NULL,
  rating integer NOT NULL CHECK (rating >= 1 AND rating <= 5),
  comment text,
  created_at timestamp with time zone DEFAULT now(),
  CONSTRAINT reviews_customer_id_fkey FOREIGN KEY (customer_id) REFERENCES public.customers(customer_id) ON DELETE CASCADE,
  CONSTRAINT reviews_item_id_fkey FOREIGN KEY (item_id) REFERENCES public.menu_items(item_id) ON DELETE CASCADE
);

-- Enable RLS
ALTER TABLE public.reviews ENABLE ROW LEVEL SECURITY;

-- RLS Policies
CREATE POLICY "Anyone can view reviews"
ON public.reviews
FOR SELECT
USING (true);

CREATE POLICY "Users can create reviews for their orders"
ON public.reviews
FOR INSERT
WITH CHECK (
  customer_id IN (
    SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can update their own reviews"
ON public.reviews
FOR UPDATE
USING (
  customer_id IN (
    SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid()
  )
);

CREATE POLICY "Users can delete their own reviews"
ON public.reviews
FOR DELETE
USING (
  customer_id IN (
    SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid()
  )
);

-- Indexes
CREATE INDEX IF NOT EXISTS idx_reviews_item_id ON public.reviews(item_id);
CREATE INDEX IF NOT EXISTS idx_reviews_customer_id ON public.reviews(customer_id);

-- Enable Realtime
ALTER PUBLICATION supabase_realtime ADD TABLE public.orders;
```

### Migration 5: Security Improvements - PII Protection

```sql
-- Drop Overly Permissive Policies
DROP POLICY IF EXISTS "Anyone can view chefs" ON public.chefs;
DROP POLICY IF EXISTS "Anyone can view nutritionists" ON public.nutritionists;

-- Create Public Views
CREATE OR REPLACE VIEW public.chefs_public AS
SELECT 
  chef_id,
  name,
  specialty,
  experience_years,
  availability,
  created_at
FROM public.chefs;

CREATE OR REPLACE VIEW public.nutritionists_public AS
SELECT 
  nutritionist_id,
  name,
  specialization,
  qualification,
  created_at
FROM public.nutritionists;

-- Grant Access to Views
GRANT SELECT ON public.chefs_public TO anon, authenticated;
GRANT SELECT ON public.nutritionists_public TO anon, authenticated;

-- Restricted Policies for Authenticated Users
CREATE POLICY "Authenticated users can view chef info"
ON public.chefs
FOR SELECT
TO authenticated
USING (true);

CREATE POLICY "Authenticated users can view nutritionist info"
ON public.nutritionists
FOR SELECT
TO authenticated
USING (true);

-- Fix customer_order_summary View
CREATE OR REPLACE VIEW customer_order_summary
WITH (security_invoker=true)
AS
SELECT 
  c.customer_id,
  c.name AS customer_name,
  c.email,
  c.phone,
  c.diet_type,
  COUNT(DISTINCT o.order_id) AS total_orders,
  COALESCE(SUM(o.total_price), 0) AS total_spent,
  MAX(o.order_date) AS last_order_date
FROM public.customers c
LEFT JOIN public.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.email, c.phone, c.diet_type;
```

### Migration 6: Critical RLS Fix

```sql
-- Make auth_user_id NOT NULL for security
ALTER TABLE public.customers 
ALTER COLUMN auth_user_id SET NOT NULL;
```

---

## 🔒 Security & RLS Policies

### Row Level Security (RLS) Overview

RLS ensures users can only access their own data. Every table has policies that enforce:

1. **SELECT**: What data users can read
2. **INSERT**: What data users can create
3. **UPDATE**: What data users can modify
4. **DELETE**: What data users can remove

### Key Security Principles

1. **User Data Isolation**:
   - Users can only see their own orders, payments, reviews
   - Implemented via `auth.uid() = auth_user_id` checks

2. **Public Data Access**:
   - Menu items visible to everyone
   - Chef/nutritionist public views hide PII

3. **Cascade Relationships**:
   - Deleting user cascades to customer profile
   - Deleting order cascades to order items and payments

4. **Function Security**:
   - Database functions use `SECURITY DEFINER`
   - Edge functions verify JWT tokens
   - Payment verification uses cryptographic signatures

### Policy Examples

#### Customer Data (Private)
```sql
-- Users can only view their own data
CREATE POLICY "Users can view their own customer data" 
ON customers FOR SELECT 
USING (auth.uid() = auth_user_id);
```

#### Menu Items (Public)
```sql
-- Anyone can view menu
CREATE POLICY "Anyone can view menu items" 
ON menu_items FOR SELECT 
USING (true);
```

#### Orders (Owner Only)
```sql
-- Users can only view their orders
CREATE POLICY "Users can view their own orders" 
ON orders FOR SELECT 
USING (
  customer_id IN (
    SELECT customer_id 
    FROM customers 
    WHERE auth_user_id = auth.uid()
  )
);
```

---

## 🚢 Deployment

### Frontend Deployment (Lovable)

1. **Automatic Deployment**:
   - Push to main branch
   - Lovable auto-builds and deploys
   - Live at: `https://lovable.dev/projects/<project-id>`

2. **Custom Domain** (optional):
   - Navigate to Project > Settings > Domains
   - Add custom domain
   - Configure DNS records

### Backend Deployment (Supabase)

1. **Database**:
   - Migrations auto-applied on push
   - Types regenerated automatically

2. **Edge Functions**:
   ```bash
   supabase functions deploy create-razorpay-order
   supabase functions deploy verify-razorpay-payment
   ```

3. **Environment Secrets**:
   - Set in Supabase Dashboard
   - Edge Functions > Settings

### Production Checklist

- [ ] Database migrations applied
- [ ] RLS policies tested
- [ ] Edge functions deployed
- [ ] Secrets configured
- [ ] Razorpay keys (live mode)
- [ ] Email templates configured
- [ ] Custom domain (if applicable)
- [ ] Error monitoring setup
- [ ] Performance testing completed

---

## 📊 Database Statistics

- **Total Tables**: 9 core tables + 3 views
- **Total Enum Types**: 7
- **Total Functions**: 2
- **Total Triggers**: 2
- **Total RLS Policies**: 15+
- **Total Indexes**: 4 custom indexes

---

## 🤝 Contributing

1. Fork the repository
2. Create feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open Pull Request

---

## 📄 License

This project is private and proprietary.

---

## 🆘 Support

For support, please contact the development team or open an issue in the repository.

---

## 📝 Additional Notes

### Testing Accounts

Create test accounts for different scenarios:
- Regular customer
- Customer with orders
- Customer with diet plan

### Development Tips

1. **Hot Reload**: Vite provides instant HMR
2. **Type Safety**: TypeScript catches errors at compile time
3. **Database Types**: Auto-generated from Supabase schema
4. **Local Testing**: Use Supabase local development

### Common Issues

1. **CORS Errors**: Ensure edge functions have CORS headers
2. **RLS Blocks Query**: Check policies match your use case
3. **Payment Verification Fails**: Verify Razorpay keys are correct
4. **Stock Not Updating**: Check trigger is active

---

## 🎓 Learning Resources

- [React Documentation](https://react.dev)
- [Supabase Documentation](https://supabase.com/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Razorpay Documentation](https://razorpay.com/docs)

---

**Built with ❤️ by the AmazKitchen Team**

Project URL: https://lovable.dev/projects/75939326-9569-402c-b237-be2ae2f6cf71
