# Dyad Portal Mini-Store Template - Complete Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Configuration Files](#configuration-files)
4. [Data Models (Collections)](#data-models-collections)
5. [Frontend Pages & Routes](#frontend-pages--routes)
6. [API Endpoints](#api-endpoints)
7. [Components](#components)
8. [Business Logic](#business-logic)
9. [Styling & Design](#styling--design)
10. [Development Workflow](#development-workflow)
11. [Security Considerations](#security-considerations)
12. [Customization Guide](#customization-guide)

---

## Project Overview

### What is this template?
A complete e-commerce portal template built with Next.js 15 and Payload CMS 3.0, designed for selling products (snacks in the demo) with role-based access control.

### Key Features
- **Public Browsing**: View products without authentication
- **User Authentication**: Registration, login, password reset
- **Shopping Cart**: Client-side cart with localStorage persistence
- **Order Management**: Place orders and track status
- **Admin Dashboard**: Manage orders and inventory
- **Responsive Design**: Mobile-first approach
- **Role-Based Access**: User vs Admin permissions

### Technology Stack
- **Frontend**: Next.js 15, React 19, TypeScript
- **Backend**: Payload CMS 3.0
- **Database**: PostgreSQL
- **Styling**: Tailwind CSS + Radix UI
- **Authentication**: Payload built-in auth
- **Image Processing**: Sharp
- **Email**: Nodemailer with Gmail

---

## Architecture

### System Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Payload CMS   │    │   Database      │
│   (Next.js)     │◄──►│   (Backend)     │◄──►│  (Postgres)     │
│                 │    │                 │    │                 │
│ - Pages         │    │ - Collections   │    │ - Users         │
│ - Components    │    │ - API Routes    │    │ - Snacks        │
│ - Cart Context  │    │ - Auth          │    │ - Orders        │
│ - UI Library    │    │ - Access Control│    │ - Media         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Data Flow
1. **Product Browsing**: Frontend fetches snacks from Payload API
2. **Cart Management**: Client-side state with localStorage persistence
3. **Order Placement**: Frontend sends cart data to API endpoint
4. **Order Processing**: Backend validates data and creates order record
5. **Admin Management**: Admin dashboard fetches and updates orders

### Authentication Flow
1. **Registration**: User creates account → Payload creates user with 'user' role
2. **Login**: Credentials verified → JWT token issued
3. **Protected Routes**: Middleware checks authentication and roles
4. **API Access**: Headers contain auth token for API requests

---

## Configuration Files

### `package.json`
**Purpose**: Defines dependencies, scripts, and project metadata

**Key Dependencies**:
- `payload`: Core CMS functionality
- `@payloadcms/next`: Next.js integration
- `@payloadcms/db-vercel-postgres`: Database adapter
- `@payloadcms/email-nodemailer`: Email functionality
- `next`: React framework
- `@radix-ui/*`: UI component primitives
- `tailwindcss`: Styling framework

**Scripts**:
- `dev`: Development server
- `build`: Production build
- `generate:types`: Generate TypeScript types from Payload
- `migrate:create`: Create database migrations

### `src/payload.config.ts`
**Purpose**: Main Payload CMS configuration

**Key Sections**:
```typescript
export default buildConfig({
  admin: {
    components: {
      beforeDashboard: ['@/components/before-dashboard'], // Custom admin dashboard component
    },
    user: Users.slug, // Use Users collection for authentication
  },
  collections: [Users, Media, Snacks, Orders], // Data models
  db: vercelPostgresAdapter({...}), // Database configuration
  email: nodemailerAdapter({...}), // Email configuration
})
```

**Functions**:
- `getServerSideURL()`: Determines server URL for different environments
- Configures CORS, admin panel, collections, and plugins

### `next.config.ts`
**Purpose**: Next.js configuration (standard setup)

### Environment Variables (.env.example)
```env
PAYLOAD_SECRET=your-secret-key
POSTGRES_URL=your-postgres-connection-string
NEXT_PUBLIC_SERVER_URL=your-app-url
GMAIL_USER=your-gmail-address
GOOGLE_APP_PASSWORD=your-app-password
EMAIL_DEFAULT_FROM_NAME=Dyad app
DYAD_DISABLE_DB_PUSH=true # Optional: disable auto DB pushes
```

---

## Data Models (Collections)

### Users Collection (`src/collections/Users.ts`)
**Purpose**: User management and authentication

**Fields**:
- `email`: User email (built-in)
- `password`: User password (built-in, hashed)
- `firstName`: User's first name (required)
- `lastName`: User's last name (required)
- `role`: User role ('user' or 'admin', default: 'user')

**Access Control**:
- `create`: `anyone` (allows registration)
- `read`: `adminsOrSelf` (users can read own profile, admins can read all)
- `update`: `adminsOrSelf` (users can update own profile, admins can update all)
- `admin`: `adminsOnly` (only admins can access admin panel)

**Special Features**:
- Built-in authentication with forgot password functionality
- Custom email template for password reset
- Role-based field access (role field only visible to admins)

### Snacks Collection (`src/collections/Snacks.ts`)
**Purpose**: Product catalog management

**Fields**:
- `name`: Product name (required)
- `description`: Product description (required)
- `price`: Product price (required, min: 0)
- `image`: Product image (upload to Media collection)
- `imageUrl`: External image URL (fallback)
- `available`: Availability status (checkbox, default: true)
- `category`: Product category (select: chips, candy, cookies, nuts, crackers, drinks)

**Access Control**:
- `read`: `anyone` (public browsing)
- `create/update/delete`: `admins` (admin only)
- `admin`: `adminsOnly`

**Business Logic**:
- Either `image` or `imageUrl` should be provided
- Category system for product organization
- Availability toggle for inventory management

### Orders Collection (`src/collections/Orders.ts`)
**Purpose**: Order management and tracking

**Fields**:
- `user`: Relationship to Users collection (required)
- `items`: Array of snack items with quantities (required, min: 1)
- `status`: Order status ('pending', 'completed', 'cancelled', default: 'pending')
- `totalAmount`: Calculated total price (required, min: 0)
- `orderDate`: Order timestamp (default: current date)

**Access Control**:
- `read`: `adminsOrOwner('user')` (users see own orders, admins see all)
- `create`: `authenticated` (any logged-in user can create orders)
- `update/delete`: `admins` (admin only)
- `admin`: `adminsOnly`

**Business Logic**:
- Items array contains snack references and quantities
- Total amount calculated server-side to prevent manipulation
- Status tracking for order fulfillment

### Media Collection (`src/collections/Media.ts`)
**Purpose**: File upload and management

**Features**:
- Image upload with Sharp processing
- Alt text for accessibility
- Integration with Snacks collection
- Standard Payload Media collection

---

## Frontend Pages & Routes

### Homepage (`src/app/(frontend)/page.tsx`)
**Purpose**: Main product browsing interface

**Key Features**:
- Hero section with animated background
- Product grid with hover effects
- Modal dialogs for product details
- Authentication-aware "Order Now" buttons
- Responsive design with animations

**Data Flow**:
1. Fetches all available snacks from Payload API
2. Displays products in responsive grid
3. Shows product details in modal dialogs
4. Handles authentication state for ordering

**Components Used**:
- `SiteHeader`: Navigation with user state
- `Card`, `Badge`, `Button`: UI components
- `Dialog`: Product detail modal
- `AddToCartButton`: Cart functionality

### Authentication Pages

#### Login (`src/app/(frontend)/login/page.tsx`)
**Purpose**: User authentication

**Features**:
- Email/password login form
- Redirect to dashboard or homepage
- Error handling for invalid credentials
- Link to registration and password reset

#### Register (`src/app/(frontend)/register/page.tsx`)
**Purpose**: New user registration

**Features**:
- Registration form with validation
- First name, last name, email, password fields
- Automatic login after registration
- Default 'user' role assignment

#### Forgot Password (`src/app/(frontend)/forgot-password/page.tsx`)
**Purpose**: Password reset initiation

**Features**:
- Email input for password reset
- Integration with Payload's forgot password
- Success message and instructions

#### Reset Password (`src/app/(frontend)/reset-password/page.tsx`)
**Purpose**: Password reset completion

**Features**:
- New password form with token validation
- Token-based password reset
- Redirect to login after success

### Checkout Flow

#### Checkout Page (`src/app/(frontend)/checkout/page.tsx`)
**Purpose**: Order placement and cart review

**Features**:
- Cart items display with quantities
- Total price calculation
- Order confirmation form
- Integration with order API

#### Checkout Form (`src/app/(frontend)/checkout/checkout-form.tsx`)
**Purpose**: Order submission form

**Logic**:
- Validates cart items
- Submits order to API
- Clears cart after successful order
- Redirects to order history

### Order Management

#### My Orders (`src/app/(frontend)/my-orders/page.tsx`)
**Purpose**: User order history

**Features**:
- Display user's order history
- Order status tracking
- Order details with items
- Responsive order cards

#### Order Detail (`src/app/(frontend)/order/[id]/page.tsx`)
**Purpose**: Individual order view

**Features**:
- Detailed order information
- Item list with quantities
- Order status and date
- Navigation back to order list

### Admin Dashboard

#### Admin Dashboard (`src/app/(frontend)/admin-dashboard/page.tsx`)
**Purpose**: Order management for admins

**Features**:
- Order statistics overview
- All orders list with filtering
- Status update buttons
- Real-time status changes

#### Order Status Update (`src/app/(frontend)/admin-dashboard/order-status-update.tsx`)
**Purpose**: Order status management

**Logic**:
- Updates order status via API
- Real-time UI updates
- Admin-only access validation

### Product Detail

#### Snack Detail (`src/app/(frontend)/snack/[id]/page.tsx`)
**Purpose**: Individual product page

**Features**:
- Product information display
- Add to cart functionality
- Related products (if implemented)
- Breadcrumb navigation

---

## API Endpoints

### Orders API (`src/app/api/orders/route.ts`)
**Purpose**: Order creation and retrieval

#### POST `/api/orders`
**Function**: Create new order
**Authentication**: Required
**Request Body**:
```json
{
  "items": [
    {
      "snack": "snack-id",
      "quantity": 2
    }
  ]
}
```

**Validation**:
- User authentication check
- Item validation (existence, availability)
- Quantity validation (1-100, integer)
- Price validation and recalculation
- Server-side total computation

**Response**:
```json
{
  "success": true,
  "doc": {
    "id": "order-id",
    "user": "user-id",
    "items": [...],
    "totalAmount": 19.98,
    "status": "pending",
    "orderDate": "2024-01-01T00:00:00.000Z"
  }
}
```

#### GET `/api/orders`
**Function**: Get user's orders
**Authentication**: Required
**Response**:
```json
{
  "orders": [
    {
      "id": "order-id",
      "items": [...],
      "totalAmount": 19.98,
      "status": "pending",
      "orderDate": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

### Order Status Update API (`src/app/api/orders/update-status/route.ts`)
**Purpose**: Update order status (admin only)

#### PATCH `/api/orders/update-status`
**Authentication**: Admin required
**Request Body**:
```json
{
  "orderId": "order-id",
  "status": "completed"
}
```

**Validation**:
- Admin role verification
- Order existence check
- Valid status values

### User Authentication API (`src/app/api/users/forgot-password/route.ts`)
**Purpose**: Password reset functionality

#### POST `/api/users/forgot-password`
**Function**: Initiate password reset
**Request Body**:
```json
{
  "email": "user@example.com"
}
```

**Logic**:
- Validates email existence
- Generates reset token
- Sends reset email
- Returns success message

### Seed Data API (`src/app/(frontend)/next/seed/route.ts`)
**Purpose**: Populate database with sample data

#### GET `/next/seed`
**Function**: Create sample snacks and admin user
**Authentication**: None (development only)
**Features**:
- Creates sample snack items
- Creates admin user if not exists
- Uses placeholder images

---

## Components

### UI Components (`src/components/ui/`)
**Purpose**: Reusable UI components based on Radix UI

**Key Components**:
- `Button`: Styled button with variants
- `Card`: Content container with header/footer
- `Dialog`: Modal dialog component
- `Input`: Form input with validation
- `Badge`: Status/category indicators
- `Separator`: Visual dividers
- `Sheet`: Slide-out panel (used for cart)

### Business Components

#### Cart Context (`src/lib/cart-context.tsx`)
**Purpose**: Shopping cart state management

**Features**:
- React Context for global cart state
- localStorage persistence
- Cart actions (add, remove, update, clear)
- Total calculation utilities

**State Structure**:
```typescript
interface CartState {
  items: CartItem[]
  isOpen: boolean
}

interface CartItem {
  id: string
  name: string
  price: number
  image?: { url: string; alt?: string }
  quantity: number
  category: string
}
```

**Actions**:
- `addItem`: Add item to cart (increments quantity if exists)
- `removeItem`: Remove item from cart
- `updateQuantity`: Update item quantity
- `clearCart`: Remove all items
- `toggleCart/openCart/closeCart`: Cart visibility control

#### Site Header (`src/components/site-header.tsx`)
**Purpose**: Main navigation component

**Features**:
- User authentication state display
- Navigation links
- Cart button with item count
- Logout functionality
- Responsive design

#### Cart Button (`src/components/cart-button.tsx`)
**Purpose**: Cart access and item count display

**Features**:
- Displays total item count
- Opens cart sidebar
- Animated badge for count
- Responsive sizing

#### Add to Cart Button (`src/components/add-to-cart-button.tsx`)
**Purpose**: Product cart integration

**Features**:
- Adds snack to cart
- Success feedback
- Loading states
- Integration with cart context

#### Cart Sidebar (`src/components/cart-sidebar.tsx`)
**Purpose**: Cart management interface

**Features**:
- Display cart items
- Quantity controls
- Remove items
- Total calculation
- Checkout button
- Clear cart option

#### Logout Button (`src/components/logout-button.tsx`)
**Purpose**: User logout functionality

**Features**:
- Payload logout integration
- Redirect to homepage
- Confirmation dialog

#### Before Dashboard (`src/components/before-dashboard/`)
**Purpose**: Custom admin dashboard welcome component

**Features**:
- Welcome message for admin panel
- Seed data button for development
- Custom styling for admin interface

---

## Business Logic

### Shopping Cart System
**Implementation**: Client-side React Context with localStorage

**Workflow**:
1. **Add to Cart**: User clicks "Add to Cart" → Item added to context → Saved to localStorage
2. **Cart Persistence**: Cart data persists across sessions via localStorage
3. **Quantity Management**: Users can update/remove items in cart sidebar
4. **Checkout**: Cart items sent to API for order creation
5. **Cart Clearing**: Cart cleared after successful order

**Security Considerations**:
- Prices validated server-side during order creation
- Item availability checked during order processing
- Quantity limits enforced (1-100 per item)

### Order Processing Workflow
**Steps**:
1. **Cart Review**: User reviews items in cart
2. **Order Submission**: Cart data sent to `/api/orders`
3. **Server Validation**: Items, prices, and availability verified
4. **Order Creation**: Order record created in database
5. **Cart Clearing**: Client cart cleared
6. **Confirmation**: User redirected to order history

**Price Security**:
- Prices recalculated server-side using current snack prices
- Prevents client-side price manipulation
- Validates snack availability at order time

### Authentication System
**Implementation**: Payload built-in authentication

**User Roles**:
- **user**: Can browse products, place orders, view own orders
- **admin**: All user permissions + order management, inventory management

**Access Control Patterns**:
```typescript
// Access control functions
export const anyone = () => true
export const authenticated = ({ req }) => req.user !== null
export const admins = ({ req }) => req.user?.role === 'admin'
export const adminsOnly = ({ req }) => req.user?.role === 'admin'
export const adminsOrSelf = ({ req, id }) => {
  const user = req.user
  return user?.role === 'admin' || user?.id === id
}
export const adminsOrOwner = (userField) => ({ req, doc }) => {
  const user = req.user
  return user?.role === 'admin' || user?.id === doc[userField]?.id
}
```

### Email System
**Implementation**: Nodemailer with Gmail

**Features**:
- Password reset emails
- Custom email templates
- Environment-based configuration

**Configuration**:
```typescript
email: nodemailerAdapter({
  defaultFromAddress: process.env.GMAIL_USER,
  defaultFromName: process.env.EMAIL_DEFAULT_FROM_NAME,
  transport: await nodemailer.createTransport({
    service: 'gmail',
    auth: {
      user: process.env.GMAIL_USER,
      pass: process.env.GOOGLE_APP_PASSWORD,
    },
  }),
})
```

---

## Styling & Design

### Tailwind CSS Configuration
**Purpose**: Utility-first styling framework

**Key Features**:
- Custom color palette (amber, rose, blue gradients)
- Responsive breakpoints (sm, md, lg, xl)
- Animation utilities
- Component variants

### Design System
**Color Palette**:
- Primary: Amber (`amber-500`, `amber-600`)
- Secondary: Rose (`rose-500`, `rose-600`)
- Accent: Blue (`blue-200`, `blue-300`)
- Neutral: Gray shades (`gray-50` to `gray-900`)

**Typography**:
- Headings: `font-black`, `font-bold`
- Body: `font-normal`
- Gradients: `bg-gradient-to-r` for text effects

**Animations**:
- `animate-pulse`: Breathing effects
- `animate-bounce`: Bouncing elements
- `animate-fade-in`: Fade-in animations
- `animate-gradient-x`: Gradient animations
- Custom animation delays for staggered effects

### Responsive Design
**Breakpoints**:
- Mobile: `< 768px`
- Tablet: `768px - 1199px`
- Desktop: `1200px+`

**Patterns**:
- Grid layouts: `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
- Responsive spacing: `px-4 sm:px-6 lg:px-8`
- Mobile-first approach

### Component Styling
**Cards**:
- Rounded corners: `rounded-3xl`
- Shadows: `shadow-xl`, `shadow-2xl`
- Hover effects: `hover:scale-[1.02]`, `hover:shadow-2xl`
- Glass morphism: `bg-white/95 backdrop-blur-xl`

**Buttons**:
- Gradient backgrounds: `bg-gradient-to-r from-amber-500 to-rose-500`
- Hover states: `hover:from-amber-600 hover:to-rose-600`
- Transitions: `transition-all duration-300`
- Rounded: `rounded-full`

---

## Development Workflow

### Setup Instructions
1. **Clone Repository**:
   ```bash
   git clone <repository-url>
   cd portal-mini-store-template-dyad
   ```

2. **Install Dependencies**:
   ```bash
   pnpm install
   ```

3. **Environment Setup**:
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Database Setup**:
   - Configure your PostgreSQL database (self-hosted or any PostgreSQL provider)
   - Set `POSTGRES_URL` in environment with your PostgreSQL connection string

5. **Start Development**:
   ```bash
   pnpm dev
   ```

### Development Commands
```bash
pnpm dev          # Start development server
pnpm build        # Build for production
pnpm start        # Start production server
pnpm generate:types  # Generate TypeScript types
pnpm lint         # Run ESLint
pnpm migrate:create  # Create database migration
```

### First Time Setup
1. **Create Admin User**:
   - Navigate to `/admin`
   - Create first admin account
   - This user will have admin privileges

2. **Add Products**:
   - Use admin panel to add snack items
   - Upload images or use external URLs
   - Set prices and categories

3. **Test User Flow**:
   - Create regular user account
   - Test product browsing and ordering
   - Verify order management

### Database Management
**Migrations**:
- Payload handles schema migrations automatically
- Use `DYAD_DISABLE_DB_PUSH=true` to disable auto-pushes
- Create manual migrations with `pnpm migrate:create`

**Seed Data**:
- Visit `/next/seed` to populate sample data
- Creates sample snacks and admin user
- Useful for development and testing

---

## Security Considerations

### Authentication Security
- **Password Hashing**: Payload automatically hashes passwords
- **JWT Tokens**: Secure token-based authentication
- **Role-Based Access**: Proper permission separation
- **Session Management**: Secure cookie handling

### API Security
- **Input Validation**: All API inputs validated
- **Price Verification**: Server-side price calculation
- **Authentication Checks**: Protected endpoints require auth
- **Rate Limiting**: Consider implementing for production

### Data Security
- **Environment Variables**: Sensitive data in .env
- **Database Security**: Use connection strings with proper permissions
- **CORS Configuration**: Properly configured for allowed origins
- **SQL Injection**: Payload ORM prevents SQL injection

### Client-Side Security
- **XSS Prevention**: React's built-in XSS protection
- **CSRF Protection**: Payload includes CSRF protection
- **Content Security Policy**: Consider implementing CSP headers
- **Secure Cookies**: Use secure, httpOnly cookies in production

---

## Customization Guide

### Adding New Product Fields
1. **Update Snacks Collection** (`src/collections/Snacks.ts`):
   ```typescript
   fields: [
     // ... existing fields
     {
       name: 'newField',
       type: 'text',
       required: false,
     },
   ]
   ```

2. **Update Frontend Display**:
   - Modify product cards to show new field
   - Update product detail pages
   - Add to cart form if needed

### Creating New User Roles
1. **Update Users Collection**:
   ```typescript
   {
     name: 'role',
     type: 'select',
     options: [
       { label: 'Admin', value: 'admin' },
       { label: 'User', value: 'user' },
       { label: 'Manager', value: 'manager' }, // New role
     ],
   }
   ```

2. **Update Access Control**:
   - Add new access control functions
   - Update existing permissions
   - Create role-based UI components

### Adding New API Endpoints
1. **Create Route File**:
   ```typescript
   // src/app/api/custom/route.ts
   export async function POST(request: NextRequest) {
     // Custom logic
   }
   ```

2. **Add Authentication**:
   ```typescript
   const { user } = await payload.auth({ headers: request.headers })
   if (!user) {
     return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
   }
   ```

### Customizing Styling
1. **Update Tailwind Config**:
   - Add custom colors
   - Define custom animations
   - Configure breakpoints

2. **Create Custom Components**:
   - Extend existing UI components
   - Create new design system components
   - Maintain consistency with existing patterns

### Adding New Pages
1. **Create Page Component**:
   ```typescript
   // src/app/(frontend)/new-page/page.tsx
   export default function NewPage() {
     return <div>New Page Content</div>
   }
   ```

2. **Add Navigation**:
   - Update `SiteHeader` component
   - Add routing links
   - Consider authentication requirements

### Database Customization
1. **New Collections**:
   - Create collection file in `src/collections/`
   - Define fields and access control
   - Add to `payload.config.ts`

2. **Collection Relationships**:
   - Use `relationship` fields for connections
   - Configure proper access control
   - Handle depth in API queries

### Email Customization
1. **Custom Email Templates**:
   - Modify forgot password template
   - Add new email types
   - Use HTML templates for rich formatting

2. **Email Configuration**:
   - Configure different email providers
   - Add email queue for production
   - Handle email failures gracefully

---

## File Structure Summary

```
src/
├── app/
│   ├── (frontend)/           # Frontend pages
│   │   ├── page.tsx         # Homepage
│   │   ├── login/           # Authentication pages
│   │   ├── checkout/        # Checkout flow
│   │   ├── admin-dashboard/ # Admin interface
│   │   └── api/             # API routes
│   └── (payload)/           # Payload CMS admin
├── collections/             # Data models
│   ├── Users.ts            # User management
│   ├── Snacks.ts           # Product catalog
│   ├── Orders.ts           # Order management
│   ├── Media.ts            # File uploads
│   └── access/             # Access control functions
├── components/             # React components
│   ├── ui/                 # UI component library
│   ├── cart-button.tsx     # Cart access
│   ├── add-to-cart-button.tsx
│   ├── cart-sidebar.tsx    # Cart management
│   └── site-header.tsx     # Navigation
├── lib/                    # Utilities and context
│   ├── cart-context.tsx    # Cart state management
│   └── utils.ts            # Helper functions
├── hooks/                  # Custom React hooks
├── endpoints/              # Payload endpoints
├── migrations/             # Database migrations
├── payload.config.ts       # Payload configuration
└── payload-types.ts        # Generated types
```

This documentation provides a comprehensive reference for understanding and modifying the portal mini-store template. Each section covers the purpose, implementation details, and customization options for that part of the system.
