# Student TradeHub - Backend API Documentation

## Table of Contents
- [Overview](#overview)
- [Base URL](#base-url)
- [Authentication](#authentication)
- [Error Handling](#error-handling)
- [Data Models](#data-models)
- [API Endpoints](#api-endpoints)
  - [Authentication](#authentication-endpoints)
  - [Users](#user-endpoints)
  - [Products](#product-endpoints)
  - [Orders](#order-endpoints)
  - [Reviews](#review-endpoints)
- [Setup & Installation](#setup--installation)
- [File Structure](#file-structure)
- [Quick Start Examples](#quick-start-examples)
- [Testing](#testing)
- [Security Features](#security-features)
- [Development Notes](#development-notes)

---

## Overview

Student TradeHub is a closed marketplace platform for students to buy, sell, and exchange items safely within their institution. Access is restricted to users with valid Memorial University email addresses (@mun.ca).

**Technology Stack:**
- Backend: Node.js, Express v5
- Database: MongoDB with Mongoose
- Authentication: JWT (JSON Web Tokens)
- Email Service: Nodemailer
- File Upload: Multer
- Password Hashing: bcrypt
- Testing: Jest, Supertest

---

## Base URL

```
http://localhost:8800/api
```

---

## Authentication

The API uses JWT (JSON Web Token) for authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your_token>
```

### Token Structure
```javascript
{
  userId: "user_id",
  userEmail: "user@mun.ca",
  role: "user" | "admin"
}
```

---

## Error Handling

### Error Response Format

```json
{
  "message": "Error description"
}
```

### HTTP Status Codes

| Code | Status | Description |
|------|--------|-------------|
| `200` | OK | Success |
| `201` | Created | Resource created |
| `400` | Bad Request | Invalid input |
| `401` | Unauthorized | Authentication failed |
| `403` | Forbidden | Access denied or account blocked |
| `404` | Not Found | Resource not found |
| `500` | Internal Server Error | Server error |

### Common Error Examples

**Missing Token:**
```json
{
  "message": "Authentication failed"
}
```

**Blocked Account:**
```json
{
  "message": "Account is blocked or no longer exists"
}
```

**Email Not Verified:**
```json
{
  "message": "Please verify your email before logging in. Check your inbox for the verification link."
}
```

---

## Data Models

### User Model

```javascript
{
  firstName: String (required, 2-50 chars),
  lastName: String (required, 2-50 chars),
  email: String (required, unique, @mun.ca only),
  password: String (required, min 8 chars, hashed),
  role: String (user/admin, default: user),
  status: String (active/blocked, default: active),
  isEmailVerified: Boolean (default: false),
  emailVerificationToken: String (hashed),
  emailVerificationExpires: Date,
  resetPasswordToken: String (hashed),
  resetPasswordExpires: Date,
  sellerRating: {
    averageRating: Number (0-5, default: 0),
    totalReviews: Number (default: 0)
  },
  paymentMethod: {
    cardHolderName: String,
    cardNumber: String,
    expiryDate: String,
    cvv: String (not returned in responses),
    type: String (Credit/Debit)
  },
  defaultPaymentMethod: {
    cardHolderName: String,
    last4: String,
    expiryDate: String,
    type: String
  },
  defaultDeliveryAddress: {
    line1: String,
    line2: String,
    city: String,
    state: String,
    postalCode: String,
    country: String (default: Canada)
  },
  pickupAddress: Object (same structure as deliveryAddress),
  productList: [Product IDs],
  createdAt: Date,
  updatedAt: Date
}
```

### Product Model

```javascript
{
  name: String (required, 2-100 chars),
  description: String (max 1000 chars),
  price: Number (required, min 0),
  category: String (required),
  quantity: Number (required, min 0, default: 0),
  imageUrl: String,
  status: String (active/inactive/draft, default: active),
  condition: String (Brand New/Like New/Good/Used/Damaged, required),
  createdBy: User ID (required),
  createdAt: Date,
  updatedAt: Date
}
```

**Product Status Rules:**
- `active`: Visible to all users
- `inactive`: Only visible to creator, admin, or users with orders for this product (set automatically when quantity reaches 0)
- `draft`: Only visible to creator and admin

### Order Model

```javascript
{
  orderNumber: String (auto-generated, unique),
  product: Product ID (required),
  seller: User ID (required),
  buyer: User ID (required),
  quantity: Number (required, min 1),
  amount: Number (required, min 0),
  paymentStatus: String (pending/paid/failed/refunded, default: pending),
  fulfillmentStatus: String (
    pending/confirmed/ready_for_pickup/out_for_delivery/delivered/picked_up/cancelled,
    default: pending
  ),
  deliveryType: String (pickup/deliver, required),
  deliveryDetails: {
    pickupAddress: Object,
    shippingAddress: Object
  },
  paymentMethod: {
    cardHolderName: String,
    last4: String,
    expiryDate: String,
    type: String
  },
  notes: String,
  isReviewed: Boolean (default: false),
  reviewSkipped: Boolean (default: false),
  createdAt: Date,
  updatedAt: Date
}
```

**Fulfillment Status Pipelines:**
- **Delivery**: pending → confirmed → out_for_delivery → delivered
- **Pickup**: pending → ready_for_pickup → picked_up
- **Both**: Can be cancelled only from pending status

### Review Model

```javascript
{
  order: Order ID (required, unique),
  seller: User ID (required),
  buyer: User ID (required),
  product: Product ID (required),
  rating: Number (required, 1-5),
  comment: String (max 500 chars),
  createdAt: Date,
  updatedAt: Date
}
```

---

## API Endpoints

### Authentication Endpoints

#### 1. User Signup
**POST** `/auth/signup`

Creates a new user account and sends email verification link.

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@mun.ca",
  "password": "securePassword123"
}
```

**Response (201):**
```json
{
  "message": "Account created successfully! Please check your email to verify your account before logging in.",
  "email": "john.doe@mun.ca"
}
```

**Errors:**
- 400: Missing fields or email already exists
- 500: Error sending verification email

---

#### 2. Verify Email
**POST** `/auth/verify-email`

Verifies user email with token from verification link.

**Request:**
```json
{
  "token": "verification_token_from_email"
}
```

**Response (200):**
```json
{
  "message": "Email verified successfully! You can now log in to your account."
}
```

**Errors:**
- 400: Invalid or expired token

---

#### 3. User Login
**POST** `/auth/login`

Authenticates user and returns JWT token.

**Request:**
```json
{
  "email": "john.doe@mun.ca",
  "password": "securePassword123"
}
```

**Response (200):**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Errors:**
- 400: Missing fields
- 401: Wrong email or password
- 403: Email not verified or account blocked

---

#### 4. Forgot Password
**POST** `/auth/forgot-password`

Sends password reset link to user's email.

**Request:**
```json
{
  "email": "john.doe@mun.ca"
}
```

**Response (200):**
```json
{
  "message": "If an account with that email exists, a password reset link has been sent."
}
```

---

#### 5. Reset Password
**POST** `/auth/reset-password`

Resets user password using token from reset email.

**Request:**
```json
{
  "token": "reset_token_from_email",
  "newPassword": "newSecurePassword456"
}
```

**Response (200):**
```json
{
  "message": "Password has been reset successfully. You can now log in with your new password."
}
```

**Errors:**
- 400: Invalid/expired token or password too short

---

#### 6. Get Current User
**GET** `/auth/me`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "_id": "user_id",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@mun.ca",
  "role": "user",
  "status": "active",
  "sellerRating": {
    "averageRating": 4.5,
    "totalReviews": 10
  },
  "productList": [],
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

---

### User Endpoints

#### 1. Get All Users (Admin)
**GET** `/users`

**Headers:** `Authorization: Bearer <admin_token>`

**Response (200):** Array of users with populated products

---

#### 2. Get User by ID
**GET** `/users/:id`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "_id": "user_id",
  "firstName": "John",
  "lastName": "Doe",
  "email": "john.doe@mun.ca",
  "sellerRating": {
    "averageRating": 4.5,
    "totalReviews": 10
  },
  "productList": [...],
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

---

#### 3. Update User
**PUT** `/users/:id`

Users can only update their own profile.

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane.smith@mun.ca",
  "currentPassword": "oldPassword123",
  "password": "newPassword456"
}
```

**Response (200):**
```json
{
  "message": "User updated successfully.",
  "user": { ... }
}
```

**Notes:**
- Current password required when changing password
- Email change requires uniqueness validation

---

#### 4. Delete User
**DELETE** `/users/:id`

Users can only delete their own account.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "message": "User deleted successfully."
}
```

---

#### 5. Add/Update Payment Method
**POST** `/users/payment/add`

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "cardHolderName": "John Doe",
  "cardNumber": "1234567890123456",
  "expiryDate": "12/25",
  "cvv": "123",
  "type": "Credit"
}
```

**Response (201):**
```json
{
  "message": "Payment method added",
  "data": {
    "cardHolderName": "John Doe",
    "last4": "3456",
    "expiryDate": "12/25",
    "type": "Credit"
  }
}
```

---

#### 6. Get User Preferences
**GET** `/users/me/preferences`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "paymentMethod": {
    "cardHolderName": "John Doe",
    "last4": "3456",
    "expiryDate": "12/25",
    "type": "Credit"
  },
  "deliveryAddress": {
    "line1": "123 Main St",
    "city": "St. John's",
    "state": "NL",
    "postalCode": "A1A 1A1",
    "country": "Canada"
  },
  "pickupAddress": null
}
```

---

#### 7. Update User Preferences
**PATCH** `/users/me/preferences`

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "deliveryAddress": {
    "line1": "456 New St",
    "city": "St. John's",
    "state": "NL",
    "postalCode": "A1B 2C3",
    "country": "Canada"
  },
  "pickupAddress": {
    "line1": "University Drive",
    "city": "St. John's",
    "state": "NL"
  }
}
```

**Response (200):**
```json
{
  "message": "Preferences updated successfully.",
  "preferences": { ... }
}
```

---

#### 8. Update User Status (Admin)
**PATCH** `/users/:id/status`

**Headers:** `Authorization: Bearer <admin_token>`

**Request:**
```json
{
  "status": "blocked"
}
```

**Response (200):**
```json
{
  "message": "User status updated successfully.",
  "user": { ... }
}
```

**Notes:**
- When user is blocked, all their active/draft products are set to inactive

---

#### 9. Get User Activity Summary (Admin)
**GET** `/users/:id/activity`

**Headers:** `Authorization: Bearer <admin_token>`

**Response (200):**
```json
{
  "productCount": 5,
  "boughtCount": 12,
  "soldCount": 8
}
```

---

### Product Endpoints

#### 1. Get All Products
**GET** `/products`

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `search` - Search by name
- `category` - Filter by category (or use `cat`)
- `status` - Filter by status (all/active/inactive/draft)
- `condition` - Filter by condition
- `minPrice` / `maxPrice` - Price range
- `inStock` - Filter in-stock items (true/false)
- `createdBy` - Filter by creator user ID
- `sort` - Sort order (newest/price/-price/name/-name)
- `page` / `limit` - Pagination (default: page=1, limit=12)

**Example:**
```
GET /products?category=Electronics&minPrice=100&maxPrice=1000&status=active&page=1&limit=12
```

**Response (200):**
```json
{
  "products": [
    {
      "_id": "product_id",
      "name": "Gaming Laptop",
      "price": 1200,
      "category": "Electronics",
      "quantity": 1,
      "status": "active",
      "condition": "Like New",
      "imageUrl": "public/images/uuid.jpg",
      "createdBy": {
        "_id": "user_id",
        "firstName": "John",
        "lastName": "Doe",
        "sellerRating": {
          "averageRating": 4.5,
          "totalReviews": 10
        }
      },
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "pagination": {
    "total": 25,
    "page": 1,
    "limit": 12,
    "pages": 3,
    "hasNext": true
  }
}
```

**Notes:**
- Returns only active products by default (unless user is creator/admin)
- Inactive products visible to: creator, admin, users with orders for that product

---

#### 2. Get Product by ID
**GET** `/products/:pid`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "product": {
    "_id": "product_id",
    "name": "Gaming Laptop",
    "description": "High-performance gaming laptop",
    "price": 1200,
    "category": "Electronics",
    "quantity": 1,
    "status": "active",
    "condition": "Like New",
    "imageUrl": "public/images/uuid.jpg",
    "createdBy": {
      "_id": "user_id",
      "firstName": "John",
      "lastName": "Doe",
      "pickupAddress": { ... },
      "defaultDeliveryAddress": { ... },
      "sellerRating": {
        "averageRating": 4.5,
        "totalReviews": 10
      }
    },
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Errors:**
- 403: Not authorized to view inactive/draft product
- 404: Product not found

---

#### 3. Create Product
**POST** `/products/new`

**Headers:** `Authorization: Bearer <token>`, `Content-Type: multipart/form-data`

**Form Data:**
- `name` (required)
- `description`
- `price` (required)
- `category` (required)
- `quantity` (required)
- `condition` (required: Brand New/Like New/Good/Used/Damaged)
- `status` (active/draft, default: active)
- `image` (required, max 500KB, PNG/JPEG/JPG)

**Response (201):**
```json
{
  "product": {
    "_id": "product_id",
    "name": "Gaming Laptop",
    "price": 1200,
    "quantity": 1,
    "status": "active",
    "imageUrl": "public/images/uuid.jpg",
    ...
  }
}
```

**Errors:**
- 400: Missing image, cannot create with inactive status
- 500: File upload error

---

#### 4. Update Product
**PATCH** `/products/:pid`

Only creator can update their products.

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "name": "Updated Gaming Laptop",
  "price": 1100,
  "description": "Updated description",
  "quantity": 2,
  "status": "active"
}
```

**Response (200):**
```json
{
  "product": { ... }
}
```

**Errors:**
- 400: Cannot edit sold out products, cannot set status to inactive manually
- 401: Not creator of product

---

#### 5. Delete Product
**DELETE** `/products/:pid`

Only creator can delete their products.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "message": "Product deleted successfully."
}
```

**Errors:**
- 400: Cannot delete sold out products
- 401: Not creator of product

---

#### 6. Update Product Status (Admin)
**PATCH** `/products/:pid/status`

**Headers:** `Authorization: Bearer <admin_token>`

**Request:**
```json
{
  "status": "inactive"
}
```

**Response (200):**
```json
{
  "message": "Product status updated successfully.",
  "product": { ... }
}
```

---

#### 7. Get Product Suggestions
**GET** `/products/suggest`

**Query Parameters:**
- `q` - Search query

**Example:**
```
GET /products/suggest?q=lap
```

**Response (200):**
```json
[
  { "_id": "id1", "name": "Laptop" },
  { "_id": "id2", "name": "Laptop Stand" }
]
```

---

### Order Endpoints

#### 1. Create Order
**POST** `/orders`

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "productId": "product_id",
  "quantity": 1,
  "paymentMethod": {
    "cardHolderName": "John Doe",
    "cardNumber": "1234567890123456",
    "expiryDate": "12/25",
    "cvv": "123",
    "type": "Credit"
  },
  "deliveryOption": {
    "type": "deliver",
    "address": {
      "line1": "123 Main St",
      "city": "St. John's",
      "state": "NL",
      "postalCode": "A1A 1A1",
      "country": "Canada"
    }
  },
  "savePaymentMethod": true,
  "saveDeliveryAddress": true
}
```

**Response (201):**
```json
{
  "message": "Order placed successfully.",
  "order": {
    "_id": "order_id",
    "orderNumber": "ORD-20241204-143025123-ABC1",
    "product": "product_id",
    "seller": "seller_id",
    "buyer": "buyer_id",
    "quantity": 1,
    "amount": 1200,
    "paymentStatus": "paid",
    "fulfillmentStatus": "pending",
    "deliveryType": "deliver",
    ...
  }
}
```

**Notes:**
- Automatically reduces product quantity
- Sets product to inactive when quantity reaches 0
- Cannot order your own product
- Requires payment method
- For delivery: requires shipping address
- For pickup: uses seller's pickup address

**Errors:**
- 400: Missing product/quantity, invalid quantity, out of stock, cannot buy own product, missing addresses
- 404: Product not found

---

#### 2. Get User Orders
**GET** `/orders`

**Headers:** `Authorization: Bearer <token>`

**Query Parameters:**
- `role` - Filter by role (buyer/seller, default: buyer)

**Response (200):**
```json
{
  "orders": [
    {
      "_id": "order_id",
      "orderNumber": "ORD-20241204-143025123-ABC1",
      "product": { ... },
      "buyer": { ... },
      "seller": { ... },
      "quantity": 1,
      "amount": 1200,
      "paymentStatus": "paid",
      "fulfillmentStatus": "pending",
      "deliveryType": "deliver",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

#### 3. Get Order by ID
**GET** `/orders/:id`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "order": { ... }
}
```

**Access Control:**
- Buyer, seller, or admin can view

---

#### 4. Update Order Status (Seller)
**PATCH** `/orders/:id/status`

Only seller can update fulfillment status.

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "status": "confirmed"
}
```

**Response (200):**
```json
{
  "order": { ... }
}
```

**Status Transitions:**
- **Delivery**: pending → confirmed → out_for_delivery → delivered
- **Pickup**: pending → ready_for_pickup → picked_up
- **Cancel**: Only from pending status

**Errors:**
- 400: Invalid status, cannot move backwards in pipeline
- 403: Only seller can update

---

#### 5. Get Orders for User (Admin)
**GET** `/orders/admin/user/:id`

**Headers:** `Authorization: Bearer <admin_token>`

**Query Parameters:**
- `role` - Filter by role (buyer/seller)

---

#### 6. Get All Orders (Admin)
**GET** `/orders/admin/all`

**Headers:** `Authorization: Bearer <admin_token>`

**Query Parameters:**
- `search` - Search by order number, buyer/seller name, product name
- `paymentStatus` - Filter by payment status
- `fulfillmentStatus` - Filter by fulfillment status
- `deliveryType` - Filter by delivery type
- `page` / `limit` - Pagination (default: page=1, limit=50)

**Response (200):**
```json
{
  "orders": [...],
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 50,
    "pages": 2,
    "hasNext": true
  }
}
```

---

#### 7. Update Order Status (Admin)
**PATCH** `/orders/:id/status/admin`

**Headers:** `Authorization: Bearer <admin_token>`

**Request:**
```json
{
  "paymentStatus": "refunded",
  "fulfillmentStatus": "cancelled"
}
```

**Response (200):**
```json
{
  "message": "Order status updated successfully.",
  "order": { ... }
}
```

---

#### 8. Get Order Statistics (Admin)
**GET** `/orders/admin/stats`

**Headers:** `Authorization: Bearer <admin_token>`

**Response (200):**
```json
{
  "totalOrders": 150,
  "totalRevenue": 125000,
  "pendingOrders": 20,
  "completedOrders": 120,
  "cancelledOrders": 10,
  "paymentStatusBreakdown": {
    "paid": 140,
    "pending": 5,
    "refunded": 5
  },
  "fulfillmentStatusBreakdown": {
    "pending": 20,
    "confirmed": 15,
    "delivered": 100,
    "picked_up": 20,
    "cancelled": 10
  }
}
```

---

### Review Endpoints

#### 1. Create Review
**POST** `/reviews`

Buyers can review completed orders.

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "orderId": "order_id",
  "rating": 5,
  "comment": "Great seller, item as described!"
}
```

**Response (201):**
```json
{
  "message": "Review submitted successfully!",
  "review": {
    "_id": "review_id",
    "order": "order_id",
    "seller": "seller_id",
    "buyer": "buyer_id",
    "product": "product_id",
    "rating": 5,
    "comment": "Great seller, item as described!",
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

**Errors:**
- 400: Missing fields, invalid rating, already reviewed, order not completed
- 403: Not order buyer

---

#### 2. Get Seller Reviews
**GET** `/reviews/seller/:sellerId`

**Query Parameters:**
- `page` / `limit` - Pagination (default: page=1, limit=10)

**Response (200):**
```json
{
  "reviews": [
    {
      "_id": "review_id",
      "buyer": {
        "firstName": "John",
        "lastName": "Doe"
      },
      "product": {
        "name": "Gaming Laptop",
        "imageUrl": "..."
      },
      "rating": 5,
      "comment": "Great seller!",
      "createdAt": "2024-01-01T00:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 15,
    "pages": 2
  }
}
```

---

#### 3. Get Pending Reviews
**GET** `/reviews/pending`

Get orders awaiting review for current buyer.

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "pendingOrders": [
    {
      "_id": "order_id",
      "orderNumber": "ORD-...",
      "product": { ... },
      "seller": { ... },
      "fulfillmentStatus": "delivered",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ]
}
```

---

#### 4. Skip Review
**POST** `/reviews/skip`

Mark that buyer doesn't want to review an order.

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "orderId": "order_id"
}
```

**Response (200):**
```json
{
  "message": "Review skipped."
}
```

---

#### 5. Get Review by Order
**GET** `/reviews/order/:orderId`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "review": { ... }
}
```

**Errors:**
- 403: Can only view own reviews
- 404: Review not found

---

## Setup & Installation

### Prerequisites
- Node.js v16.20.1 or higher
- MongoDB Atlas Account
- Git

### Installation Steps

**1. Clone Repository**
```bash
git clone <repository-url>
cd student-tradehub
```

**2. Install Dependencies**
```bash
npm install
```

**3. Create Environment File**
```bash
touch .env
```

Add to `.env`:
```env
PORT=8800
NODE_ENV=development
DB_USERNAME=your_mongodb_username
DB_PASSWORD=your_mongodb_password
JWT_SECRET=your_super_secure_secret_key_here
EMAIL_USER=your_gmail_address
EMAIL_PASSWORD=your_gmail_app_password
FRONTEND_URL=http://localhost:3000
```

**Email Setup (Gmail):**
1. Enable 2-factor authentication on Gmail
2. Generate App Password: Google Account → Security → 2-Step Verification → App passwords
3. Use generated password for EMAIL_PASSWORD

**4. Create Image Directory**
```bash
mkdir -p public/images
```

**5. Start Server**
```bash
npm start
```

Server runs at: `http://localhost:8800`

---

## File Structure

```
student-tradehub/
├── __tests__/                # Test files
│   ├── e2e/                 # End-to-end tests
│   └── setupTestDB.js       # Test database setup
├── controllers/              # Business logic
│   ├── auth.controller.js
│   ├── user.controller.js
│   ├── product.controller.js
│   ├── order.controller.js
│   └── review.controller.js
├── models/                   # Database schemas
│   ├── user.model.js
│   ├── product.model.js
│   ├── order.model.js
│   └── review.model.js
├── routes/                   # API endpoints
│   ├── auth.routes.js
│   ├── user.routes.js
│   ├── products.routes.js
│   ├── order.routes.js
│   └── review.routes.js
├── middlewares/              # Custom middleware
│   ├── auth.middleware.js
│   └── fileUpload.middleware.js
├── utils/                    # Utility functions
│   └── email.util.js        # Email sending
├── public/images/            # Uploaded images
├── app.js                    # Main application
├── jest.config.