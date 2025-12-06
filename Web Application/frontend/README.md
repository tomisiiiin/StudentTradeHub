# Student TradeHub – Frontend

This is the **frontend** of the Student TradeHub project — a student-only marketplace platform built for Memorial University (MUN) students to buy, sell, and trade items safely within their community.

## 📋 Table of Contents
- [Overview](#-overview)
- [Tech Stack](#️-tech-stack)
- [Project Structure](#-project-structure)
- [Getting Started](#-getting-started)
- [Architecture](#-architecture)
- [Pages & Routing](#-pages--routing)
- [Components](#-components)
- [Context (State Management)](#-context-state-management)
- [API Integration](#-api-integration)
- [Authentication Flow](#-authentication-flow)
- [Testing](#-testing)
- [Environment Variables](#-environment-variables)
- [Contributors](#-contributors)

---

## 🧭 Overview

The frontend is built using **Next.js 15** with the App Router and **Tailwind CSS v4**, providing a responsive and modern interface for browsing, posting, and managing product listings. The application supports both regular users and administrators with role-based access control.

### Key Features
- **Product Marketplace** – Browse, search, and filter products by category, condition, and status
- **Seller Dashboard** – List, edit, and manage your own products
- **Order Management** – Track purchases and sales with real-time status updates
- **Review System** – Rate sellers after completed orders
- **Admin Panel** – Manage users, products, and orders platform-wide
- **User Preferences** – Save payment methods and addresses for quick checkout

---

## ⚙️ Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| **Next.js** | 15.5.4 | React framework with App Router |
| **React** | 19.1.0 | UI library |
| **Tailwind CSS** | 4.x | Utility-first CSS framework |
| **React Icons** | 5.5.0 | Icon library |
| **Jest** | 29.7.0 | Testing framework |
| **React Testing Library** | 16.3.0 | Component testing utilities |

---

## 📁 Project Structure

```
frontend/
├── app/                          # Next.js App Router pages
│   ├── layout.js                 # Root layout with providers
│   ├── page.js                   # Home page (redirects to /buy)
│   ├── globals.css               # Global styles
│   ├── login/                    # Login page
│   ├── signup/                   # User registration
│   ├── forgot-password/          # Password recovery
│   ├── reset-password/           # Password reset
│   ├── verify-email/             # Email verification
│   ├── buy/                      # Product marketplace
│   ├── sell/                     # Seller dashboard
│   ├── product/[pid]/            # Product detail page
│   ├── checkout/                 # Checkout flow
│   ├── orders/                   # Order management
│   │   ├── page.jsx              # Orders list
│   │   └── [id]/page.jsx         # Order detail
│   ├── payment/                  # Payment method management
│   ├── address/                  # Address preferences
│   └── admin/                    # Admin dashboard
│       ├── layout.jsx            # Admin layout with protection
│       ├── page.jsx              # Admin overview
│       ├── users/                # User management
│       ├── products/             # Product management
│       └── orders/               # Order management
├── components/                   # Reusable React components
│   ├── Navbar.jsx               # Main navigation bar
│   ├── ProductCard.jsx          # Product display card
│   ├── ProductForm.jsx          # Product create/edit form
│   ├── EditProfile.jsx          # Profile editing modal
│   ├── ReviewModal.jsx          # Review submission modal
│   ├── ReviewPrompt.jsx         # Pending review prompt
│   ├── AddPaymentMethod.jsx     # Payment method form
│   ├── UserRoute.js             # User route protection
│   └── AdminRoute.js            # Admin route protection
├── context/                      # React Context providers
│   ├── AuthContext.js           # Authentication state
│   └── SearchContext.js         # Search and filter state
├── libs/                         # Utility functions and API calls
│   ├── auth.js                  # Authentication API functions
│   └── utlis.js                 # General utility and API functions
├── __tests__/                    # Test files
│   ├── components/              # Component tests
│   ├── context/                 # Context tests
│   ├── libs/                    # Library tests
│   └── pages/                   # Page tests
├── jest.config.js               # Jest configuration
├── jest.setup.js                # Jest setup file
├── next.config.mjs              # Next.js configuration
├── postcss.config.mjs           # PostCSS configuration
├── jsconfig.json                # JavaScript path aliases
└── package.json                 # Dependencies and scripts
```

---

## 🚀 Getting Started

### Prerequisites
- Node.js 18.x or higher
- npm 9.x or higher
- Backend server running on `http://localhost:8800`

### 1. Clone the repository
```bash
git clone https://github.com/<your-team>/StudentTradeHub.git
cd StudentTradeHub/Web\ Application/frontend
```

### 2. Install dependencies
```bash
npm install
```

### 3. Set up environment variables
Create a `.env.local` file in the frontend directory:
```env
NEXT_PUBLIC_API_URL=http://localhost:8800
```

### 4. Run the development server
```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

### Available Scripts

| Script | Description |
|--------|-------------|
| `npm run dev` | Start development server with Turbopack |
| `npm run build` | Build for production |
| `npm start` | Start production server |


---

## 🏗 Architecture

### App Router Structure
The application uses Next.js 15's App Router with the following pattern:

```
app/
├── layout.js          # Root layout wraps all pages
├── page.js            # Home route (/)
└── [route]/
    └── page.jsx       # Route component
```

### Provider Hierarchy
```jsx
<html>
  <body>
    <AuthProvider>           {/* Authentication state */}
      <SearchProvider>       {/* Search & filter state */}
        <Navbar />           {/* Global navigation */}
        <ReviewPrompt />     {/* Review reminders */}
        {children}           {/* Page content */}
      </SearchProvider>
    </AuthProvider>
  </body>
</html>
```

### Route Protection
Two higher-order components protect routes based on user roles:

- **`UserRoute`** – Ensures user is authenticated with `role: "user"`
- **`AdminRoute`** – Ensures user is authenticated with `role: "admin"`

---

## 📄 Pages & Routing

### Public Routes (No Authentication)

| Route | Page | Description |
|-------|------|-------------|
| `/login` | Login | User authentication |
| `/signup` | SignUp | New user registration (MUN email required) |
| `/forgot-password` | ForgotPassword | Request password reset email |
| `/reset-password` | ResetPassword | Set new password via email link |
| `/verify-email` | VerifyEmail | Email verification confirmation |

### User Routes (Authenticated Users)

| Route | Page | Description |
|-------|------|-------------|
| `/` | Dashboard | Redirects to `/buy` |
| `/buy` | BuyPage | Browse marketplace products |
| `/sell` | SellPage | Manage your listings |
| `/sell?edit=[id]` | SellPage | Edit existing product |
| `/product/[pid]` | ProductDetails | View product details |
| `/checkout?product=[id]` | Checkout | Purchase flow |
| `/orders` | OrdersPage | View buying/selling orders |
| `/orders/[id]` | OrderDetail | Order details and status |
| `/payment` | Payment | Manage payment methods |
| `/address` | AddressPreferences | Manage delivery/pickup addresses |

### Admin Routes (Admin Users)

| Route | Page | Description |
|-------|------|-------------|
| `/admin` | AdminDashboard | Overview with statistics |
| `/admin/users` | UserManagement | View all users |
| `/admin/users/[id]` | UserDetail | User details |
| `/admin/products` | ProductManagement | All products |
| `/admin/products/[pid]` | ProductDetail | Product details |
| `/admin/orders` | OrderManagement | All orders |
| `/admin/orders/[id]` | OrderDetail | Order details |

---

## 🧩 Components

### Navbar (`components/Navbar.jsx`)
Main navigation component featuring:
- Logo and branding
- Global search bar
- Category, condition, and status filters
- Navigation tabs (Buy/Sell/Orders or Admin)
- User profile dropdown with:
  - Profile picture
  - Seller rating display
  - Edit Profile option
  - Payment details link
  - Address preferences link
  - Logout functionality

### ProductCard (`components/ProductCard.jsx`)
Reusable product display card with:
- Product image with error fallback
- Category and condition badges
- Status badge (Available/Out of Stock/Inactive)
- Price and quantity display
- Seller information with rating
- Management mode (Edit/Delete) for product owners
- Links to product details

### ProductForm (`components/ProductForm.jsx`)
Product creation/editing form including:
- Name, description, price, quantity fields
- Category dropdown (Electronics, Books, Furniture, etc.)
- Condition selector (Brand New, Like New, Good, Used, Damaged)
- Status selector (Active, Draft)
- Image upload with preview (max 5MB)
- Form validation

### EditProfile (`components/EditProfile.jsx`)
Modal for editing user profile:
- Profile picture upload
- First/last name editing
- Password change with strength indicator
- Validation and error handling

### ReviewModal (`components/ReviewModal.jsx`)
Order review submission dialog:
- Star rating selection (1-5)
- Optional comment field
- Product and seller information display
- Submit/Skip actions

### ReviewPrompt (`components/ReviewPrompt.jsx`)
Automatic prompt for pending reviews:
- Checks for delivered orders without reviews
- Sequential review prompts
- Session-based checking

### Route Guards

**UserRoute (`components/UserRoute.js`)**
```jsx
// Protects routes for authenticated users with role: "user"
// Redirects to /login if not authenticated
// Redirects to /admin if user is admin
```

**AdminRoute (`components/AdminRoute.js`)**
```jsx
// Protects routes for authenticated users with role: "admin"
// Redirects to /login if not authenticated
// Redirects to /buy if user is not admin
```

---

## 🔄 Context (State Management)

### AuthContext (`context/AuthContext.js`)
Manages authentication state across the application.

**Provided Values:**
```javascript
{
  user,           // Current user object or null
  loading,        // Authentication check loading state
  signup,         // Signup function
  login,          // Login function
  logout,         // Logout function
  checkAuth       // Re-check authentication status
}
```

**User Object Structure:**
```javascript
{
  _id: string,
  firstName: string,
  lastName: string,
  email: string,
  role: "user" | "admin",
  sellerRating: {
    averageRating: number,
    totalReviews: number
  }
}
```

### SearchContext (`context/SearchContext.js`)
Manages search and filter state for product/order listings.

**Provided Values:**
```javascript
{
  searchTerm,              // Search query string
  setSearchTerm,           // Update search query
  selectedCategory,        // Selected category filter
  setSelectedCategory,     // Update category filter
  selectedCondition,       // Selected condition filter
  setSelectedCondition,    // Update condition filter
  selectedStatus,          // Selected status filter
  setSelectedStatus        // Update status filter
}
```

**Available Categories:**
- Electronics, Books, Furniture, Clothing, Sports & Outdoors, Tools, Home & Kitchen, Other

**Available Conditions:**
- Brand New, Like New, Good, Used, Damaged

**Available Statuses:**
- Product: active, draft, inactive
- Order: pending, confirmed, ready_for_pickup, picked_up, out_for_delivery, delivered, cancelled

---

## 🔌 API Integration

### Authentication Functions (`libs/auth.js`)

| Function | Description |
|----------|-------------|
| `signup({ firstName, lastName, email, password })` | Register new user |
| `login({ email, password })` | Authenticate user |
| `logout()` | End user session |

### Utility Functions (`libs/utlis.js`)

**User Functions:**
| Function | Description |
|----------|-------------|
| `fetchUserProfile(userId)` | Get user profile data |
| `updateUserInfo(userId, data)` | Update user info (JSON) |
| `updateUserInfoWithPicture(userId, data, file)` | Update user with profile picture |
| `fetchUserPreferences()` | Get payment/address preferences |
| `updateUserPreferences(data)` | Update preferences |

**Product Functions:**
| Function | Description |
|----------|-------------|
| `fetchAllProducts()` | Get all products |
| `fetchUserProducts()` | Get current user's products |
| `fetchProductById(productId)` | Get single product |
| `createProduct(formData)` | Create new product |
| `updateProduct(productId, formData)` | Update product |
| `deleteProduct(productId)` | Delete product |

**Order Functions:**
| Function | Description |
|----------|-------------|
| `fetchOrders(role)` | Get orders (buyer/seller) |
| `fetchOrderById(orderId)` | Get single order |
| `createOrder(payload)` | Create new order |
| `updateOrderStatus(orderId, status)` | Update order status |

**Review Functions:**
| Function | Description |
|----------|-------------|
| `createReview(orderId, rating, comment)` | Submit review |
| `skipReview(orderId)` | Skip review for order |
| `getSellerReviews(sellerId, page, limit)` | Get seller's reviews |
| `getReviewByOrder(orderId)` | Get review for order |

---

## 🔐 Authentication Flow

### Registration Flow
1. User enters details on `/signup` (requires @mun.ca email)
2. Password validation (min 6 chars, uppercase, lowercase, number)
3. Backend sends verification email
4. User clicks link to verify at `/verify-email`
5. Account activated, user can login

### Login Flow
1. User enters credentials at `/login`
2. Backend validates and returns JWT token
3. Token stored in `localStorage`
4. `AuthContext` updates user state
5. Redirected to home page (→ `/buy`)

### Session Management
- JWT token stored in `localStorage`
- `AuthContext.checkAuth()` validates token on app load
- Invalid tokens cleared automatically
- Protected routes check auth state before rendering

### Password Reset Flow
1. User requests reset at `/forgot-password`
2. Backend sends reset email with token
3. User clicks link to `/reset-password?token=...`
4. User sets new password
5. Redirected to login

---

## 🧪 Testing

The project uses Jest and React Testing Library for testing.

### Test Structure
```
__tests__/
├── components/          # Component unit tests
│   ├── Navbar.test.jsx
│   ├── ProductCard.test.jsx
│   ├── ProductForm.test.jsx
│   └── ...
├── context/             # Context provider tests
│   ├── AuthContext.test.jsx
│   └── SearchContext.test.jsx
├── libs/                # Utility function tests
│   ├── auth.test.js
│   └── utils.test.js
└── pages/               # Page integration tests
    ├── login.test.jsx
    ├── signup.test.jsx
    ├── buy.test.jsx
    └── ...
```

### Running Tests
```bash
# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run tests with coverage
npm run test:coverage
```

### Configuration Files
- `jest.config.js` – Jest configuration
- `jest.setup.js` – Test environment setup

---

## 🌍 Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | Backend API base URL | `http://localhost:8800` |

Create a `.env.local` file for local development:
```env
NEXT_PUBLIC_API_URL=http://localhost:8800
```

---

## 📊 Order Fulfillment Statuses

### Delivery Orders
```
pending → confirmed → out_for_delivery → delivered
```

### Pickup Orders
```
pending → ready_for_pickup → picked_up
```

### Status Descriptions
| Status | Description |
|--------|-------------|
| `pending` | Order received, awaiting seller action |
| `confirmed` | Seller accepted the order |
| `ready_for_pickup` | Item ready for buyer pickup |
| `picked_up` | Buyer collected the item |
| `out_for_delivery` | Item in transit |
| `delivered` | Item delivered to buyer |
| `cancelled` | Order cancelled |

---

## 🎨 Design System

### Color Palette
The UI uses Tailwind CSS with a slate-based color scheme:
- **Primary:** `slate-900` (buttons, active states)
- **Background:** `slate-50` (page background)
- **Cards:** `white` with `slate-200` borders
- **Text:** `slate-900` (primary), `slate-600` (secondary)
- **Success:** `emerald-600`
- **Warning:** `amber-500`
- **Error:** `rose-600`

### Typography
- **Headings:** Semibold, slate-900
- **Body:** Regular, slate-700
- **Labels:** Small, semibold, slate-500 uppercase

---

## 👥 Contributors
- Olaiya Oluwatomisin – tomisiiiin
- Labib Islam – labib-islam
- Nafiur Rahman – Nafiur-rhyme
- Anya Anya – Chuxs
- Md Minhajul Abedin – Minhajul99
- Yi Zhang – 1mag1ne1
- Yixuan Liu – Yixuan-Liu1

---

## 📝 License

This project is part of a university course project at Memorial University of Newfoundland.
