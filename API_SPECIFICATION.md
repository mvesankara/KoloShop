# KoloShop - API Specification & Integration Guide

## Table des matières

1. [Authentication Endpoints](#authentication-endpoints)
2. [User Management](#user-management)
3. [Products & Catalog](#products--catalog)
4. [Orders](#orders)
5. [Payments](#payments)
6. [Delivery](#delivery)
7. [Sellers](#sellers)
8. [Admin API](#admin-api)
9. [Webhooks](#webhooks)
10. [Error Handling](#error-handling)

---

## Base URL

```
Development: http://localhost:3000/api/v1
Staging: https://staging-api.koloshop.cm/api/v1
Production: https://api.koloshop.cm/api/v1
```

## Authentication

All endpoints (except auth) require:
```
Header: Authorization: Bearer {accessToken}
```

---

## Authentication Endpoints

### POST /auth/login

Login user and get tokens.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "deviceFingerprint": "device-id-hash"
}
```

**Response (200):**
```json
{
  "data": {
    "accessToken": "eyJhbGc...",
    "user": {
      "id": "1234",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "userType": "customer"
    }
  }
}
```

**Errors:**
- `401 UNAUTHORIZED` - Invalid credentials
- `429 TOO_MANY_REQUESTS` - Brute force detected

---

### POST /auth/register

Register new user.

**Request:**
```json
{
  "email": "user@example.com",
  "phone": "+237601234567",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "userType": "customer",
  "acceptTerms": true
}
```

**Response (201):**
```json
{
  "data": {
    "userId": "1234",
    "message": "Check email for verification"
  }
}
```

---

### POST /auth/refresh

Refresh access token.

**Request:**
```
Cookie: refreshToken=xyz123
```

**Response (200):**
```json
{
  "data": {
    "accessToken": "eyJhbGc..."
  }
}
```

---

## User Management

### GET /users/profile

Get current user profile.

**Response (200):**
```json
{
  "data": {
    "id": "1234",
    "email": "user@example.com",
    "phone": "+237601234567",
    "firstName": "John",
    "lastName": "Doe",
    "avatarUrl": "https://...",
    "countryCode": "CM",
    "timezone": "Africa/Douala",
    "kyc": {
      "status": "verified",
      "verifiedAt": "2024-01-15T10:00:00Z"
    }
  }
}
```

---

### PATCH /users/profile

Update user profile.

**Request:**
```json
{
  "firstName": "Jean",
  "avatarUrl": "https://...",
  "timezone": "Africa/Yaoundé"
}
```

**Response (200):**
```json
{
  "data": { ...updated user }
}
```

---

### POST /users/verify-email

Verify email with OTP code.

**Request:**
```json
{
  "email": "user@example.com",
  "otp": "123456"
}
```

**Response (200):**
```json
{
  "data": { "message": "Email verified" }
}
```

---

### POST /users/addresses

Create delivery address.

**Request:**
```json
{
  "type": "home",
  "label": "My Home",
  "fullAddress": "123 Main Street, Douala, Cameroon",
  "latitude": 3.8667,
  "longitude": 11.5167,
  "isDefault": true
}
```

**Response (201):**
```json
{
  "data": {
    "id": "addr-123",
    "type": "home",
    "label": "My Home",
    "fullAddress": "123 Main Street, Douala, Cameroon",
    "latitude": 3.8667,
    "longitude": 11.5167,
    "isDefault": true
  }
}
```

---

## Products & Catalog

### GET /products

List products with filters.

**Query Parameters:**
```
?page=1
&limit=20
&category=electronics
&priceMin=10000
&priceMax=500000
&sortBy=newest|popular|rating|price
&search=iphone
```

**Response (200):**
```json
{
  "data": {
    "products": [
      {
        "id": "prod-1",
        "name": "iPhone 15",
        "slug": "iphone-15",
        "description": "...",
        "basePrice": 450000,
        "discountPrice": 400000,
        "discountPercentage": 11,
        "rating": 4.5,
        "ratingCount": 234,
        "thumbnailUrl": "https://...",
        "seller": {
          "id": "seller-1",
          "businessName": "TechStore",
          "rating": 4.8,
          "verificationStatus": "verified"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 500,
      "hasMore": true
    }
  }
}
```

---

### GET /products/{productId}

Get product details.

**Response (200):**
```json
{
  "data": {
    "id": "prod-1",
    "name": "iPhone 15",
    "description": "...",
    "basePrice": 450000,
    "discountPrice": 400000,
    "discountPercentage": 11,
    "stockQuantity": 50,
    "rating": 4.5,
    "ratingCount": 234,
    "images": [
      { "url": "https://...", "alt": "Front view" },
      { "url": "https://...", "alt": "Back view" }
    ],
    "variants": [
      {
        "id": "var-1",
        "name": "Color",
        "value": "Black",
        "priceAdjustment": 0,
        "stockQuantity": 20
      }
    ],
    "seller": {
      "id": "seller-1",
      "businessName": "TechStore",
      "avatarUrl": "https://...",
      "rating": 4.8,
      "reviewCount": 1200,
      "location": "Douala, CM"
    },
    "reviews": [
      {
        "id": "review-1",
        "rating": 5,
        "title": "Excellent product",
        "comment": "Fast delivery, great quality",
        "author": "John D.",
        "createdAt": "2024-01-10T14:30:00Z"
      }
    ]
  }
}
```

---

## Orders

### POST /orders

Create new order.

**Request:**
```json
{
  "items": [
    {
      "productId": "prod-1",
      "variantId": "var-1",
      "quantity": 2
    }
  ],
  "deliveryAddressId": "addr-1",
  "sellerId": "seller-1",
  "paymentMethod": "mobile_money",
  "customerNotes": "Please handle with care"
}
```

**Response (201):**
```json
{
  "data": {
    "id": "order-123",
    "orderNumber": "ORD-2024-001234",
    "status": "pending",
    "subtotal": 800000,
    "deliveryFee": 5000,
    "taxAmount": 128000,
    "total": 933000,
    "currency": "XAF",
    "items": [
      {
        "productId": "prod-1",
        "productName": "iPhone 15",
        "quantity": 2,
        "unitPrice": 400000,
        "totalPrice": 800000
      }
    ]
  }
}
```

---

### GET /orders

List user orders.

**Query Parameters:**
```
?page=1
&limit=10
&status=pending|confirmed|shipped|delivered|cancelled
&sortBy=newest|oldest
```

**Response (200):**
```json
{
  "data": {
    "orders": [
      {
        "id": "order-123",
        "orderNumber": "ORD-2024-001234",
        "status": "delivered",
        "total": 933000,
        "createdAt": "2024-01-15T10:00:00Z",
        "deliveredAt": "2024-01-18T14:30:00Z",
        "seller": {
          "id": "seller-1",
          "businessName": "TechStore"
        }
      }
    ],
    "pagination": { ... }
  }
}
```

---

### GET /orders/{orderId}

Get order details with tracking.

**Response (200):**
```json
{
  "data": {
    "id": "order-123",
    "orderNumber": "ORD-2024-001234",
    "status": "shipped",
    "statusHistory": [
      { "status": "pending", "timestamp": "2024-01-15T10:00:00Z" },
      { "status": "confirmed", "timestamp": "2024-01-15T10:15:00Z" },
      { "status": "shipped", "timestamp": "2024-01-17T08:00:00Z" }
    ],
    "items": [ ... ],
    "delivery": {
      "status": "in_transit",
      "currentLocation": {
        "latitude": 3.8667,
        "longitude": 11.5167
      },
      "estimatedDeliveryTime": "2024-01-18T18:00:00Z",
      "rider": {
        "id": "rider-1",
        "name": "Brice M.",
        "phone": "+237601234567",
        "riderRating": 4.9,
        "vehicle": "Motorbike"
      }
    },
    "payment": {
      "status": "completed",
      "method": "mobile_money",
      "provider": "moov",
      "amount": 933000
    }
  }
}
```

---

### PATCH /orders/{orderId}/cancel

Cancel order (before shipped).

**Response (200):**
```json
{
  "data": { "message": "Order cancelled successfully" }
}
```

---

## Payments

### POST /payments/initiate

Initiate payment for order.

**Request:**
```json
{
  "orderId": "order-123",
  "paymentMethodId": "method-1",
  "paymentMethod": "mobile_money",
  "provider": "moov"
}
```

**Response (200):**
```json
{
  "data": {
    "paymentId": "pay-123",
    "status": "processing",
    "amount": 933000,
    "currency": "XAF",
    "provider": "moov",
    "message": "USSD prompt sent to your phone"
  }
}
```

---

### GET /payments/{paymentId}

Check payment status.

**Response (200):**
```json
{
  "data": {
    "id": "pay-123",
    "status": "completed",
    "amount": 933000,
    "orderId": "order-123",
    "provider": "moov",
    "providerTransactionId": "TXN-2024-00123",
    "completedAt": "2024-01-15T10:05:30Z"
  }
}
```

---

### POST /payments/webhook/moov

Moov payment webhook (provider → platform).

**Request:**
```json
{
  "id": "webhook-123",
  "event": "payment.completed",
  "externalId": "pay-123",
  "status": "success",
  "amount": 933000,
  "timestamp": "2024-01-15T10:05:30Z",
  "signature": "hmac-sha256-signature"
}
```

**Response (200):**
```json
{
  "data": { "message": "Webhook processed" }
}
```

---

## Delivery

### GET /deliveries/{orderId}

Get delivery tracking info.

**Response (200):**
```json
{
  "data": {
    "id": "delivery-1",
    "orderId": "order-123",
    "status": "in_transit",
    "rider": {
      "id": "rider-1",
      "name": "Brice M.",
      "phone": "+237601234567",
      "rating": 4.9,
      "photoUrl": "https://..."
    },
    "currentLocation": {
      "latitude": 3.8700,
      "longitude": 11.5200,
      "lastUpdate": "2024-01-18T15:30:00Z"
    },
    "estimatedDeliveryTime": "2024-01-18T18:00:00Z",
    "distanceRemaining": 2.5,
    "progressPercent": 85,
    "deliveryAddress": {
      "fullAddress": "123 Main Street, Douala",
      "landmark": "Near the green pharmacy"
    },
    "tracking": [
      { "event": "picked_up", "timestamp": "2024-01-17T09:00:00Z" },
      { "event": "in_transit", "timestamp": "2024-01-17T10:00:00Z" },
      { "event": "location_update", "timestamp": "2024-01-18T15:30:00Z" }
    ]
  }
}
```

---

### POST /deliveries/{orderId}/complete

Mark delivery as completed (rider confirmatio).

**Request:**
```json
{
  "otp": "123456",
  "photoUrl": "https://...",
  "signatureUrl": "https://..."
}
```

**Response (200):**
```json
{
  "data": { "message": "Delivery completed" }
}
```

---

## Sellers

### POST /sellers/register

Register as seller.

**Request:**
```json
{
  "businessName": "TechStore",
  "businessRegistration": "CM123456789",
  "countryCode": "CM",
  "city": "Douala",
  "address": "123 Commercial Street",
  "mobileMoneyOperator": "moov",
  "mobileMoneyNumber": "+237601234567",
  "identityDocumentUrl": "https://...",
  "proofOfAddressUrl": "https://..."
}
```

**Response (201):**
```json
{
  "data": {
    "sellerId": "seller-1",
    "status": "pending_verification",
    "message": "Your business is under verification"
  }
}
```

---

### GET /sellers/dashboard

Get seller dashboard stats.

**Response (200):**
```json
{
  "data": {
    "totalOrders": 1234,
    "totalRevenue": 45000000,
    "averageRating": 4.8,
    "reviewCount": 567,
    "activeProducts": 89,
    "pendingOrders": 12,
    "thisMonthRevenue": 2500000,
    "thisMonthOrders": 120,
    "topProducts": [
      {
        "productId": "prod-1",
        "name": "iPhone 15",
        "sales": 250,
        "revenue": 100000000
      }
    ]
  }
}
```

---

### POST /sellers/products

Create product (seller).

**Request:**
```json
{
  "name": "iPhone 15",
  "description": "Latest iPhone model",
  "basePrice": 450000,
  "discountPrice": 400000,
  "stockQuantity": 50,
  "categoryId": "cat-1",
  "images": [
    { "url": "https://...", "alt": "Front" }
  ],
  "variants": [
    {
      "name": "Color",
      "value": "Black",
      "priceAdjustment": 0,
      "stockQuantity": 20
    }
  ],
  "metaTitle": "Buy iPhone 15 Online",
  "metaDescription": "Latest Apple iPhone 15"
}
```

**Response (201):**
```json
{
  "data": {
    "productId": "prod-1",
    "status": "active"
  }
}
```

---

### GET /sellers/payouts

List payouts.

**Query Parameters:**
```
?page=1
&status=pending|processing|completed|failed
```

**Response (200):**
```json
{
  "data": {
    "payouts": [
      {
        "id": "payout-1",
        "amount": 2000000,
        "status": "completed",
        "requestedAt": "2024-01-20T10:00:00Z",
        "completedAt": "2024-01-22T15:30:00Z",
        "paymentMethod": "mobile_money"
      }
    ]
  }
}
```

---

## Admin API

### GET /admin/users

List all users (admin only).

**Response (200):**
```json
{
  "data": {
    "users": [
      {
        "id": "user-1",
        "email": "user@example.com",
        "userType": "customer",
        "status": "active",
        "kycStatus": "verified",
        "createdAt": "2024-01-10T10:00:00Z",
        "lastLoginAt": "2024-01-20T15:30:00Z"
      }
    ]
  }
}
```

---

### POST /admin/disputes/{orderId}/resolve

Resolve order dispute.

**Request:**
```json
{
  "decision": "refund_customer|refund_seller",
  "reason": "Product defective"
}
```

**Response (200):**
```json
{
  "data": { "message": "Dispute resolved" }
}
```

---

## Webhooks

### Incoming Webhooks (Platform receives)

- `payment.completed` - Payment successful
- `payment.failed` - Payment failed
- `rider.arrived` - Rider arrived at location
- `delivery.completed` - Delivery completed

### Outgoing Webhooks (Platform sends to sellers)

Configure at: `POST /webhooks/endpoints`

**Events:**
- `order.created` - New order
- `order.confirmed` - Payment confirmed
- `order.shipped` - Order shipped
- `order.cancelled` - Order cancelled
- `review.created` - New review

**Webhook Payload:**
```json
{
  "id": "webhook-123",
  "event": "order.created",
  "data": {
    "orderId": "order-123",
    "customerId": "user-1",
    "total": 933000
  },
  "timestamp": "2024-01-15T10:00:00Z",
  "signature": "sha256=hmac-signature"
}
```

---

## Error Handling

### Error Response Format

All errors follow this format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable message",
    "details": {
      "field": "Additional context"
    },
    "requestId": "uuid"
  }
}
```

### Common Error Codes

| Code | Status | Description |
|------|--------|-------------|
| `VALIDATION_ERROR` | 400 | Invalid input parameters |
| `UNAUTHORIZED` | 401 | Missing/invalid auth token |
| `FORBIDDEN` | 403 | Insufficient permissions |
| `NOT_FOUND` | 404 | Resource not found |
| `CONFLICT` | 409 | Resource already exists |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTERNAL_ERROR` | 500 | Internal server error |
| `PAYMENT_FAILED` | 400 | Payment processing failed |
| `DELIVERY_UNAVAILABLE` | 400 | Delivery not available for area |
| `INVENTORY_ERROR` | 400 | Product out of stock |

---

**API Specification v1.0**  
Last Updated: May 27, 2026
