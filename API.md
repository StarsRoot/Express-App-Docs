# EasyExpressPro API Documentation

## Overview

EasyExpressPro provides a comprehensive REST API for managing express shipping operations. The API is built with NestJS and follows RESTful principles.

## Base URL

- **Development**: `http://localhost:3001`
- **Production**: `https://api.easyexpresspro.com`

## Authentication

All API endpoints require authentication using JWT tokens from Clerk. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

## Rate Limiting

The API implements rate limiting to prevent abuse:

- **General endpoints**: 200 requests per minute
- **Authentication endpoints**: 5 requests per minute
- **Payment endpoints**: 10 requests per 10 seconds
- **Shipment creation**: 20 requests per 10 seconds

## Error Handling

All errors follow a consistent format:

```json
{
  "success": false,
  "error": "Error message",
  "errorCode": "ERROR_CODE",
  "statusCode": 400,
  "timestamp": "2024-01-01T00:00:00.000Z",
  "path": "/api/shipments",
  "method": "POST"
}
```

## Endpoints

### Shipments

#### Create Shipment
```http
POST /api/shipments
```

**Request Body:**
```json
{
  "customerId": "user_123",
  "branchId": "branch_123",
  "pickupAddress": "123 Main St, New York, NY 10001",
  "deliveryAddress": "456 Oak Ave, Brooklyn, NY 11201",
  "price": 25.50,
  "paymentInfo": {
    "method": "card",
    "amount": 25.50,
    "currency": "USD",
    "paid": false
  }
}
```

**Response:**
```json
{
  "id": "shipment_123",
  "customerId": "user_123",
  "branchId": "branch_123",
  "pickupAddress": "123 Main St, New York, NY 10001",
  "deliveryAddress": "456 Oak Ave, Brooklyn, NY 11201",
  "price": 25.50,
  "status": "pending",
  "statusHistory": [
    {
      "status": "pending",
      "timestamp": "2024-01-01T00:00:00.000Z",
      "note": "Shipment created"
    }
  ],
  "paymentInfo": {
    "method": "card",
    "amount": 25.50,
    "currency": "USD",
    "paid": false
  },
  "createdAt": "2024-01-01T00:00:00.000Z",
  "updatedAt": "2024-01-01T00:00:00.000Z"
}
```

#### Get All Shipments
```http
GET /api/shipments
```

**Query Parameters:**
- `status` (optional): Filter by status
- `branchId` (optional): Filter by branch
- `customerId` (optional): Filter by customer
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)

#### Get Shipment by ID
```http
GET /api/shipments/:id
```

#### Update Shipment Status
```http
PATCH /api/shipments/:id/status
```

**Request Body:**
```json
{
  "status": "confirmed",
  "note": "Shipment confirmed by dispatcher"
}
```

#### Get Shipment Statistics
```http
GET /api/shipments/stats
```

**Query Parameters:**
- `branchId` (optional): Filter by branch

**Response:**
```json
{
  "total": 100,
  "byStatus": {
    "pending": 10,
    "confirmed": 15,
    "picked_up": 20,
    "in_transit": 25,
    "delivered": 25,
    "cancelled": 3,
    "failed": 2
  },
  "totalRevenue": 2500.00
}
```

### Users

#### Get All Users
```http
GET /api/users
```

#### Get User by ID
```http
GET /api/users/:id
```

#### Create User
```http
POST /api/users
```

**Request Body:**
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "role": "customer",
  "branchId": "branch_123"
}
```

#### Update User
```http
PATCH /api/users/:id
```

#### Delete User
```http
DELETE /api/users/:id
```

### Branches

#### Get All Branches
```http
GET /api/branches
```

#### Get Branch by ID
```http
GET /api/branches/:id
```

#### Create Branch
```http
POST /api/branches
```

**Request Body:**
```json
{
  "name": "Main Branch",
  "address": "123 Main St, New York, NY 10001",
  "phone": "(555) 123-4567",
  "email": "main@easyexpress.com",
  "status": "active",
  "managerId": "user_123"
}
```

### Payments

#### Create Payment Session
```http
POST /api/payments/create-session
```

**Request Body:**
```json
{
  "shipmentId": "shipment_123"
}
```

**Response:**
```json
{
  "sessionId": "cs_test_123",
  "url": "https://checkout.stripe.com/pay/cs_test_123"
}
```

#### Handle Payment Webhook
```http
POST /api/payments/webhook
```

### Storage

#### Upload File
```http
POST /api/storage/upload
```

**Form Data:**
- `file`: File to upload
- `folder`: Folder name (optional)

#### Generate Presigned URL
```http
POST /api/storage/presigned-url
```

**Request Body:**
```json
{
  "fileName": "image.jpg",
  "mimeType": "image/jpeg",
  "folder": "proof-of-delivery"
}
```

#### Upload Proof of Delivery
```http
POST /api/storage/proof-of-delivery
```

**Form Data:**
- `file`: Image file

### Notifications

#### Get User Notifications
```http
GET /api/notifications
```

#### Update Notification Preferences
```http
PATCH /api/notifications/preferences
```

**Request Body:**
```json
{
  "emailNotifications": true,
  "pushNotifications": true,
  "shipmentUpdates": true,
  "paymentNotifications": true,
  "marketingEmails": false
}
```

#### Subscribe to Push Notifications
```http
POST /api/notifications/push-subscribe
```

**Request Body:**
```json
{
  "endpoint": "https://fcm.googleapis.com/fcm/send/...",
  "keys": {
    "p256dh": "key1",
    "auth": "key2"
  }
}
```

### Webhooks

#### Clerk Webhook
```http
POST /api/webhooks/clerk
```

Handles user creation and updates from Clerk.

## WebSocket Events

Connect to WebSocket at: `ws://localhost:3001/shipments`

### Events

#### Connection
```json
{
  "event": "connected",
  "data": {
    "message": "Connected to real-time updates",
    "user": {
      "id": "user_123",
      "email": "user@example.com",
      "role": "customer"
    }
  }
}
```

#### Shipment Updates
```json
{
  "event": "shipment_update",
  "data": {
    "shipmentId": "shipment_123",
    "status": "confirmed",
    "previousStatus": "pending",
    "note": "Shipment confirmed",
    "updatedBy": "user_123",
    "timestamp": "2024-01-01T00:00:00.000Z"
  }
}
```

#### Notifications
```json
{
  "event": "notification",
  "data": {
    "title": "Shipment Update",
    "message": "Your shipment has been confirmed",
    "type": "info",
    "timestamp": "2024-01-01T00:00:00.000Z"
  }
}
```

### WebSocket Methods

#### Join Shipment Room
```javascript
socket.emit('join_shipment', { shipmentId: 'shipment_123' });
```

#### Leave Shipment Room
```javascript
socket.emit('leave_shipment', { shipmentId: 'shipment_123' });
```

#### Ping
```javascript
socket.emit('ping');
```

## Status Codes

- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `409` - Conflict
- `429` - Too Many Requests
- `500` - Internal Server Error

## Data Models

### Shipment Status Flow
```
pending → confirmed → picked_up → in_transit → delivered
    ↓         ↓           ↓           ↓
cancelled  cancelled  cancelled  cancelled
    ↓
  failed → picked_up (retry)
```

### User Roles
- `super_admin`: Full system access
- `branch_manager`: Branch management
- `dispatcher`: Shipment coordination
- `driver`: Delivery operations
- `customer`: Basic access

### Payment Methods
- `card`: Credit/Debit card
- `digital_wallet`: Digital wallet
- `cash`: Cash payment
- `bank_transfer`: Bank transfer

## Examples

### Creating a Shipment with Payment

```javascript
// 1. Create shipment
const shipmentResponse = await fetch('/api/shipments', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    customerId: 'user_123',
    branchId: 'branch_123',
    pickupAddress: '123 Main St, New York, NY 10001',
    deliveryAddress: '456 Oak Ave, Brooklyn, NY 11201',
    price: 25.50,
    paymentInfo: {
      method: 'card',
      amount: 25.50,
      currency: 'USD',
      paid: false,
    },
  }),
});

const shipment = await shipmentResponse.json();

// 2. Create payment session
const paymentResponse = await fetch('/api/payments/create-session', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    shipmentId: shipment.data.id,
  }),
});

const payment = await paymentResponse.json();

// 3. Redirect to Stripe Checkout
window.location.href = payment.data.url;
```

### WebSocket Connection

```javascript
import { io } from 'socket.io-client';

const socket = io('http://localhost:3001/shipments', {
  auth: {
    token: 'your-jwt-token',
  },
});

socket.on('connected', (data) => {
  console.log('Connected:', data);
});

socket.on('shipment_update', (data) => {
  console.log('Shipment update:', data);
});

socket.on('notification', (data) => {
  console.log('Notification:', data);
});

// Join a shipment room
socket.emit('join_shipment', { shipmentId: 'shipment_123' });
```