# API Design Standard Document
## Mobile-Backend Integration Guidelines

**Version:** 1.0  
**Last Updated:** December 2025  
**Applicable To:** All Backend Teams (PHP, .NET, Python, Java, etc.)

---

## Table of Contents

1. [Introduction](#introduction)
2. [General Principles](#general-principles)
3. [Authentication & Authorization](#authentication--authorization)
4. [User Profile Management](#user-profile-management)
5. [Common API Patterns](#common-api-patterns)
6. [Data Standards](#data-standards)
7. [Error Handling](#error-handling)
8. [Security Requirements](#security-requirements)
9. [Performance Guidelines](#performance-guidelines)
10. [Versioning Strategy](#versioning-strategy)

---

## Introduction

This document establishes the API design standards for all backend services that interface with mobile applications. The goal is to ensure consistency, security, and maintainability across all projects regardless of backend technology.

### Objectives
- Provide consistent API interface for mobile developers
- Reduce integration time for new projects
- Ensure security best practices are followed
- Facilitate onboarding of new team members
- Enable seamless technology migration

---

## General Principles

### Base URL Structure
```
https://api.{domain}.com/{version}/{resource}
```

**Example:**
```
https://api.example.com/v1/users
https://api.example.com/v1/auth/login
```

### HTTP Methods
- **GET** - Retrieve resources (idempotent)
- **POST** - Create new resources or non-idempotent operations
- **PUT** - Update entire resource (idempotent)
- **PATCH** - Partial update of resource
- **DELETE** - Remove resource (idempotent)

### Content Type
All requests and responses should use JSON format:
```
Content-Type: application/json
Accept: application/json
```

### Response Structure
All API responses must follow this standard structure:

**Success Response:**
```json
{
  "success": true,
  "data": {
    // Response data here
  },
  "message": "Operation completed successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Error Response:**
```json
{
  "success": false,
  "error": {
    "code": "AUTH_001",
    "message": "Invalid credentials",
    "details": "The email or password provided is incorrect"
  },
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

## Authentication & Authorization

### Authentication Flow Standards

All authentication must use **OAuth 2.0** principles with **JWT (JSON Web Tokens)**.

#### 1. User Registration

**Endpoint:** `POST /v1/auth/register`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe",
  "phone": "+201234567890",
  "deviceType": "ios",
  "deviceToken": "fcm_or_apns_token"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid-here",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "phone": "+201234567890",
      "isVerified": false,
      "createdAt": "2025-12-07T10:30:00Z"
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIs...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    }
  },
  "message": "Registration successful. Please verify your email.",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Password Requirements:**
- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one number
- At least one special character

---

#### 2. User Login

**Endpoint:** `POST /v1/auth/login`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "deviceType": "android",
  "deviceToken": "fcm_token_here"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid-here",
      "email": "user@example.com",
      "firstName": "John",
      "lastName": "Doe",
      "profileImage": "https://cdn.example.com/images/profile.jpg",
      "isVerified": true
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIs...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    }
  },
  "message": "Login successful",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

#### 3. Token Refresh

**Endpoint:** `POST /v1/auth/refresh`

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIs...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600,
    "tokenType": "Bearer"
  },
  "message": "Token refreshed successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Token Expiry Standards:**
- Access Token: 1 hour (3600 seconds)
- Refresh Token: 30 days
- Tokens should be invalidated on logout

---

#### 4. Forgot Password

**Endpoint:** `POST /v1/auth/forgot-password`

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Password reset instructions sent to your email",
    "resetTokenExpiresIn": 900
  },
  "message": "Password reset email sent",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Security Notes:**
- Always return success even if email doesn't exist (prevent email enumeration)
- Reset token expires in 15 minutes
- Reset link should be one-time use only

---

#### 5. Reset Password

**Endpoint:** `POST /v1/auth/reset-password`

**Request:**
```json
{
  "token": "reset_token_from_email",
  "newPassword": "NewSecurePass123!",
  "confirmPassword": "NewSecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": null,
  "message": "Password reset successfully. Please login with your new password.",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

#### 6. Change Password (Authenticated)

**Endpoint:** `POST /v1/auth/change-password`

**Headers:**
```
Authorization: Bearer {accessToken}
```

**Request:**
```json
{
  "currentPassword": "OldPassword123!",
  "newPassword": "NewSecurePass123!",
  "confirmPassword": "NewSecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": null,
  "message": "Password changed successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

#### 7. Logout

**Endpoint:** `POST /v1/auth/logout`

**Headers:**
```
Authorization: Bearer {accessToken}
```

**Request:**
```json
{
  "deviceToken": "fcm_or_apns_token"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": null,
  "message": "Logged out successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Backend Action:**
- Invalidate access and refresh tokens
- Remove device token from push notification service

---

#### 8. Social Login (Google/Facebook/Apple)

**Endpoint:** `POST /v1/auth/social-login`

**Request:**
```json
{
  "provider": "google",
  "accessToken": "google_access_token_here",
  "deviceType": "ios",
  "deviceToken": "apns_token_here"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid-here",
      "email": "user@gmail.com",
      "firstName": "John",
      "lastName": "Doe",
      "profileImage": "https://graph.facebook.com/picture",
      "provider": "google",
      "isVerified": true
    },
    "tokens": {
      "accessToken": "eyJhbGciOiJIUzI1NiIs...",
      "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
      "expiresIn": 3600,
      "tokenType": "Bearer"
    },
    "isNewUser": false
  },
  "message": "Social login successful",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Supported Providers:**
- google
- facebook
- apple

---

## User Profile Management

### 1. Get User Profile

**Endpoint:** `GET /v1/users/profile`

**Headers:**
```
Authorization: Bearer {accessToken}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid-here",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Doe",
    "phone": "+201234567890",
    "profileImage": "https://cdn.example.com/images/profile.jpg",
    "dateOfBirth": "1990-01-15",
    "gender": "male",
    "address": {
      "street": "123 Main St",
      "city": "Cairo",
      "country": "Egypt",
      "postalCode": "11511"
    },
    "isVerified": true,
    "createdAt": "2025-01-01T10:00:00Z",
    "updatedAt": "2025-12-07T10:30:00Z"
  },
  "message": "Profile retrieved successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

### 2. Update User Profile

**Endpoint:** `PATCH /v1/users/profile`

**Headers:**
```
Authorization: Bearer {accessToken}
```

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "phone": "+201234567890",
  "dateOfBirth": "1990-01-15",
  "gender": "male",
  "address": {
    "street": "456 New St",
    "city": "Cairo",
    "country": "Egypt",
    "postalCode": "11511"
  }
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "uuid-here",
    "email": "user@example.com",
    "firstName": "John",
    "lastName": "Smith",
    "phone": "+201234567890",
    "profileImage": "https://cdn.example.com/images/profile.jpg",
    "dateOfBirth": "1990-01-15",
    "gender": "male",
    "address": {
      "street": "456 New St",
      "city": "Cairo",
      "country": "Egypt",
      "postalCode": "11511"
    },
    "updatedAt": "2025-12-07T10:35:00Z"
  },
  "message": "Profile updated successfully",
  "timestamp": "2025-12-07T10:35:00Z"
}
```

---

### 3. Upload Profile Image

**Endpoint:** `POST /v1/users/profile/image`

**Headers:**
```
Authorization: Bearer {accessToken}
Content-Type: multipart/form-data
```

**Request:**
```
Form Data:
- image: [binary file]
```

**Allowed Formats:** JPG, PNG, WEBP  
**Max Size:** 5MB  
**Recommended Dimensions:** 512x512px

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "imageUrl": "https://cdn.example.com/images/profile_new.jpg",
    "thumbnailUrl": "https://cdn.example.com/images/profile_new_thumb.jpg"
  },
  "message": "Profile image uploaded successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

### 4. Delete Account

**Endpoint:** `DELETE /v1/users/account`

**Headers:**
```
Authorization: Bearer {accessToken}
```

**Request:**
```json
{
  "password": "CurrentPassword123!",
  "reason": "No longer need the service"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": null,
  "message": "Account deleted successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

**Backend Actions:**
- Soft delete user data (keep for 30 days for recovery)
- Invalidate all tokens
- Remove from all active sessions
- Schedule permanent deletion after 30 days

---

## Common API Patterns

### Pagination

All list endpoints must support pagination using the following query parameters:

**Query Parameters:**
- `page` - Page number (default: 1)
- `limit` - Items per page (default: 20, max: 100)
- `sortBy` - Field to sort by (default: createdAt)
- `sortOrder` - Sort direction: asc or desc (default: desc)

**Example Request:**
```
GET /v1/products?page=2&limit=20&sortBy=price&sortOrder=asc
```

**Response Structure:**
```json
{
  "success": true,
  "data": {
    "items": [
      // Array of items
    ],
    "pagination": {
      "currentPage": 2,
      "totalPages": 10,
      "totalItems": 195,
      "itemsPerPage": 20,
      "hasNextPage": true,
      "hasPreviousPage": true
    }
  },
  "message": "Products retrieved successfully",
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

### Search & Filtering

**Query Parameters:**
- `search` - General search query
- `filter[fieldName]` - Filter by specific field

**Example Request:**
```
GET /v1/products?search=laptop&filter[category]=electronics&filter[minPrice]=500&filter[maxPrice]=2000
```

**Response:** Same structure as pagination with filtered results

---

### File Upload

**Endpoint Pattern:** `POST /v1/{resource}/upload`

**General Requirements:**
- Maximum file size: 10MB (configurable per resource)
- Supported formats must be specified per endpoint
- Use multipart/form-data encoding
- Return CDN URLs in response

**Progress Tracking:**
Mobile apps should show upload progress using standard HTTP upload progress events.

---

## Data Standards

### Date & Time Format
All dates must use **ISO 8601** format in **UTC timezone**:
```
2025-12-07T10:30:00Z
```

Mobile apps are responsible for converting to local timezone.

---

### Phone Numbers
Use **E.164** international format:
```
+201234567890
```

---

### Currency & Money
Always use smallest currency unit (cents) as integers:
```json
{
  "price": {
    "amount": 2999,
    "currency": "USD"
  }
}
```
Display value: $29.99

**Supported Currency Codes:** ISO 4217 (USD, EUR, EGP, etc.)

---

### Boolean Values
Always use `true` or `false` (lowercase), never:
- "true" or "false" (strings)
- 1 or 0
- "yes" or "no"

---

### Null Values
Include null fields in response for consistency:
```json
{
  "middleName": null,
  "profileImage": null
}
```

---

### Enumeration Values
Use lowercase strings with underscores:
```json
{
  "status": "pending_approval",
  "orderType": "home_delivery"
}
```

---

## Error Handling

### HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET, PUT, PATCH, DELETE |
| 201 | Created | Successful POST creating new resource |
| 204 | No Content | Successful DELETE with no response body |
| 400 | Bad Request | Invalid request format or parameters |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but not authorized |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource already exists (duplicate) |
| 422 | Unprocessable Entity | Validation errors |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server-side error |
| 503 | Service Unavailable | Maintenance or overload |

---

### Error Response Structure

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {
      "email": ["Email is required", "Email format is invalid"],
      "password": ["Password must be at least 8 characters"]
    }
  },
  "timestamp": "2025-12-07T10:30:00Z"
}
```

---

### Standard Error Codes

**Authentication Errors (AUTH_xxx)**
- `AUTH_001` - Invalid credentials
- `AUTH_002` - Token expired
- `AUTH_003` - Token invalid
- `AUTH_004` - Account not verified
- `AUTH_005` - Account suspended
- `AUTH_006` - Account deleted

**Validation Errors (VAL_xxx)**
- `VAL_001` - Required field missing
- `VAL_002` - Invalid format
- `VAL_003` - Value out of range
- `VAL_004` - Invalid file type
- `VAL_005` - File too large

**Resource Errors (RES_xxx)**
- `RES_001` - Resource not found
- `RES_002` - Resource already exists
- `RES_003` - Resource access denied

**Server Errors (SRV_xxx)**
- `SRV_001` - Internal server error
- `SRV_002` - Service temporarily unavailable
- `SRV_003` - Database connection failed

---

## Security Requirements

### 1. HTTPS Only
All API endpoints must use HTTPS. Reject HTTP requests.

### 2. Authentication Headers
```
Authorization: Bearer {accessToken}
```

### 3. Rate Limiting

**Per User:**
- Authentication endpoints: 5 requests per minute
- General endpoints: 100 requests per minute
- Upload endpoints: 10 requests per minute

**Response Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1638864000
```

**Error Response (429):**
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 60
  },
  "timestamp": "2025-12-07T10:30:00Z"
}
```

### 4. CORS Headers
Backend must set appropriate CORS headers for mobile apps.

### 5. Input Validation
- Validate all input data
- Sanitize to prevent SQL injection and XSS
- Use parameterized queries
- Implement request size limits

### 6. Password Security
- Use bcrypt or Argon2 for hashing
- Minimum 12 salt rounds
- Never store plain text passwords
- Never return passwords in responses

### 7. API Keys (if applicable)
```
X-API-Key: your_api_key_here
```

---

## Performance Guidelines

### 1. Response Time Targets
- Simple queries: < 200ms
- Complex queries: < 500ms
- File uploads: Dependent on file size
- List endpoints: < 300ms

### 2. Compression
Enable GZIP compression for all text responses.

### 3. Caching Headers
Set appropriate cache headers:
```
Cache-Control: private, max-age=3600
ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"
```

### 4. Lazy Loading
For nested resources, provide option to include related data:
```
GET /v1/orders?include=items,customer
```

### 5. Field Selection
Allow clients to specify required fields:
```
GET /v1/users/profile?fields=firstName,lastName,email
```

### 6. Database Optimization
- Use database indexing
- Implement query optimization
- Use connection pooling
- Cache frequently accessed data

---

## Versioning Strategy

### URL Versioning (Recommended)
```
https://api.example.com/v1/users
https://api.example.com/v2/users
```

### Version Lifecycle
- **v1** - Current stable version
- **v2** - New features (parallel support)
- Deprecate old version after 6 months notice

### Deprecation Headers
```
Deprecation: true
Sunset: Sat, 31 Dec 2025 23:59:59 GMT
Link: <https://api.example.com/v2/users>; rel="successor-version"
```

### Breaking Changes
Only introduce in new major versions. Examples:
- Removing endpoints
- Changing response structure
- Removing fields
- Changing authentication method

### Non-Breaking Changes
Can be added to existing versions:
- Adding new endpoints
- Adding optional parameters
- Adding new fields to responses

---

## Implementation Checklist

### For Backend Teams

- [ ] Implement all authentication endpoints as specified
- [ ] Use JWT tokens with specified expiry times
- [ ] Follow standard response structure
- [ ] Implement proper error handling with standard codes
- [ ] Add rate limiting to all endpoints
- [ ] Enable HTTPS only
- [ ] Implement pagination for all list endpoints
- [ ] Add proper logging and monitoring
- [ ] Write API documentation (Swagger/OpenAPI)
- [ ] Implement input validation
- [ ] Add unit and integration tests
- [ ] Set up CORS properly
- [ ] Implement request/response compression
- [ ] Add proper database indexing
- [ ] Set up caching where appropriate

### For Mobile Teams

- [ ] Implement token refresh mechanism
- [ ] Handle all error codes appropriately
- [ ] Show user-friendly error messages
- [ ] Implement retry logic for failed requests
- [ ] Cache responses where appropriate
- [ ] Implement offline mode
- [ ] Handle rate limiting gracefully
- [ ] Validate data before sending
- [ ] Implement request timeout handling
- [ ] Add proper loading indicators
- [ ] Handle file upload progress
- [ ] Implement secure token storage
- [ ] Clear tokens on logout
- [ ] Handle token expiration

---

## Testing Requirements

### Backend Testing
1. **Unit Tests** - Test individual functions
2. **Integration Tests** - Test API endpoints
3. **Security Tests** - Test authentication and authorization
4. **Performance Tests** - Test response times under load
5. **Validation Tests** - Test input validation

### Test Coverage
Minimum 80% code coverage required for all authentication and critical endpoints.

---

## Documentation Requirements

Each backend implementation must provide:

1. **Swagger/OpenAPI Specification** - Interactive API documentation
2. **Postman Collection** - For testing and examples
3. **Authentication Guide** - How to obtain and use tokens
4. **Error Code Reference** - Complete list of error codes
5. **Migration Guide** - For version upgrades

---

## Support & Maintenance

### Issue Reporting
Report API issues to: api-support@example.com

### Change Requests
Submit via company's project management system with:
- Detailed description
- Use case justification
- Impact analysis
- Proposed implementation

### Review Cycle
This document is reviewed quarterly and updated as needed.

---

## Appendix A: Complete Example Flow

### Registration → Login → Profile Update Flow

```
1. Register User
POST /v1/auth/register
→ Returns user object + tokens

2. Verify Email (if required)
GET /v1/auth/verify?token=xxx
→ Marks user as verified

3. Login
POST /v1/auth/login
→ Returns user object + tokens

4. Get Profile
GET /v1/users/profile
Authorization: Bearer {accessToken}
→ Returns complete profile

5. Update Profile
PATCH /v1/users/profile
Authorization: Bearer {accessToken}
→ Returns updated profile

6. Upload Profile Image
POST /v1/users/profile/image
Authorization: Bearer {accessToken}
→ Returns image URLs

7. Refresh Token (when access token expires)
POST /v1/auth/refresh
→ Returns new access token

8. Logout
POST /v1/auth/logout
Authorization: Bearer {accessToken}
→ Invalidates tokens
```

---

## Appendix B: Quick Reference

### Essential Endpoints Summary

| Endpoint | Method | Auth Required | Purpose |
|----------|--------|---------------|---------|
| `/v1/auth/register` | POST | No | Register new user |
| `/v1/auth/login` | POST | No | Login user |
| `/v1/auth/refresh` | POST | No | Refresh access token |
| `/v1/auth/forgot-password` | POST | No | Request password reset |
| `/v1/auth/reset-password` | POST | No | Reset password with token |
| `/v1/auth/change-password` | POST | Yes | Change password |
| `/v1/auth/logout` | POST | Yes | Logout user |
| `/v1/auth/social-login` | POST | No | Social media login |
| `/v1/users/profile` | GET | Yes | Get user profile |
| `/v1/users/profile` | PATCH | Yes | Update user profile |
| `/v1/users/profile/image` | POST | Yes | Upload profile image |
| `/v1/users/account` | DELETE | Yes | Delete user account |

---

**Document End**

*For questions or clarifications, contact the API Standards Committee*