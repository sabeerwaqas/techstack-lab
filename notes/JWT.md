# JWT Authentication — Practical Developer Guide

JWT is one of the most common authentication mechanisms used in modern web applications and APIs.

This guide focuses on the **practical knowledge a backend or backend-heavy full-stack developer needs in day-to-day development**.

---

## Table of Contents

- [1. What is JWT?](#1-what-is-jwt)
- [2. Structure of a JWT](#2-structure-of-a-jwt)
- [3. JWT Header](#3-jwt-header)
- [4. JWT Payload](#4-jwt-payload)
- [5. JWT Signature](#5-jwt-signature)
- [6. How JWT Authentication Works](#6-how-jwt-authentication-works)
- [7. Access Token](#7-access-token)
- [8. Refresh Token](#8-refresh-token)
- [9. Access Token vs Refresh Token](#9-access-token-vs-refresh-token)
- [10. How Token Refresh Works](#10-how-token-refresh-works)
- [11. Where Tokens Are Stored](#11-where-tokens-are-stored)
- [12. How the Client Knows the Token Expired](#12-how-the-client-knows-the-token-expired)
- [13. Refresh Token API](#13-refresh-token-api)
- [14. Axios Interceptor](#14-axios-interceptor)
- [15. What Happens When Refresh Token Expires](#15-what-happens-when-refresh-token-expires)
- [16. JWT vs Session Authentication](#16-jwt-vs-session-authentication)
- [17. JWT vs Access Token](#17-jwt-vs-access-token)
- [18. JWT vs ID Token](#18-jwt-vs-id-token)
- [19. Authentication vs Authorization](#19-authentication-vs-authorization)
- [20. HTTP Status Codes](#20-http-status-codes)
- [21. JWT Security Essentials](#21-jwt-security-essentials)
- [22. Common JWT Mistakes](#22-common-jwt-mistakes)
- [23. Spring Security Mental Model](#23-spring-security-mental-model)
- [24. Practical Authentication API](#24-practical-authentication-api)
- [25. Complete JWT Lifecycle](#25-complete-jwt-lifecycle)
- [26. Key Takeaways](#26-key-takeaways)

---

# 1. What is JWT?

**JWT (JSON Web Token)** is a compact token format commonly used to authenticate requests between a client and backend.

A JWT looks like:

```text
xxxxx.yyyyy.zzzzz
```

It contains three parts:

```text
Header.Payload.Signature
```

JWTs are commonly used to carry information about an authenticated user between the client and server.

For example:

```text
Client
   |
   | Login
   v
Backend
   |
   | Generate JWT
   v
Client
   |
   | Authorization: Bearer <JWT>
   v
Backend
```

> **Important:** JWT is a token format. JWT itself is not the same thing as authentication, access tokens, or refresh tokens.

---

# 2. Structure of a JWT

A JWT consists of three Base64URL-encoded sections separated by dots:

```text
HEADER.PAYLOAD.SIGNATURE
```

For example:

```text
eyJhbGciOiJIUzI1NiJ9
.
eyJzdWIiOiIxMjMiLCJyb2xlIjoiVVNFUiJ9
.
abc123signature
```

The three parts are:

```text
┌─────────────┐
│   Header    │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Payload   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Signature  │
└─────────────┘
```

---

# 3. JWT Header

The header contains information about the token.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

Common fields:

| Field | Meaning |
|---|---|
| `alg` | Signing algorithm |
| `typ` | Token type |

Common algorithms include:

```text
HS256
HS384
HS512

RS256
RS384
RS512

ES256
ES384
ES512
```

---

# 4. JWT Payload

The payload contains **claims**.

Example:

```json
{
  "sub": "123",
  "role": "USER",
  "iat": 1750000000,
  "exp": 1750000900
}
```

Common claims:

| Claim | Meaning |
|---|---|
| `sub` | Subject, usually user ID |
| `iat` | Issued-at time |
| `exp` | Expiration time |
| `iss` | Issuer |
| `aud` | Audience |

Applications can also define custom claims:

```json
{
  "sub": "123",
  "role": "ADMIN",
  "department": "FINANCE"
}
```

### Important

JWT payloads are normally **encoded, not encrypted**.

Anyone who obtains the token can decode the payload.

Therefore, never put sensitive information such as:

```text
passwords
credit card numbers
secret keys
private information
```

inside a JWT payload.

---

# 5. JWT Signature

The signature is used to verify that the JWT has not been modified.

Conceptually:

```text
Signature =
    Sign(
        Base64URL(Header) + "." + Base64URL(Payload),
        Secret/Private Key
    )
```

For example:

```text
Header
   +
Payload
   |
   v
Signing Algorithm
   +
Secret/Private Key
   |
   v
Signature
```

If someone changes:

```json
{
  "role": "USER"
}
```

to:

```json
{
  "role": "ADMIN"
}
```

the original signature will no longer match.

The backend can therefore reject the token.

---

# 6. How JWT Authentication Works

A typical login flow looks like this:

```text
Client
  |
  | POST /auth/login
  | email + password
  v
Backend
  |
  | Validate credentials
  v
User authenticated
  |
  | Generate tokens
  v
┌───────────────────────────────┐
│ Access Token                  │
│ Refresh Token                 │
└───────────────────────────────┘
  |
  v
Client
```

The client then uses the access token for protected APIs.

Example:

```http
GET /api/orders
Authorization: Bearer <access-token>
```

On the backend:

```text
HTTP Request
     |
     v
Extract JWT
     |
     v
Verify Signature
     |
     v
Check Expiration
     |
     v
Extract Claims
     |
     v
Create Authentication
     |
     v
Check Authorization
     |
     v
Controller
```

---

# 7. Access Token

An **access token** is a credential used to access protected resources/APIs.

Example:

```http
GET /api/orders
Authorization: Bearer <access-token>
```

Access tokens are usually:

- Short-lived
- Sent frequently
- Used for API requests
- Associated with an authenticated user
- Limited in scope/permissions

Example lifetime:

```text
Access Token → 15 minutes
```

The exact lifetime depends on the application.

### Mental Model

Think:

> **Access Token = permission to access the API**

---

# 8. Refresh Token

A **refresh token** is used to obtain a new access token after the access token expires.

Example:

```http
POST /api/auth/refresh
```

The refresh token is normally **not sent with every API request**.

Example:

```text
Access Token
    |
    | Used frequently
    v
Protected APIs


Refresh Token
    |
    | Used occasionally
    v
/auth/refresh
```

Refresh tokens are usually:

- Long-lived
- Used less frequently
- Stored more securely
- Revocable
- Associated with a user/session

Example:

```text
Access Token  → 15 minutes
Refresh Token → 7 days
```

### Mental Model

Think:

> **Refresh Token = permission to obtain a new access token**

---

# 9. Access Token vs Refresh Token

Access Token and Refresh Token describe the **purpose** of a token.

They are not JWT types.

| Feature | Access Token | Refresh Token |
|---|---|---|
| Purpose | Access APIs | Get new access token |
| Lifetime | Short | Longer |
| Usage | Frequent | Occasional |
| Sent to APIs | Yes | Usually no |
| Storage | Less sensitive | More sensitive |
| Can be revoked | Yes | Yes |
| Must be JWT | No | No |

A common architecture is:

```text
Access Token
    |
    └── JWT

Refresh Token
    |
    └── Random opaque token
```

Another architecture is:

```text
Access Token
    |
    └── JWT

Refresh Token
    |
    └── JWT
```

Both are possible.

---

# 10. How Token Refresh Works

This is the most important practical flow.

Suppose:

```text
Access Token → 15 minutes
Refresh Token → 7 days
```

The user logs in:

```text
LOGIN
  |
  v
Backend
  |
  ├───────────────┐
  v               v
Access Token   Refresh Token
15 minutes      7 days
  |               |
  v               v
Client        Secure Cookie
```

The client uses the access token:

```text
Client
   |
   | GET /api/orders
   | Authorization: Bearer <access-token>
   v
Backend
```

After 15 minutes:

```text
Client
   |
   | GET /api/orders
   | expired access token
   v
Backend
   |
   v
401 Unauthorized
```

The client then requests a new access token:

```text
Client
   |
   | POST /api/auth/refresh
   v
Backend
   |
   | Validate refresh token
   v
New Access Token
   |
   v
Client
```

The client can then retry the original request:

```text
GET /api/orders
        |
        | 401
        v
POST /api/auth/refresh
        |
        | new access token
        v
GET /api/orders
        |
        v
      200 OK
```

---

# 11. Where Tokens Are Stored

Token storage is an important security decision.

## Access Token

Possible storage options include:

```text
Memory
sessionStorage
localStorage
Cookie
```

For browser applications, keeping a short-lived access token in memory can reduce the exposure associated with persistent JavaScript-accessible storage.

## Refresh Token

A common browser architecture is:

```text
HttpOnly
Secure
SameSite
Cookie
```

Example:

```http
Set-Cookie: refreshToken=abc123;
            HttpOnly;
            Secure;
            SameSite=Strict
```

### HttpOnly

JavaScript cannot directly read the cookie.

```text
JavaScript
     |
     X
     |
refreshToken
```

But the browser can send the cookie to the appropriate backend.

```text
Browser
   |
   | Cookie: refreshToken=abc123
   v
Backend
```

### Secure

The cookie should only be sent over HTTPS.

### SameSite

Controls when cookies are sent in cross-site requests and is an important part of CSRF protection.

> The exact `SameSite` configuration depends on your frontend/backend architecture.

---

# 12. How the Client Knows the Token Expired

There are two common approaches.

## Approach 1 — Backend Returns `401`

The client sends:

```http
GET /api/orders
Authorization: Bearer <expired-token>
```

Backend checks the token and discovers:

```text
Token expired
```

Backend responds:

```http
401 Unauthorized
```

The client then calls:

```http
POST /api/auth/refresh
```

This is a common and straightforward approach.

---

## Approach 2 — Client Reads `exp`

A JWT contains:

```json
{
  "sub": "123",
  "exp": 1750000900
}
```

The frontend can decode the JWT and determine approximately when it expires.

It can then refresh before expiration.

However:

> **Decoding is not validation.**

The frontend should never decide that a JWT is trustworthy simply because it can decode it.

The backend must validate:

- Signature
- Expiration
- Issuer, if applicable
- Audience, if applicable
- Other relevant claims

---

# 13. Refresh Token API

Yes, applications commonly have a separate endpoint for refreshing tokens.

For example:

```http
POST /api/auth/refresh
```

If the refresh token is stored in an HttpOnly cookie, the frontend does not need to manually read it.

The browser automatically sends the cookie when the request meets the cookie's rules.

Conceptually:

```http
POST /api/auth/refresh
Cookie: refreshToken=abc123...
```

The backend:

```text
Receive refresh request
        |
        v
Extract refresh token
        |
        v
Validate refresh token
        |
        v
Check expiration/revocation
        |
        v
Find associated user/session
        |
        v
Generate new access token
        |
        v
Return access token
```

Example response:

```json
{
  "accessToken": "new-access-token",
  "expiresIn": 900
}
```

---

# 14. Axios Interceptor

In React applications, token refresh is commonly handled through an Axios interceptor.

The basic idea:

```text
API Request
    |
    v
Backend
    |
    v
401 Unauthorized
    |
    v
Axios Interceptor
    |
    v
POST /auth/refresh
    |
    v
New Access Token
    |
    v
Retry Original Request
```

Simplified example:

```javascript
try {
    return await api.request(config);
} catch (error) {

    if (error.response?.status === 401) {

        const response = await api.post("/auth/refresh");

        setAccessToken(response.data.accessToken);

        return api.request(config);
    }

    throw error;
}
```

A production implementation needs additional protection against:

- Infinite refresh loops
- Refreshing the refresh request itself
- Multiple simultaneous refresh requests
- Refresh-token failure
- Race conditions between requests

---

# 15. What Happens When Refresh Token Expires?

Suppose:

```text
Access Token  → expired
Refresh Token → still valid
```

The client can refresh:

```text
Expired Access Token
        |
        v
/auth/refresh
        |
        v
New Access Token
```

But eventually:

```text
Access Token  → expired
Refresh Token → expired/revoked
```

Then:

```text
/auth/refresh
        |
        v
401 Unauthorized
        |
        v
Clear authentication state
        |
        v
User must log in again
```

Complete lifecycle:

```text
Login
  |
  v
Access Token + Refresh Token
  |
  v
Access Token expires
  |
  v
Refresh Token
  |
  v
New Access Token
  |
  v
Continue using application
  |
  v
Refresh Token expires/revoked
  |
  v
Login again
```

---

# 16. JWT vs Session Authentication

## Session Authentication

Traditional session-based authentication works like:

```text
Login
  |
  v
Server creates session
  |
  v
Session stored on server
  |
  v
Client receives session ID
  |
  v
Client sends session ID
  |
  v
Server looks up session
```

The server maintains authentication state.

Example:

```text
Client
  |
  | Session ID
  v
Backend
  |
  | Lookup
  v
Session Store
```

---

## JWT Authentication

With JWT:

```text
Login
  |
  v
Backend generates JWT
  |
  v
Client receives JWT
  |
  v
Client sends JWT
  |
  v
Backend verifies JWT
```

The backend can verify the token without necessarily looking up a server-side session for every request.

This can be useful for:

- REST APIs
- Distributed systems
- Microservices
- Stateless services

However:

> JWT does **not automatically mean stateless, scalable, or secure**.

---

# 17. JWT vs Access Token

These terms should not be confused.

```text
JWT
 |
 └── Token format


Access Token
 |
 └── Token purpose


Refresh Token
 |
 └── Token purpose


ID Token
 |
 └── Token purpose
```

For example:

```text
Access Token
     |
     v
    JWT
```

But:

```text
Refresh Token
     |
     v
Opaque random token
```

is also perfectly valid.

---

# 18. JWT vs ID Token

When working with **OAuth 2.0 / OpenID Connect**, you will encounter ID Tokens.

### Access Token

Answers:

> "What APIs can this client access?"

### ID Token

Answers:

> "Who authenticated?"

An ID token contains identity information intended for the client/application.

Do not automatically treat an ID token as an API access token.

---

# 19. Authentication vs Authorization

These concepts are different.

## Authentication

Authentication answers:

> **Who are you?**

Example:

```text
JWT
 |
 v
User ID = 123
```

---

## Authorization

Authorization answers:

> **What are you allowed to do?**

Example:

```text
User ID = 123
Role = ADMIN
```

The backend might allow:

```text
GET /users       → Allowed
DELETE /users    → Allowed
```

A regular user might have:

```text
User ID = 456
Role = USER
```

and:

```text
GET /users       → Allowed
DELETE /users    → Forbidden
```

JWT claims can contain information such as roles:

```json
{
  "sub": "123",
  "role": "ADMIN"
}
```

But:

> The backend must enforce authorization. Never trust authorization information simply because the client sends it.

---

# 20. HTTP Status Codes

Authentication-related APIs commonly use:

| Status | Meaning |
|---|---|
| `200` | Request successful |
| `201` | Resource created |
| `400` | Bad request |
| `401` | Authentication required/failed |
| `403` | Authenticated but not authorized |
| `404` | Resource not found |

The most important distinction:

```text
401 → "Who are you?"
403 → "I know who you are, but you're not allowed."
```

For example:

```text
Expired JWT
    |
    v
401 Unauthorized
```

While:

```text
Valid JWT
    |
    v
User authenticated
    |
    v
Insufficient permissions
    |
    v
403 Forbidden
```

---

# 21. JWT Security Essentials

## 21.1 Always Use HTTPS

Never send authentication credentials over plain HTTP in production.

Use:

```text
HTTPS
```

instead.

---

## 21.2 Keep Access Tokens Short-Lived

For example:

```text
15–30 minutes
```

The correct lifetime depends on your security requirements.

Short-lived access tokens reduce the useful lifetime of a stolen access token.

---

## 21.3 Protect Refresh Tokens

Refresh tokens are powerful because they can be used to obtain new access tokens.

For browser applications, a common approach is:

```text
HttpOnly
Secure
SameSite
Cookie
```

---

## 21.4 Never Put Secrets in JWT Payload

Bad:

```json
{
  "username": "sabeer",
  "password": "123456"
}
```

Good:

```json
{
  "sub": "123",
  "role": "USER",
  "exp": 1750000900
}
```

---

## 21.5 Validate JWT on the Backend

Never trust a token simply because the client sends it.

The backend should validate the token.

At minimum, typically verify:

```text
Signature
Expiration
Issuer (when applicable)
Audience (when applicable)
```

---

## 21.6 Protect Signing Keys

If using HMAC:

```text
Secret Key
```

must remain private.

If using asymmetric cryptography such as RSA:

```text
Private Key → Sign
Public Key  → Verify
```

The private key must remain secret.

---

## 21.7 Consider Refresh Token Rotation

Instead of using the same refresh token indefinitely:

```text
Old Refresh Token
        |
        v
New Refresh Token
```

The old token can be invalidated.

This can reduce the impact of refresh-token theft.

---

## 21.8 Implement Revocation

JWT access tokens are often difficult to revoke individually once issued.

Refresh tokens can commonly be tracked and revoked server-side.

Example database record:

```text
RefreshToken
-------------
id
user_id
token_hash
expires_at
revoked
created_at
```

---

# 22. Common JWT Mistakes

## ❌ Putting Passwords in JWT

JWT payloads are readable.

Never store passwords inside them.

---

## ❌ Making Access Tokens Valid for Months

If the token is stolen:

```text
Attacker
   |
   v
Stolen Access Token
   |
   v
Potential API access
   |
   v
For a very long time
```

Use an appropriate short lifetime instead.

---

## ❌ Storing Refresh Tokens Carelessly

Refresh tokens are long-lived credentials.

Treat them as highly sensitive.

---

## ❌ Trusting the Client's Role

Never trust:

```json
{
  "role": "ADMIN"
}
```

simply because the client sent it.

The backend must verify the authentication and authorization information.

---

## ❌ Confusing `401` and `403`

```text
401 → Authentication problem
403 → Authorization problem
```

---

## ❌ Creating an Infinite Refresh Loop

Bad:

```text
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
401
 ↓
Refresh
 ↓
...
```

Your client must detect refresh failure and stop retrying.

---

## ❌ Refreshing Multiple Times Concurrently

Suppose five API requests fail simultaneously:

```text
Request 1 → 401
Request 2 → 401
Request 3 → 401
Request 4 → 401
Request 5 → 401
```

A poorly designed client might send:

```text
5 refresh requests
```

A better implementation coordinates the refresh process so that one refresh request can serve multiple waiting requests.

---

# 23. Spring Security Mental Model

For a Spring Boot backend, JWT authentication commonly follows this flow:

```text
HTTP Request
     |
     v
Spring Security Filter Chain
     |
     v
JWT Authentication Filter
     |
     ├── Extract Bearer Token
     |
     ├── Validate JWT
     |
     ├── Extract Claims
     |
     └── Create Authentication
     |
     v
SecurityContext
     |
     v
Authorization
     |
     ├── @PreAuthorize
     ├── hasRole()
     └── hasAuthority()
     |
     v
Controller
```

The important Spring Security concepts to understand are:

```text
SecurityFilterChain
        |
        v
Filter
        |
        v
JWT Validation
        |
        v
Authentication
        |
        v
SecurityContext
        |
        v
Authorization
        |
        v
Controller
```

### Typical Request

```http
GET /api/users/me
Authorization: Bearer eyJhbGciOiJIUzI1Ni...
```

Spring Security:

```text
Extract token
      |
      v
Validate token
      |
      v
Extract user information
      |
      v
Create Authentication object
      |
      v
Store in SecurityContext
      |
      v
Check authorization
      |
      v
Controller
```

---

# 24. Practical Authentication API

A typical backend authentication API might contain:

```text
POST /api/auth/register
POST /api/auth/login
POST /api/auth/refresh
POST /api/auth/logout
```

Protected APIs might look like:

```text
GET    /api/users/me
GET    /api/orders
POST   /api/orders
DELETE /api/orders/{id}
```

---

## Login

Request:

```http
POST /api/auth/login
Content-Type: application/json
```

```json
{
  "email": "user@example.com",
  "password": "password"
}
```

Response:

```json
{
  "accessToken": "...",
  "expiresIn": 900
}
```

The refresh token may be returned as an HttpOnly cookie.

---

## Protected Request

```http
GET /api/orders
Authorization: Bearer <access-token>
```

---

## Refresh

```http
POST /api/auth/refresh
```

Response:

```json
{
  "accessToken": "...",
  "expiresIn": 900
}
```

---

## Logout

```http
POST /api/auth/logout
```

The backend can revoke/delete the associated refresh-token session.

---

# 25. Complete JWT Lifecycle

Here is the complete practical lifecycle to remember:

```text
                         LOGIN
                           |
                           v
                        Backend
                           |
                  Validate Credentials
                           |
                           v
                  Generate Credentials
                           |
              ┌────────────┴────────────┐
              v                         v
        Access Token              Refresh Token
        Short-lived                Long-lived
              |                         |
              v                         v
           Client                  Secure Storage
              |
              v
       API Request
              |
              v
        Backend validates
        Access Token
              |
        ┌─────┴─────┐
        |           |
      Valid       Expired
        |           |
        v           v
   API Response    401
                    |
                    v
            /auth/refresh
                    |
                    v
          Validate Refresh Token
                    |
              ┌─────┴─────┐
              |           |
            Valid       Invalid
              |           |
              v           v
       New Access Token   Login
              |
              v
       Retry API Request
              |
              v
           Success
```

---

# 26. Key Takeaways

If you are building full-stack or backend-heavy applications, remember these concepts.

## JWT

```text
JWT
 ├── Header
 ├── Payload
 └── Signature
```

JWT is a **token format**.

---

## Access Token

```text
Access Token
 ├── Used to access protected APIs
 ├── Usually short-lived
 └── Commonly sent as Bearer token
```

---

## Refresh Token

```text
Refresh Token
 ├── Used to obtain new access token
 ├── Usually long-lived
 ├── Should be protected carefully
 └── Can be revoked
```

---

## Typical Browser Architecture

```text
Access Token
    |
    └── Short-lived
    └── Used for API requests
    └── Often kept in memory


Refresh Token
    |
    └── Long-lived
    └── Used only for refresh
    └── Commonly stored in HttpOnly Secure cookie
```

---

## Complete Lifecycle

```text
LOGIN
  |
  v
Access Token + Refresh Token
  |
  v
Access Token → API Requests
  |
  v
Access Token Expires
  |
  v
401 Unauthorized
  |
  v
POST /auth/refresh
  |
  v
Refresh Token Valid?
  |
  ├── YES → New Access Token → Retry Request
  |
  └── NO  → Login Again
```

---

# Core Mental Model

> **Access Token gets you access. Refresh Token gets you a new access token. JWT is simply one format that can be used for these tokens.**

```text
                 Authentication
                       |
        ┌──────────────┴──────────────┐
        v                             v
   Access Token                 Refresh Token
        |                             |
        v                             v
   Access APIs                Get new Access Token
        |                             |
        v                             v
 Short-lived                  Long-lived + Protected
```

This is the core JWT knowledge you should be able to explain and implement as a backend/full-stack developer.
