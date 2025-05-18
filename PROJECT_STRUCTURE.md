# Tea World AI - Project Structure

## System Architecture

```
tea-world-ai/
├── client/                 # Customer-facing application
│   ├── public/             # Static assets
│   └── src/
│       ├── components/     # React components
│       │   ├── layout/     # Layout components
│       │   ├── menu/       # Menu-related components
│       │   ├── cart/       # Shopping cart components
│       │   ├── auth/       # Authentication components
│       │   └── ui/         # Reusable UI components
│       ├── pages/          # Next.js pages
│       ├── hooks/          # Custom React hooks
│       ├── services/       # Service connectors
│       │   ├── supabase.ts # Supabase client setup
│       │   ├── auth.ts     # Authentication services
│       │   ├── menu.ts     # Menu-related services
│       │   └── ai.ts       # AI integration services
│       ├── utils/          # Utility functions
│       ├── styles/         # Global styles
│       ├── contexts/       # React context providers
│       └── types/          # TypeScript type definitions
│
├── admin/                  # Admin dashboard application
│   ├── public/             # Static assets
│   └── src/
│       ├── components/     # React components
│       │   ├── layout/     # Layout components
│       │   ├── dashboard/  # Dashboard widgets
│       │   ├── menu/       # Menu management
│       │   ├── orders/     # Order management
│       │   ├── marketing/  # Marketing tools
│       │   ├── auth/       # Authentication components
│       │   └── ui/         # Reusable UI components
│       ├── pages/          # Next.js pages
│       ├── hooks/          # Custom React hooks
│       ├── services/       # Service connectors
│       │   ├── supabase.ts # Supabase client setup
│       │   ├── auth.ts     # Authentication services
│       │   ├── menu.ts     # Menu management services
│       │   ├── analytics.ts# Analytics services
│       │   └── ai.ts       # AI integration services
│       ├── utils/          # Utility functions
│       ├── styles/         # Global styles
│       ├── contexts/       # React context providers
│       └── types/          # TypeScript type definitions
│
└── shared/                 # Shared code between projects
    ├── types/              # Shared TypeScript types
    └── utils/              # Shared utility functions
```

## Database Schema (Supabase)

### Tables

1. **users**
   - id (primary key)
   - email
   - encrypted_password
   - name
   - phone
   - created_at
   - role (customer, admin, staff)
   - preferences (JSON)
   - is_guest (boolean)
   - guest_id (UUID, for anonymous users)

2. **products**
   - id (primary key)
   - name
   - category_id (foreign key)
   - description
   - ai_description
   - price
   - image_url
   - is_available
   - created_at
   - updated_at
   - attributes (JSON - sweetness, ice level, etc.)

3. **categories**
   - id (primary key)
   - name
   - description
   - display_order
   - image_url

4. **toppings**
   - id (primary key)
   - name
   - price
   - is_available
   - image_url

5. **orders**
   - id (primary key)
   - user_id (foreign key, nullable for anonymous orders)
   - guest_id (UUID, for anonymous users)
   - contact_info (JSON - for guest checkout: name, phone, email)
   - total_amount
   - status (pending, confirmed, preparing, ready, completed, cancelled)
   - created_at
   - completed_at
   - payment_method
   - notes
   - order_type (pickup, delivery)
   - is_anonymous (boolean)

6. **order_items**
   - id (primary key)
   - order_id (foreign key)
   - product_id (foreign key)
   - quantity
   - price
   - customizations (JSON)
   - subtotal

7. **order_item_toppings**
   - id (primary key)
   - order_item_id (foreign key)
   - topping_id (foreign key)
   - quantity
   - price

8. **reviews**
   - id (primary key)
   - user_id (foreign key, nullable)
   - guest_id (UUID, nullable, for anonymous reviews)
   - product_id (foreign key)
   - rating
   - comment
   - created_at
   - is_anonymous (boolean)

9. **promotions**
   - id (primary key)
   - name
   - description
   - discount_type (percentage, fixed)
   - discount_value
   - start_date
   - end_date
   - is_active
   - conditions (JSON)
   - guest_eligible (boolean)

10. **ai_interactions**
    - id (primary key)
    - user_id (foreign key, nullable)
    - guest_id (UUID, nullable, for anonymous users)
    - interaction_type (recommendation, chat, description)
    - input
    - output
    - created_at
    - feedback (JSON)
    - is_anonymous (boolean)

11. **guest_sessions**
    - id (primary key, UUID)
    - created_at
    - last_active_at
    - device_info (JSON)
    - preferences (JSON)
    - conversion_status (browsing, cart_created, ordered, converted_to_user)

## Supabase Features Utilization

### Authentication
- Email/password authentication
- Social login providers (Google, Facebook)
- Role-based access control (customer, admin, staff)
- Session management with JWT
- Anonymous sessions with unique guest IDs

### Database Access
- Row Level Security (RLS) policies for data protection
- Direct table access with proper permissions
- Real-time subscriptions for live updates
- Special policies for anonymous access

### Edge Functions

#### Customer Functions
- `auth-register` - Custom registration with preferences
- `auth-anonymous-session` - Create and manage anonymous sessions
- `ai-recommend` - Get personalized recommendations (works for both registered and anonymous users)
- `ai-chat` - Chat with the AI assistant
- `order-create` - Create new order with validation
- `order-anonymous` - Create anonymous order with contact info

#### Admin Functions
- `dashboard-summary` - Generate dashboard summary data
- `ai-generate-description` - Generate AI description for products
- `ai-marketing-content` - Create AI marketing content
- `promotions-send` - Send promotions to customer segments
- `guest-analytics` - Analyze anonymous user behavior

### Triggers and Webhooks
- Order status change notifications
- Inventory updates on order completion
- Review submission alerts
- New user welcome message
- Anonymous to registered user conversion tracking

## Anonymous Ordering Flow

1. **Session Creation**:
   - Generate unique guest_id when a new visitor accesses the site
   - Store in browser storage for session persistence
   - Create entry in guest_sessions table

2. **Browsing Experience**:
   - Track preferences and interactions anonymously
   - Provide AI recommendations based on session behavior
   - Store cart data linked to guest_id

3. **Checkout Process**:
   - Collect minimal contact info (name, phone/email) at checkout
   - Option to create account or continue as guest
   - Process payment and create order with guest_id

4. **Post-Order**:
   - Provide order tracking link via SMS/email
   - Offer account creation incentives
   - If account created later, associate past orders with new account

## Technology Dependencies

- **Frontend**
  - Next.js
  - React
  - SCSS modules
  - Ant Design
  - Jotai
  - Supabase JS Client

- **Database & Infrastructure**
  - Supabase (PostgreSQL)
  - Supabase Auth
  - Supabase Storage
  - Supabase Edge Functions
  - Supabase Realtime
  - Vercel (hosting)

- **AI Integration**
  - OpenAI GPT-4 API (via Supabase Edge Functions)
  - OpenAI DALL-E API (via Supabase Edge Functions)
  - OpenAI Embeddings API (via Supabase pgvector) 