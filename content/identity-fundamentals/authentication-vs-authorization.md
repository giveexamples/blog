---
title: "Authentication vs Authorization"
date: 2026-01-26
draft: false
description: "Understanding the fundamental difference between authentication and authorization"
weight: 1
tags: ["authentication", "authorization", "auth", "login", "security", "access-control", "IAM"]
comments: true
---

# Authentication vs Authorization

## Introduction

One of the most common sources of confusion in identity and access management is the difference between authentication and authorization. While these terms are often used interchangeably, they represent distinct concepts that serve different purposes in security systems.

## Authentication: Who Are You?

**Authentication** is the process of verifying someone's identity. It answers the question: "Who are you?"

### Common Authentication Methods

- **Something you know**: Passwords, PINs, security questions
- **Something you have**: Hardware tokens, mobile devices, smart cards
- **Something you are**: Biometrics (fingerprints, face recognition, iris scans)
- **Multi-factor authentication (MFA)**: Combining multiple methods for stronger security

### Authentication Flow Example

```
User → "I'm Alice" → System checks credentials → Verified ✓
```

## Authorization: What Can You Do?

**Authorization** is the process of determining what an authenticated user is allowed to do. It answers the question: "What are you allowed to access?"

### Authorization Models

- **Role-Based Access Control (RBAC)**: Permissions based on roles
- **Attribute-Based Access Control (ABAC)**: Permissions based on attributes
- **Policy-Based Access Control (PBAC)**: Permissions based on policies

### Authorization Flow Example

```
Authenticated User (Alice) → Request: "Read file X" → Check permissions → Allowed/Denied
```

## The Key Difference

- **Authentication** happens first and only once per session
- **Authorization** happens continuously for every action
- You can be authenticated but not authorized for a specific action

## Real-World Analogy

Think of an airport:
- **Authentication**: Showing your passport at security (proving who you are)
- **Authorization**: Your boarding pass determines which plane you can board (what you can access)

## Common Mistakes

1. **Assuming authentication implies authorization**: Just because someone is logged in doesn't mean they should access everything
2. **Implementing them together**: Separating these concerns makes systems more maintainable
3. **Weak authorization after strong authentication**: Strong passwords don't help if anyone can access any resource

## Best Practices

- Always authenticate users before authorizing them
- Implement the principle of least privilege
- Separate authentication and authorization logic
- Log both authentication and authorization events
- Regularly review and audit permissions

## Next Steps

Now that you understand the difference between authentication and authorization, let's explore the protocols that implement these concepts: [OAuth 2.0, OIDC, and SAML]({{< ref "oauth2-oidc-saml" >}}).
