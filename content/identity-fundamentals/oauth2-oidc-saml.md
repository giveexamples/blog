---
title: "OAuth 2.0, OIDC, and SAML"
date: 2026-01-26
draft: false
description: "A deep dive into the major identity protocols and when to use each"
weight: 2
tags: ["oauth2", "oidc", "saml", "protocols", "sso", "federation", "authentication"]
comments: true
---

## Introduction

Modern identity systems rely on standardized protocols to handle authentication and authorization. The three most important protocols you'll encounter are OAuth 2.0, OpenID Connect (OIDC), and SAML. Understanding when to use each is crucial for building secure applications.

## OAuth 2.0: Authorization Framework

**OAuth 2.0** is an authorization framework that enables applications to obtain limited access to user accounts without exposing passwords.

### What OAuth 2.0 Is

- An **authorization** protocol (not authentication!)
- Designed for delegated access
- Uses access tokens to grant permissions

### OAuth 2.0 Grant Types

1. **Authorization Code**: Most secure, used by web/mobile apps
2. **Client Credentials**: Machine-to-machine communication
3. **Device Code**: For devices without browsers
4. **Refresh Token**: Obtain new access tokens without re-authentication

### OAuth 2.0 Flow Example

```
User → App → "Login with Google" → Google → User approves →
App receives access token → App accesses Google resources
```

### When to Use OAuth 2.0

- API authorization
- Third-party application access
- Delegated access scenarios
- Mobile and single-page applications

## OpenID Connect (OIDC): Authentication Layer

**OIDC** is an identity layer built on top of OAuth 2.0. It extends OAuth 2.0 to provide authentication.

### What OIDC Adds

- **ID Token**: JWT containing user identity information
- **UserInfo Endpoint**: Standard way to get user profile data
- **Standard claims**: email, name, picture, etc.

### OIDC Tokens

```json
{
  "iss": "https://accounts.example.com",
  "sub": "alice@example.com",
  "aud": "your-client-id",
  "exp": 1735228800,
  "iat": 1735225200,
  "name": "Alice Smith",
  "email": "alice@example.com"
}
```

### When to Use OIDC

- Single Sign-On (SSO) for modern applications
- Mobile and web app authentication
- Microservices identity propagation
- Consumer applications with social login

## SAML: Enterprise Federation

**SAML** (Security Assertion Markup Language) is an XML-based protocol primarily used for enterprise SSO.

### What SAML Provides

- XML-based assertions
- Enterprise SSO
- Strong support in legacy systems
- Rich attribute exchange

### SAML Flow

```
User → Service Provider (SP) → Identity Provider (IdP) →
IdP authenticates → SAML assertion → SP grants access
```

### When to Use SAML

- Enterprise SSO requirements
- Legacy application integration
- Compliance requirements (HIPAA, SOC 2)
- B2B integrations

## Comparison

| Feature | OAuth 2.0 | OIDC | SAML |
|---------|-----------|------|------|
| **Purpose** | Authorization | Authentication | Authentication |
| **Format** | JSON | JSON (JWT) | XML |
| **Mobile Support** | Excellent | Excellent | Limited |
| **Complexity** | Medium | Medium | High |
| **Use Case** | API access | Modern apps | Enterprise SSO |

## Choosing the Right Protocol

### Use OAuth 2.0 when:
- You need API authorization
- Building third-party integrations
- Implementing delegated access

### Use OIDC when:
- You need authentication for modern apps
- Building consumer-facing applications
- Implementing SSO for web/mobile

### Use SAML when:
- Required by enterprise customers
- Integrating with legacy systems
- Meeting specific compliance requirements

## Common Misconceptions

1. **"OAuth 2.0 is for authentication"**: No, it's for authorization. Use OIDC for authentication.
2. **"SAML is outdated"**: It's still widely used in enterprises
3. **"You can't use multiple protocols"**: Many systems support both OIDC and SAML

## Best Practices

- Use OIDC for new applications unless SAML is required
- Always validate tokens properly
- Use proven libraries rather than implementing from scratch
- Keep up with protocol updates and security advisories
- Consider your audience (consumers vs enterprises)

## Next Steps

Understanding protocols is essential, but so is knowing the different types of identities: [User vs Workload Identities]({{< ref "user-vs-workload-identities" >}}).