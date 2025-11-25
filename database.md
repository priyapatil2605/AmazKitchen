# Database SQL Queries

This file contains all SQL queries used to create and configure the database for this project.

---

## Migration 1: Initial Database Schema
**Date**: 2025-11-07  
**File**: `20251107102945_3ca23b20-23a3-450e-aa73-25d131dbb623.sql`

### Enum Types
```sql
-- Create enum types
CREATE TYPE diet_type AS ENUM ('vegetarian', 'vegan', 'keto', 'paleo', 'mediterranean', 'none');
CREATE TYPE meal_type AS ENUM ('breakfast', 'lunch', 'dinner', 'snack');
CREATE TYPE order_status AS ENUM ('pending', 'confirmed', 'preparing', 'ready', 'out_for_delivery', 'delivered', 'cancelled');
CREATE TYPE payment_status AS ENUM ('pending', 'completed', 'failed', 'refunded');
CREATE TYPE payment_mode AS ENUM ('cash', 'card', 'upi', 'wallet');
CREATE TYPE delivery_status AS ENUM ('pending', 'dispatched', 'in_transit', 'delivered');
CREATE TYPE menu_category AS ENUM ('appetizer', 'main_course', 'dessert', 'beverage', 'salad', 'soup');
```

### Tables

#### Nutritionists Table
```sql
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
```

#### Chefs Table
```sql
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
```

#### Customers Table
```sql
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
```

#### Menu Items Table
```sql
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
```

#### Orders Table
```sql
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
```

#### Order Items Table
```sql
CREATE TABLE public.order_items (
  order_item_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id UUID REFERENCES public.orders(order_id) ON DELETE CASCADE NOT NULL,
  item_id UUID REFERENCES public.menu_items(item_id) ON DELETE CASCADE NOT NULL,
  quantity INTEGER NOT NULL CHECK (quantity > 0),
  subtotal DECIMAL(10, 2) NOT NULL CHECK (subtotal >= 0)
);
```

#### Diet Plan Table
```sql
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
```

### Row Level Security (RLS)

#### Enable RLS
```sql
ALTER TABLE public.nutritionists ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.chefs ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.customers ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.menu_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.orders ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.order_items ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.diet_plans ENABLE ROW LEVEL SECURITY;
```

#### RLS Policies for Menu Items
```sql
CREATE POLICY "Anyone can view menu items" ON public.menu_items FOR SELECT USING (true);
```

#### RLS Policies for Customers
```sql
CREATE POLICY "Users can view their own customer data" ON public.customers FOR SELECT USING (auth.uid() = auth_user_id);
CREATE POLICY "Users can update their own customer data" ON public.customers FOR UPDATE USING (auth.uid() = auth_user_id);
CREATE POLICY "Users can insert their own customer data" ON public.customers FOR INSERT WITH CHECK (auth.uid() = auth_user_id);
```

#### RLS Policies for Orders
```sql
CREATE POLICY "Users can view their own orders" ON public.orders FOR SELECT USING (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);
CREATE POLICY "Users can create their own orders" ON public.orders FOR INSERT WITH CHECK (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);
```

#### RLS Policies for Order Items
```sql
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
```

#### RLS Policies for Diet Plans
```sql
CREATE POLICY "Users can view their own diet plans" ON public.diet_plans FOR SELECT USING (
  customer_id IN (SELECT customer_id FROM public.customers WHERE auth_user_id = auth.uid())
);
```

#### RLS Policies for Nutritionists
```sql
CREATE POLICY "Anyone can view nutritionists" ON public.nutritionists FOR SELECT USING (true);
```

#### RLS Policies for Chefs
```sql
CREATE POLICY "Anyone can view chefs" ON public.chefs FOR SELECT USING (true);
```

### Functions and Triggers

#### Stock Update Trigger
```sql
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
```

#### Customer Order Summary View
```sql
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
```

#### Auto-Create Customer Profile on Signup
```sql
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
```

### Sample Data

#### Sample Chefs
```sql
INSERT INTO public.chefs (name, age, experience_years, specialty, phone, availability) VALUES
('Marco Rossi', 45, 20, 'Italian Cuisine', '+1-555-0101', true),
('Mei Chen', 38, 15, 'Asian Fusion', '+1-555-0102', true),
('Pierre Dubois', 52, 25, 'French Pastry', '+1-555-0103', true),
('Carlos Martinez', 41, 18, 'Mexican Cuisine', '+1-555-0104', true);
```

#### Sample Nutritionists
```sql
INSERT INTO public.nutritionists (name, qualification, specialization, phone, email) VALUES
('Dr. Sarah Johnson', 'PhD in Nutrition', 'Weight Management', '+1-555-0201', 'sarah.johnson@nutrition.com'),
('Dr. Raj Patel', 'MS in Clinical Nutrition', 'Sports Nutrition', '+1-555-0202', 'raj.patel@nutrition.com'),
('Dr. Emily White', 'RD, Certified Dietitian', 'Diabetes Care', '+1-555-0203', 'emily.white@nutrition.com');
```

#### Sample Menu Items
```sql
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

---

## Migration 2: Payments Table
**Date**: 2025-11-10  
**File**: `20251110172004_60379a62-53ff-4269-a599-406e963a9c42.sql`

### Payments Table
```sql
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
```

### Enable RLS
```sql
ALTER TABLE public.payments ENABLE ROW LEVEL SECURITY;
```

### RLS Policies
```sql
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
```

### Indexes
```sql
CREATE INDEX idx_payments_order_id ON public.payments(order_id);
CREATE INDEX idx_payments_status ON public.payments(payment_status);
```

---

## Migration 3: Delivery Tracking Fields
**Date**: 2025-11-10  
**File**: `20251110175043_00339e97-7214-46a4-be54-cf19682b1ef3.sql`

### Add Delivery Tracking to Orders
```sql
ALTER TABLE public.orders 
ADD COLUMN IF NOT EXISTS estimated_delivery_time timestamp with time zone,
ADD COLUMN IF NOT EXISTS preparation_time integer DEFAULT 20,
ADD COLUMN IF NOT EXISTS actual_delivery_time timestamp with time zone;
```

### Add Payment Details
```sql
ALTER TABLE public.payments
ADD COLUMN IF NOT EXISTS card_last_four text,
ADD COLUMN IF NOT EXISTS upi_id text,
ADD COLUMN IF NOT EXISTS payment_provider text;
```

---

## Migration 4: Reviews System
**Date**: 2025-11-10  
**File**: `20251110181114_34703ea6-0fc7-4f43-a521-91b57b457fe9.sql`

### Reviews Table
```sql
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
```

### Enable RLS
```sql
ALTER TABLE public.reviews ENABLE ROW LEVEL SECURITY;
```

### RLS Policies
```sql
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
```

### Indexes
```sql
CREATE INDEX IF NOT EXISTS idx_reviews_item_id ON public.reviews(item_id);
CREATE INDEX IF NOT EXISTS idx_reviews_customer_id ON public.reviews(customer_id);
```

### Enable Realtime
```sql
ALTER PUBLICATION supabase_realtime ADD TABLE public.orders;
```

---

## Migration 5: Security Improvements - PII Protection
**Date**: 2025-11-10  
**File**: `20251110190438_7f5f32e6-7c38-4613-8638-87ad0483fe38.sql`

### Drop Overly Permissive Policies
```sql
DROP POLICY IF EXISTS "Anyone can view chefs" ON public.chefs;
DROP POLICY IF EXISTS "Anyone can view nutritionists" ON public.nutritionists;
```

### Create Public Views
```sql
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
```

### Grant Access to Views
```sql
GRANT SELECT ON public.chefs_public TO anon, authenticated;
GRANT SELECT ON public.nutritionists_public TO anon, authenticated;
```

### Restricted Policies for Authenticated Users
```sql
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
```

### Fix Customer Order Summary View
```sql
DROP VIEW IF EXISTS public.customer_order_summary;

CREATE VIEW public.customer_order_summary 
WITH (security_invoker=true)
AS
SELECT 
  c.customer_id,
  c.name as customer_name,
  c.email,
  c.phone,
  c.diet_type,
  COUNT(o.order_id) as total_orders,
  COALESCE(SUM(o.total_price), 0) as total_spent,
  MAX(o.order_date) as last_order_date
FROM public.customers c
LEFT JOIN public.orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.email, c.phone, c.diet_type;
```

---

## Migration 6: Critical RLS Fix
**Date**: 2025-11-11  
**File**: `20251111021328_c902e9bf-8a60-4657-bfe1-e17613e4c9ad.sql`

### Make auth_user_id NOT NULL
```sql
-- Verify there are no existing NULL values
DO $$
BEGIN
  IF EXISTS (SELECT 1 FROM public.customers WHERE auth_user_id IS NULL) THEN
    RAISE EXCEPTION 'Cannot make auth_user_id NOT NULL: existing records with NULL values found. Please fix data first.';
  END IF;
END $$;

-- Make the column NOT NULL
ALTER TABLE public.customers 
ALTER COLUMN auth_user_id SET NOT NULL;

-- Add a comment explaining the security requirement
COMMENT ON COLUMN public.customers.auth_user_id IS 'Required foreign key to auth.users - must never be NULL as it is the foundation of RLS policies for user data isolation';
```

---

## Summary

This database schema supports a food delivery application with:

- **User Management**: Customers, Chefs, and Nutritionists
- **Menu System**: Menu items with categories, pricing, and stock management
- **Order System**: Orders, order items, and delivery tracking
- **Payment System**: Payment processing with multiple payment modes
- **Reviews**: Customer reviews and ratings for menu items
- **Diet Plans**: Personalized diet plans managed by nutritionists
- **Security**: Row Level Security (RLS) policies to protect user data
- **Automation**: Triggers for stock management and auto-creating customer profiles

All tables have proper RLS policies to ensure users can only access their own data, while public information (menu items, reviews) is accessible to all users.
