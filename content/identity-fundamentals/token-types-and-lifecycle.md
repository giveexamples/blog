---
title: "Token Types and Lifecycle"
date: 2026-01-26
draft: false
description: "Understanding access tokens, refresh tokens, ID tokens, and how they work together"
weight: 4
tags: ["tokens", "jwt", "oauth2", "oidc", "security", "authentication"]
comments: true
---

## Introduction

Modern identity systems rely heavily on tokens to represent identity and authorization. Understanding the different types of tokens, their purposes, and lifecycles is essential for building secure applications.

## What is a Token?

A **token** is a piece of data that represents something. In identity systems, tokens typically represent:
- Who you are (identity)
- What you can access (authorization)
- For how long (lifetime)

Tokens are usually implemented as:
- **JWT (JSON Web Token)**: Self-contained, can be validated without server calls
- **Opaque tokens**: Random strings that require server lookup
- **Structured tokens**: Custom formats specific to certain systems

## The Three Main Token Types

### 1. Access Token

**Purpose**: Grants access to protected resources (APIs, services)

#### Characteristics

- Short-lived (minutes to hours)
- Contains authorization information (scopes, permissions)
- Should be treated as credentials (never logged or shared)
- Can be JWT or opaque

#### Example JWT Access Token

```json
{
  "iss": "https://auth.example.com",
  "sub": "alice@example.com",
  "aud": "https://api.example.com",
  "exp": 1735228800,
  "iat": 1735225200,
  "scope": "read:users write:posts",
  "client_id": "my-application"
}
```

#### Usage

```http
GET /api/users HTTP/1.1
Host: api.example.com
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

#### Security Considerations

- Never store in localStorage (XSS vulnerable)
- Use httpOnly cookies or memory storage
- Validate signature and expiration
- Check audience (aud) and scope claims
- Rotate frequently

### 2. Refresh Token

**Purpose**: Obtain new access tokens without re-authentication

#### Characteristics

- Long-lived (days to months)
- Opaque (usually)
- Single-use or rotating
- Should be stored securely
- Can be revoked by authorization server

#### Why Refresh Tokens Exist

Access tokens are short-lived for security. Refresh tokens allow:
- Extending sessions without re-login
- Reducing attack surface (short-lived access tokens)
- Centralized revocation capability

#### Refresh Token Flow

``` text
1. Initial authentication → Access Token (15 min) + Refresh Token (7 days)
2. Access token expires
3. Client sends refresh token → New Access Token (15 min)
4. Repeat until refresh token expires or is revoked
```

#### Security Considerations

- Store in secure, encrypted storage
- Implement refresh token rotation
- Monitor for suspicious refresh patterns
- Revoke on logout or security events
- Bind to client/device when possible

### 3. ID Token (OIDC Only)

**Purpose**: Contains user identity information

#### Characteristics

- Always a JWT
- Signed by identity provider
- Contains user profile claims
- Not used for API authorization
- Validated by client application

#### Example ID Token

```json
{
  "iss": "https://accounts.example.com",
  "sub": "123456789",
  "aud": "my-client-id",
  "exp": 1735228800,
  "iat": 1735225200,
  "email": "alice@example.com",
  "email_verified": true,
  "name": "Alice Smith",
  "picture": "https://example.com/alice.jpg"
}
```

#### Common Claims

- `sub`: Unique user identifier (never changes)
- `email`: User's email address
- `name`: User's full name
- `picture`: Profile picture URL
- Custom claims: role, department, etc.

#### Security Considerations

- Validate signature against public key
- Check issuer (iss) matches expected
- Verify audience (aud) is your client ID
- Confirm expiration (exp) is in future
- Never use for API authorization (use access tokens)

## Token Lifecycle

### Phase 1: Token Issuance

``` json
User Authentication → Authorization Server → Issues Tokens
├── Access Token (short-lived)
├── Refresh Token (long-lived)
└── ID Token (identity information)
```

### Phase 2: Token Usage

``` json
Client → Access Token → Resource Server
├── Validates token signature
├── Checks expiration
├── Verifies audience and scope
└── Grants or denies access
```

### Phase 3: Token Refresh

``` json
Client → Refresh Token → Authorization Server
├── Validates refresh token
├── Checks if revoked
├── Issues new access token
└── (Optional) Issues new refresh token (rotation)
```

### Phase 4: Token Expiration/Revocation

``` json
Token Expires:
└── Client must refresh or re-authenticate

Token Revoked:
├── User logs out
├── Security incident
├── Permission changes
└── Client must re-authenticate
```

## Token Storage Best Practices

### Browser/SPA Applications

| Token Type | Storage | Reasoning |
|------------|---------|-----------|
| Access Token | Memory | Prevents XSS attacks |
| Refresh Token | httpOnly Cookie | Secure, browser-managed |
| ID Token | Memory | Only needed for display |

### Mobile Applications

| Token Type | Storage | Reasoning |
|------------|---------|-----------|
| Access Token | Secure memory | Short-lived, in-memory |
| Refresh Token | Keychain/KeyStore | Platform secure storage |
| ID Token | Can persist | Display purposes |

### Backend Services

| Token Type | Storage | Reasoning |
|------------|---------|-----------|
| Access Token | Memory/Cache | Short-lived |
| Client Credentials | Environment/Secrets Manager | Protected configuration |

## Token Validation

### JWT Validation Checklist

```javascript
function validateJWT(token) {
  // 1. Decode without verification
  const decoded = jwt.decode(token, { complete: true });

  // 2. Check algorithm is expected
  if (decoded.header.alg !== 'RS256') throw new Error('Invalid algorithm');

  // 3. Verify signature
  const publicKey = await getPublicKey(decoded.header.kid);
  jwt.verify(token, publicKey);

  // 4. Check expiration
  if (decoded.payload.exp < Date.now() / 1000) throw new Error('Token expired');

  // 5. Check issuer
  if (decoded.payload.iss !== EXPECTED_ISSUER) throw new Error('Invalid issuer');

  // 6. Check audience
  if (!decoded.payload.aud.includes(EXPECTED_AUDIENCE)) throw new Error('Invalid audience');

  // 7. Check custom claims (scope, permissions)
  if (!decoded.payload.scope.includes('required:scope')) throw new Error('Insufficient scope');

  return decoded.payload;
}
```

## Common Token Patterns

### Backend for Frontend (BFF)

``` json
Browser → Backend (BFF) → Resource API
         ├── Session Cookie
         └── Uses access token server-side
```

Benefits:
- Tokens never exposed to browser
- Backend manages refresh logic
- Better security posture

### Token Exchange

``` json
External Token → Token Exchange Service → Internal Token
└── Allows cross-domain/cross-organization access
```

### Phantom Tokens

``` json
Opaque Token (external) → Gateway → JWT (internal) → Services
└── External clients get opaque, internal services get JWT
```

## Token Rotation Strategies

### Refresh Token Rotation

``` json
1. Client uses refresh token
2. Server issues new access + new refresh token
3. Old refresh token is invalidated
4. Client updates stored refresh token
```

Benefits:
- Limits impact of leaked refresh tokens
- Detects token theft
- Enforces maximum session lifetime

### Access Token Rotation

``` json
1. Access token expires (15 min)
2. Client automatically refreshes
3. New access token issued
4. Client continues with new token
```

## Troubleshooting Common Issues

### "Invalid Token" Errors

- Check token expiration (exp claim)
- Verify signature with correct public key
- Ensure audience (aud) matches your API
- Check issuer (iss) is expected
- Confirm clock skew isn't too large

### Token Size Issues

- JWTs can be large (especially with many claims)
- Consider using opaque tokens for large claim sets
- Use token references for internal services
- Avoid embedding excessive data in tokens

### Performance Concerns

- Cache public keys (with TTL)
- Use opaque tokens if constant DB lookups acceptable
- Consider token introspection caching
- Implement proper token caching strategies

## Security Best Practices

1. **Never log tokens**: Treat as credentials
2. **Short-lived access tokens**: Minutes to hours, not days
3. **Rotate refresh tokens**: Use rotation for better security
4. **Validate everything**: Signature, expiration, audience, issuer
5. **Use HTTPS**: Always encrypt token transmission
6. **Implement revocation**: Have a way to invalidate tokens
7. **Monitor token usage**: Detect anomalies and abuse
8. **Bind tokens when possible**: To device, IP, or client

## Conclusion

Understanding token types and their lifecycles is crucial for:
- Building secure authentication systems
- Implementing proper authorization
- Debugging identity-related issues
- Making informed architectural decisions

Tokens are the foundation of modern identity systems - handle them with care!

---

This concludes our Identity Fundamentals series. You now have a solid foundation in:
- [Authentication vs Authorization]({{< ref "authentication-vs-authorization" >}})
- [OAuth 2.0, OIDC, and SAML]({{< ref "oauth2-oidc-saml" >}})
- [User vs Workload Identities]({{< ref "user-vs-workload-identities" >}})
- Token Types and Lifecycle

Use this knowledge to build secure, scalable identity systems!
