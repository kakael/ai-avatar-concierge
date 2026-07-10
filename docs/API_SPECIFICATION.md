# AI Avatar Concierge - API Specification

## Overview

This document provides comprehensive specifications for all REST and GraphQL APIs of the AI Avatar Concierge system. The API follows RESTful principles with JSON payloads and includes GraphQL endpoints for complex queries.

## API Versioning

Current API Version: **v1**
Base URL: `https://api.ai-avatar-concierge.com/api/v1`

All endpoints use versioning in the URL path to support backward compatibility.

---

## Authentication & Authorization

### OAuth 2.0 Flow

```
Authorization Code Flow
GET /oauth/authorize?client_id={id}&redirect_uri={uri}&state={state}&scope={scopes}
↓
POST /oauth/token with authorization_code
↓
Returns: access_token, refresh_token, expires_in
```

### JWT Token Structure

```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_id",
    "user_id": "uuid",
    "email": "user@example.com",
    "roles": ["user", "admin"],
    "permissions": ["read:conversations", "write:bookings"],
    "iat": 1234567890,
    "exp": 1234571490
  }
}
```

### Authentication Methods

```
1. OAuth 2.0 (Google, Apple)
2. Email/Password with MFA
3. Phone Number with OTP
4. Biometric (fingerprint, face)
5. API Keys (backend services)
6. Guest Mode (limited access)
```

### Request Headers

```
Authorization: Bearer {jwt_token}
X-API-Key: {api_key} (for service-to-service)
Content-Type: application/json
Accept: application/json
X-Request-ID: {unique_request_id}
X-Client-Version: {client_version}
```

---

## Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input parameters",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format"
      }
    ],
    "timestamp": "2026-07-10T20:00:00Z",
    "request_id": "req_123456"
  }
}
```

### Common Error Codes

```
AUTHENTICATION_FAILED    - Invalid credentials
AUTHORIZATION_DENIED     - Insufficient permissions
VALIDATION_ERROR         - Invalid input data
NOT_FOUND                - Resource not found
CONFLICT                 - Resource already exists
RATE_LIMIT_EXCEEDED      - Too many requests
INTERNAL_SERVER_ERROR    - Server error
SERVICE_UNAVAILABLE      - Service temporarily down
```

---

## API Endpoints

### 1. Authentication API

#### POST /auth/register

Register a new user account.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password",
  "full_name": "John Doe",
  "phone_number": "+1234567890",
  "language_preference": "en",
  "accept_terms": true
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "email": "user@example.com",
    "full_name": "John Doe",
    "created_at": "2026-07-10T20:00:00Z",
    "email_verified": false
  }
}
```

---

#### POST /auth/login

Authenticate user with email and password.

**Request:**
```json
{
  "email": "user@example.com",
  "password": "secure_password",
  "device_id": "device_uuid",
  "remember_me": true
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "refresh_token_value",
    "expires_in": 3600,
    "user": {
      "user_id": "uuid",
      "email": "user@example.com",
      "full_name": "John Doe",
      "roles": ["user"],
      "avatar_preference": "avatar_1"
    },
    "mfa_required": false
  }
}
```

---

#### POST /auth/mfa/send

Send MFA code for multi-factor authentication.

**Request:**
```json
{
  "user_id": "uuid",
  "method": "email|sms|totp"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "mfa_session_id": "mfa_session_uuid",
    "method": "email",
    "delivery_target": "user@***ple.com",
    "expires_in": 300
  }
}
```

---

#### POST /auth/mfa/verify

Verify MFA code.

**Request:**
```json
{
  "mfa_session_id": "mfa_session_uuid",
  "code": "123456"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "access_token": "eyJhbGciOiJIUzI1NiIs...",
    "refresh_token": "refresh_token_value",
    "expires_in": 3600
  }
}
```

---

#### POST /auth/refresh-token

Refresh expired access token.

**Request:**
```json
{
  "refresh_token": "refresh_token_value"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "access_token": "new_access_token",
    "expires_in": 3600
  }
}
```

---

#### POST /auth/logout

Logout user and invalidate tokens.

**Response (200):**
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

---

#### POST /auth/oauth/authorize

OAuth authorization endpoint.

**Parameters:**
```
client_id: {client_id}
redirect_uri: {callback_url}
response_type: code
scope: openid profile email
state: {random_state}
```

---

#### POST /auth/oauth/token

Exchange authorization code for tokens.

**Request:**
```json
{
  "grant_type": "authorization_code",
  "code": "authorization_code",
  "client_id": "client_id",
  "client_secret": "client_secret",
  "redirect_uri": "https://callback.url"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "access_token": "access_token",
    "refresh_token": "refresh_token",
    "expires_in": 3600,
    "token_type": "Bearer"
  }
}
```

---

### 2. User Profile API

#### GET /users/profile

Get current user profile.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "email": "user@example.com",
    "full_name": "John Doe",
    "phone_number": "+1234567890",
    "avatar_url": "https://storage.url/avatar.jpg",
    "language_preference": "en",
    "timezone": "Africa/Nairobi",
    "created_at": "2026-07-01T00:00:00Z",
    "updated_at": "2026-07-10T20:00:00Z",
    "preferences": {
      "voice_preference": "male_deep",
      "voice_speed": 1.0,
      "enable_notifications": true,
      "enable_analytics": true
    }
  }
}
```

---

#### PUT /users/profile

Update user profile.

**Request:**
```json
{
  "full_name": "Jane Doe",
  "phone_number": "+0987654321",
  "timezone": "Africa/Nairobi",
  "language_preference": "sw"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user_id": "uuid",
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### POST /users/profile/avatar

Upload user profile avatar.

**Request:** (multipart/form-data)
```
avatar: {image_file}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "avatar_url": "https://storage.url/avatar_new.jpg",
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /users/preferences

Get user preferences and settings.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "voice_preference": "female_warm",
    "voice_speed": 1.0,
    "avatar_preference": "avatar_1",
    "theme": "dark",
    "language": "en",
    "enable_captions": true,
    "enable_notifications": true,
    "notification_channels": ["email", "push"],
    "privacy_settings": {
      "share_analytics": true,
      "share_conversation_history": false,
      "data_retention_days": 90
    }
  }
}
```

---

#### PUT /users/preferences

Update user preferences.

**Request:**
```json
{
  "voice_preference": "male_warm",
  "voice_speed": 1.2,
  "theme": "light",
  "enable_captions": false
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

### 3. Conversations API

#### POST /conversations

Start a new conversation.

**Request:**
```json
{
  "avatar_id": "avatar_1",
  "context": {
    "service_type": "booking",
    "location": {"latitude": 0.0, "longitude": 0.0}
  },
  "initial_message": "Hello, I need to book an appointment"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "conversation_id": "conv_uuid",
    "avatar_id": "avatar_1",
    "session_token": "session_token",
    "created_at": "2026-07-10T20:00:00Z",
    "status": "active"
  }
}
```

---

#### GET /conversations/{conversation_id}

Get conversation details.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "conversation_id": "conv_uuid",
    "user_id": "user_uuid",
    "avatar_id": "avatar_1",
    "status": "active",
    "duration": 180,
    "message_count": 10,
    "satisfaction_rating": null,
    "created_at": "2026-07-10T20:00:00Z",
    "updated_at": "2026-07-10T20:03:00Z"
  }
}
```

---

#### GET /conversations/{conversation_id}/messages

Get conversation messages.

**Query Parameters:**
```
limit: 50 (default)
offset: 0
sort: asc|desc
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "messages": [
      {
        "message_id": "msg_uuid",
        "conversation_id": "conv_uuid",
        "sender": "user|avatar",
        "content": "Hello, I need help",
        "type": "text|image|document",
        "metadata": {
          "emotion": "neutral",
          "confidence": 0.95
        },
        "timestamp": "2026-07-10T20:00:00Z"
      }
    ],
    "pagination": {
      "total": 100,
      "limit": 50,
      "offset": 0
    }
  }
}
```

---

#### POST /conversations/{conversation_id}/messages

Send message in conversation.

**Request:**
```json
{
  "content": "I need to book an appointment",
  "type": "text",
  "metadata": {
    "voice_input": false,
    "user_emotion": "neutral"
  }
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "message_id": "msg_uuid",
    "user_message": {
      "content": "I need to book an appointment",
      "timestamp": "2026-07-10T20:00:00Z"
    },
    "avatar_response": {
      "content": "I'd be happy to help you book an appointment...",
      "emotion": "helpful",
      "avatar_animation": "smiling",
      "audio_url": "https://storage.url/audio.mp3",
      "timestamp": "2026-07-10T20:00:01Z"
    }
  }
}
```

---

#### POST /conversations/{conversation_id}/end

End a conversation.

**Request:**
```json
{
  "satisfaction_rating": 4,
  "feedback": "Very helpful assistant"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "conversation_id": "conv_uuid",
    "status": "ended",
    "ended_at": "2026-07-10T20:03:00Z"
  }
}
```

---

#### GET /conversations

List user conversations.

**Query Parameters:**
```
limit: 20 (default)
offset: 0
status: active|ended|archived
sort_by: created_at|updated_at
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "conversations": [
      {
        "conversation_id": "conv_uuid",
        "avatar_id": "avatar_1",
        "status": "ended",
        "duration": 180,
        "satisfaction_rating": 5,
        "created_at": "2026-07-10T20:00:00Z"
      }
    ],
    "pagination": {
      "total": 50,
      "limit": 20,
      "offset": 0
    }
  }
}
```

---

### 4. Avatar API

#### GET /avatars

List available avatars.

**Query Parameters:**
```
category: receptionist|concierge|guide
language: en|sw
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "avatars": [
      {
        "avatar_id": "avatar_1",
        "name": "Sarah",
        "description": "Professional receptionist",
        "category": "receptionist",
        "thumbnail_url": "https://storage.url/avatar1_thumb.jpg",
        "model_url": "https://storage.url/avatar1_model.glb",
        "supported_languages": ["en", "sw"],
        "voices": [
          {
            "voice_id": "voice_1",
            "name": "Sarah - English",
            "language": "en",
            "gender": "female",
            "accent": "neutral"
          }
        ],
        "animations": {
          "idle": ["breathing", "blinking"],
          "expressions": ["happy", "neutral", "concerned"]
        }
      }
    ]
  }
}
```

---

#### GET /avatars/{avatar_id}

Get avatar details.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "avatar_id": "avatar_1",
    "name": "Sarah",
    "description": "Professional receptionist avatar",
    "category": "receptionist",
    "model_url": "https://storage.url/avatar1_model.glb",
    "animations": {
      "facial_expressions": ["happy", "sad", "neutral", "surprised", "confused"],
      "head_movements": ["nod", "shake", "tilt"],
      "hand_gestures": ["welcome", "point", "thumbs_up"],
      "body_movements": ["idle_breathe", "attentive", "thinking"]
    },
    "voices": [
      {
        "voice_id": "voice_1",
        "name": "Sarah - English",
        "language": "en",
        "gender": "female",
        "accent": "neutral",
        "audio_sample": "https://storage.url/voice_sample.mp3"
      }
    ]
  }
}
```

---

#### GET /avatars/{avatar_id}/animations

Get avatar animation details.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "idle_animations": [
      {
        "animation_id": "anim_1",
        "name": "breathing",
        "duration": 4.0,
        "loop": true
      }
    ],
    "expression_animations": [
      {
        "emotion": "happy",
        "expression_id": "expr_1",
        "duration": 1.5,
        "blend_shape_targets": ["mouth_smile", "eyes_squint"]
      }
    ],
    "gesture_animations": [
      {
        "gesture": "welcome",
        "gesture_id": "gest_1",
        "duration": 2.0
      }
    ]
  }
}
```

---

### 5. Booking API

#### POST /bookings

Create a new booking.

**Request:**
```json
{
  "service_id": "service_uuid",
  "service_type": "appointment|ticket|reservation",
  "scheduled_date": "2026-07-20",
  "scheduled_time": "14:30",
  "duration_minutes": 30,
  "location_id": "location_uuid",
  "participants": [
    {
      "name": "John Doe",
      "email": "john@example.com",
      "phone": "+1234567890"
    }
  ],
  "notes": "Please call 5 minutes before"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "booking_id": "booking_uuid",
    "reference_number": "BOOK-20260710-001",
    "status": "confirmed",
    "confirmation_code": "ABC123XYZ",
    "scheduled_date": "2026-07-20",
    "scheduled_time": "14:30",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /bookings/{booking_id}

Get booking details.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "booking_id": "booking_uuid",
    "reference_number": "BOOK-20260710-001",
    "status": "confirmed",
    "service": {
      "service_id": "service_uuid",
      "name": "Consultation",
      "price": 5000,
      "currency": "KES"
    },
    "scheduled_date": "2026-07-20",
    "scheduled_time": "14:30",
    "duration_minutes": 30,
    "location": {
      "location_id": "location_uuid",
      "name": "Downtown Office",
      "address": "123 Main St"
    },
    "confirmation_sent_to": "john@example.com",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /bookings

List user bookings.

**Query Parameters:**
```
status: confirmed|pending|cancelled
service_type: appointment|ticket|reservation
date_from: 2026-07-01
date_to: 2026-07-31
limit: 20
offset: 0
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "bookings": [
      {
        "booking_id": "booking_uuid",
        "reference_number": "BOOK-20260710-001",
        "status": "confirmed",
        "service_name": "Consultation",
        "scheduled_date": "2026-07-20",
        "scheduled_time": "14:30"
      }
    ],
    "pagination": {
      "total": 15,
      "limit": 20,
      "offset": 0
    }
  }
}
```

---

#### PUT /bookings/{booking_id}

Update booking.

**Request:**
```json
{
  "scheduled_date": "2026-07-22",
  "scheduled_time": "15:00",
  "notes": "Updated notes"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "booking_id": "booking_uuid",
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### DELETE /bookings/{booking_id}

Cancel booking.

**Request:**
```json
{
  "reason": "Change of plans",
  "refund": true
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "booking_id": "booking_uuid",
    "status": "cancelled",
    "cancelled_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /bookings/{booking_id}/availability

Check availability for rescheduling.

**Query Parameters:**
```
date_from: 2026-07-15
date_to: 2026-07-31
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "available_slots": [
      {
        "date": "2026-07-20",
        "time": "09:00",
        "duration_minutes": 30
      },
      {
        "date": "2026-07-20",
        "time": "14:30",
        "duration_minutes": 30
      }
    ]
  }
}
```

---

### 6. Payment API

#### POST /payments/process

Process a payment.

**Request:**
```json
{
  "booking_id": "booking_uuid",
  "amount": 5000,
  "currency": "KES",
  "payment_method": "card|mobile_money|bank_transfer",
  "payment_details": {
    "card_token": "tok_visa",
    "phone_number": "+254712345678"
  }
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "payment_id": "payment_uuid",
    "booking_id": "booking_uuid",
    "status": "completed",
    "amount": 5000,
    "currency": "KES",
    "transaction_id": "TXN123456",
    "receipt_url": "https://storage.url/receipt.pdf",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /payments/{payment_id}

Get payment details.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "payment_id": "payment_uuid",
    "status": "completed",
    "amount": 5000,
    "currency": "KES",
    "transaction_id": "TXN123456",
    "payment_method": "card",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### POST /payments/{payment_id}/refund

Request payment refund.

**Request:**
```json
{
  "reason": "Service not provided",
  "amount": 5000
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "refund_id": "refund_uuid",
    "payment_id": "payment_uuid",
    "status": "pending",
    "amount": 5000,
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

### 7. Knowledge Base API

#### GET /knowledge-base/faqs

Get frequently asked questions.

**Query Parameters:**
```
category: general|booking|payment|technical
language: en|sw
search: keyword
limit: 20
offset: 0
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "faqs": [
      {
        "faq_id": "faq_uuid",
        "question": "How do I book an appointment?",
        "answer": "You can book an appointment through...",
        "category": "booking",
        "language": "en",
        "helpful_count": 150,
        "created_at": "2026-06-01T00:00:00Z"
      }
    ],
    "pagination": {
      "total": 250,
      "limit": 20,
      "offset": 0
    }
  }
}
```

---

#### GET /knowledge-base/products

Get product information.

**Query Parameters:**
```
category: all|electronics|services|events
language: en|sw
search: keyword
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "products": [
      {
        "product_id": "prod_uuid",
        "name": "Premium Consultation",
        "description": "60-minute professional consultation",
        "category": "services",
        "price": 5000,
        "currency": "KES",
        "image_url": "https://storage.url/product.jpg",
        "availability": true
      }
    ]
  }
}
```

---

#### GET /knowledge-base/services

Get service information.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "services": [
      {
        "service_id": "service_uuid",
        "name": "Appointment Booking",
        "description": "Professional appointment scheduling",
        "duration_minutes": 30,
        "price": 0,
        "available": true,
        "locations": ["location_1", "location_2"]
      }
    ]
  }
}
```

---

#### GET /knowledge-base/search

Full-text search knowledge base.

**Query Parameters:**
```
q: search query (required)
type: faq|product|service|all
language: en|sw
limit: 50
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "results": [
      {
        "type": "faq",
        "id": "faq_uuid",
        "title": "How to book?",
        "snippet": "You can book through the app or website...",
        "relevance_score": 0.95
      }
    ],
    "total_results": 150
  }
}
```

---

### 8. Analytics API

#### GET /analytics/conversations

Get conversation analytics.

**Query Parameters:**
```
date_from: 2026-07-01
date_to: 2026-07-10
group_by: day|week|month
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "total_conversations": 1250,
    "avg_duration_seconds": 180,
    "avg_satisfaction_rating": 4.5,
    "completion_rate": 0.92,
    "most_common_intents": [
      {
        "intent": "booking",
        "count": 450,
        "percentage": 36.0
      }
    ],
    "language_distribution": {
      "en": 800,
      "sw": 450
    },
    "daily_breakdown": [
      {
        "date": "2026-07-10",
        "count": 150,
        "avg_duration": 185
      }
    ]
  }
}
```

---

#### GET /analytics/user-engagement

Get user engagement metrics.

**Query Parameters:**
```
date_from: 2026-07-01
date_to: 2026-07-10
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "active_users": 450,
    "new_users": 75,
    "returning_users": 375,
    "avg_sessions_per_user": 2.8,
    "feature_usage": {
      "booking": 0.65,
      "faq": 0.45,
      "product_search": 0.32
    }
  }
}
```

---

#### GET /analytics/bookings

Get booking analytics.

**Query Parameters:**
```
date_from: 2026-07-01
date_to: 2026-07-10
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "total_bookings": 280,
    "completed_bookings": 260,
    "cancelled_bookings": 20,
    "conversion_rate": 0.22,
    "total_revenue": 1400000,
    "currency": "KES",
    "avg_booking_value": 5000,
    "popular_services": [
      {
        "service_id": "service_1",
        "name": "Consultation",
        "count": 120,
        "revenue": 600000
      }
    ]
  }
}
```

---

### 9. Admin API

#### POST /admin/faqs

Create FAQ entry.

**Request:**
```json
{
  "question": "How do I reset my password?",
  "answer": "Click on 'Forgot Password' on the login page...",
  "category": "general",
  "language": "en",
  "display_order": 1
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "faq_id": "faq_uuid",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### PUT /admin/faqs/{faq_id}

Update FAQ entry.

**Request:**
```json
{
  "question": "How do I reset my password?",
  "answer": "Updated answer text..."
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "faq_id": "faq_uuid",
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### DELETE /admin/faqs/{faq_id}

Delete FAQ entry.

**Response (200):**
```json
{
  "success": true,
  "message": "FAQ deleted successfully"
}
```

---

#### POST /admin/knowledge-base/upload

Upload knowledge base document.

**Request:** (multipart/form-data)
```
file: {pdf_document}
category: services|products|general
language: en|sw
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "document_id": "doc_uuid",
    "filename": "services_guide.pdf",
    "extracted_content": "...",
    "status": "processing",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /admin/users

List all users (admin only).

**Query Parameters:**
```
limit: 50
offset: 0
role: user|admin|moderator
status: active|inactive|banned
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "user_id": "user_uuid",
        "email": "user@example.com",
        "full_name": "John Doe",
        "role": "user",
        "status": "active",
        "created_at": "2026-07-01T00:00:00Z",
        "last_login": "2026-07-10T20:00:00Z"
      }
    ],
    "pagination": {
      "total": 1500,
      "limit": 50,
      "offset": 0
    }
  }
}
```

---

#### PUT /admin/users/{user_id}/role

Update user role.

**Request:**
```json
{
  "role": "admin",
  "permissions": ["read:analytics", "write:faqs"]
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "user_id": "user_uuid",
    "role": "admin",
    "updated_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### POST /admin/content/avatars/upload

Upload new avatar model.

**Request:** (multipart/form-data)
```
model: {3d_model_file.glb}
thumbnail: {image_file}
name: "New Avatar"
category: receptionist
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "avatar_id": "avatar_new_uuid",
    "name": "New Avatar",
    "model_url": "https://storage.url/avatar_new.glb",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

### 10. Notifications API

#### POST /notifications/subscribe

Subscribe to notifications.

**Request:**
```json
{
  "notification_type": "booking_reminder|promotion|system",
  "channels": ["email", "push", "sms"],
  "frequency": "always|daily|weekly"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "subscription_id": "sub_uuid",
    "created_at": "2026-07-10T20:00:00Z"
  }
}
```

---

#### GET /notifications

Get user notifications.

**Query Parameters:**
```
type: all|booking|promotion|system
status: unread|read|archived
limit: 20
offset: 0
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "notification_id": "notif_uuid",
        "type": "booking_reminder",
        "title": "Appointment Reminder",
        "message": "Your appointment is in 1 hour",
        "status": "unread",
        "created_at": "2026-07-10T19:00:00Z"
      }
    ],
    "unread_count": 3
  }
}
```

---

## GraphQL Schema

### Query

```graphql
type Query {
  # User queries
  user(id: ID!): User
  me: User
  userPreferences: UserPreferences
  
  # Conversation queries
  conversation(id: ID!): Conversation
  conversations(limit: Int, offset: Int): ConversationConnection
  
  # Avatar queries
  avatars(category: String, language: String): [Avatar!]!
  avatar(id: ID!): Avatar
  
  # Booking queries
  booking(id: ID!): Booking
  bookings(status: String, limit: Int): BookingConnection
  availableSlots(serviceId: ID!, dateFrom: Date, dateTo: Date): [TimeSlot!]!
  
  # Knowledge base queries
  faqs(category: String, language: String, search: String): [FAQ!]!
  products(category: String, search: String): [Product!]!
  services: [Service!]!
  search(query: String!, type: String): SearchResult!
  
  # Analytics queries (admin)
  analyticsConversations(dateFrom: Date, dateTo: Date): ConversationAnalytics
  analyticsBookings(dateFrom: Date, dateTo: Date): BookingAnalytics
}

type Mutation {
  # Auth mutations
  register(input: RegisterInput!): AuthPayload!
  login(input: LoginInput!): AuthPayload!
  logout: Boolean!
  refreshToken(refreshToken: String!): TokenPayload!
  
  # User mutations
  updateProfile(input: UpdateProfileInput!): User!
  updatePreferences(input: UpdatePreferencesInput!): UserPreferences!
  
  # Conversation mutations
  startConversation(input: StartConversationInput!): Conversation!
  sendMessage(input: SendMessageInput!): Message!
  endConversation(conversationId: ID!, feedback: String): Conversation!
  
  # Booking mutations
  createBooking(input: CreateBookingInput!): Booking!
  updateBooking(id: ID!, input: UpdateBookingInput!): Booking!
  cancelBooking(id: ID!, reason: String): Booking!
  
  # Payment mutations
  processPayment(input: ProcessPaymentInput!): Payment!
  refundPayment(paymentId: ID!, reason: String): Refund!
  
  # Admin mutations
  createFAQ(input: CreateFAQInput!): FAQ!
  updateFAQ(id: ID!, input: UpdateFAQInput!): FAQ!
  deleteFAQ(id: ID!): Boolean!
  uploadKnowledgeBase(input: UploadKBInput!): Document!
  
  # Notification mutations
  subscribe(input: NotificationSubscriptionInput!): Subscription!
}

type Subscription {
  # Real-time conversation updates
  messageReceived(conversationId: ID!): Message!
  conversationStatusChanged(conversationId: ID!): Conversation!
  
  # Real-time notifications
  notificationReceived: Notification!
  
  # Admin real-time updates
  analyticsUpdated: AnalyticsUpdate!
}
```

---

## Rate Limiting

API endpoints are rate-limited per user and IP:

```
Standard Rate Limits:
- Authenticated User: 1,000 requests/hour
- Unauthenticated: 100 requests/hour
- Admin Endpoints: 5,000 requests/hour
- File Upload: 10 MB/minute per user
```

Response Headers:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1234567890
```

---

## Webhook Events

The system supports webhooks for event notifications:

```json
{
  "event_type": "booking.confirmed|booking.cancelled|payment.completed|conversation.ended",
  "timestamp": "2026-07-10T20:00:00Z",
  "data": {
    "object_id": "booking_uuid",
    "object_type": "booking",
    "status": "confirmed"
  }
}
```

---

## API Status Codes

```
200 OK - Successful request
201 Created - Resource created
204 No Content - Successful request with no response body
400 Bad Request - Invalid input
401 Unauthorized - Authentication required
403 Forbidden - Insufficient permissions
404 Not Found - Resource not found
409 Conflict - Resource already exists
422 Unprocessable Entity - Validation failed
429 Too Many Requests - Rate limit exceeded
500 Internal Server Error - Server error
503 Service Unavailable - Service down
```

---

## Pagination

All list endpoints use standard pagination:

```json
{
  "data": [...],
  "pagination": {
    "total": 1000,
    "limit": 50,
    "offset": 0,
    "has_more": true,
    "next_offset": 50
  }
}
```

---

## API Documentation & Testing

- **Swagger/OpenAPI**: Available at `/api/v1/docs`
- **Postman Collection**: Available in `docs/postman-collection.json`
- **GraphQL Playground**: Available at `/graphql`

---

This API specification provides a comprehensive guide for integrating with the AI Avatar Concierge system. All endpoints follow RESTful principles and return consistent JSON responses with proper error handling.
