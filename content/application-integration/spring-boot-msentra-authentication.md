---
title: "Securing a Spring Boot API with MS Entra ID (Azure AD)"
date: 2026-02-02
draft: false
description: "A practical guide to protecting Java Spring Boot APIs using MS Entra ID OAuth 2.0 authentication with Terraform-managed application registrations"
weight: 1
tags: ["spring-boot", "java", "gradle", "oauth2", "msentra", "azure", "authentication", "terraform", "api-security", "jwt"]
comments: true
---

## Introduction

Modern APIs need robust authentication to protect sensitive resources. MS Entra ID (formerly Azure Active Directory) provides enterprise-grade OAuth 2.0 authentication that integrates seamlessly with Spring Boot applications.

In this article, you'll learn how to:
- Register an application in MS Entra ID using Terraform
- Configure a Spring Boot API to validate OAuth 2.0 access tokens
- Implement API endpoint protection
- Test your secured API with real tokens

This is the first article in a series on Spring Boot API security with MS Entra. Future articles will cover implementing role-based access control (RBAC) using app_roles and fine-grained authorization patterns.

**Complete Code Example:** The full working implementation of this article is available in the [application-integration](https://github.com/giveexamples/application-integration) repository on GitHub.

## Prerequisites

Before starting, ensure you have:
- Java 25 or later
- Gradle 9.3.0 or later
- An Azure subscription with permissions to create app registrations
- Terraform 1.14.4 or later
- Basic understanding of [OAuth 2.0 and OIDC]({{< ref "oauth2-oidc-saml" >}})

## Understanding the OAuth 2.0 Flow

When securing an API with MS Entra ID, you're implementing the **OAuth 2.0 Bearer Token** authentication pattern:

1. A client application obtains an access token from MS Entra ID
2. The client includes this token in the `Authorization` header when calling your API
3. Your Spring Boot API validates the token signature and claims
4. If valid, the request proceeds; otherwise, returns 401 Unauthorized

``` text
┌─────────┐                                   ┌──────────────┐
│ Client  │                                   │  MS Entra ID │
│   App   │                                   │              │
└────┬────┘                                   └──────┬───────┘
     │                                               │
     │ 1. Request access token                       │
     │──────────────────────────────────────────────>│
     │                                               │
     │ 2. Return access token (JWT)                  │
     │<──────────────────────────────────────────────│
     │                                               │
┌────┴────┐                                   ┌─────┴────────┐
│ Client  │                                   │ Spring Boot  │
│   App   │                                   │     API      │
└────┬────┘                                   └──────┬───────┘
     │                                               │
     │ 3. API call with Authorization: Bearer <token>│
     │──────────────────────────────────────────────>│
     │                                               │
     │                                    4. Validate token
     │                                               │
     │ 5. Return protected resource                  │
     │<──────────────────────────────────────────────│
```

## Step 1: Register Application in MS Entra ID with Terraform

First, create an app registration in MS Entra ID. This defines your API and its permissions.

Create a Terraform configuration file:

**`terraform/app-registration.tf`**
```hcl
terraform {
  required_providers {
    azuread = {
      source  = "hashicorp/azuread"
      version = "~> 2.47"
    }
  }
}

provider "azuread" {
  tenant_id = var.tenant_id
}

# Create the API application registration
resource "azuread_application" "api" {
  display_name = "My Protected API"

  # Define the API's identifier URI
  identifier_uris = ["api://my-protected-api"]

  # Define OAuth2 permissions (scopes)
  api {
    oauth2_permission_scope {
      admin_consent_description  = "Allow the application to access the API"
      admin_consent_display_name = "Access API"
      enabled                    = true
      id                         = "00000000-0000-0000-0000-000000000001"
      type                       = "User"
      user_consent_description   = "Allow the application to access the API on your behalf"
      user_consent_display_name  = "Access API"
      value                      = "API.Access"
    }
  }

  # Enable access token issuance
  web {
    implicit_grant {
      access_token_issuance_enabled = true
    }
  }
}

# Create a service principal for the application
resource "azuread_service_principal" "api" {
  client_id = azuread_application.api.client_id
}

# Output important values for Spring Boot configuration
output "client_id" {
  value       = azuread_application.api.client_id
  description = "The application (client) ID"
}

output "tenant_id" {
  value       = var.tenant_id
  description = "The Azure AD tenant ID"
}

output "identifier_uri" {
  value       = "api://my-protected-api"
  description = "The API identifier URI"
}
```

**`terraform/variables.tf`**
```hcl
variable "tenant_id" {
  description = "Azure AD tenant ID"
  type        = string
}
```

**`terraform/terraform.tfvars`**
```hcl
tenant_id = "your-tenant-id-here"
```

Apply the Terraform configuration:
```bash
cd terraform
terraform init
terraform apply
```

Save the output values - you'll need them for Spring Boot configuration.

## Step 2: Create Spring Boot Application

Create a new Spring Boot project with Gradle.

**`build.gradle`**
```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.2'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'

java {
    sourceCompatibility = '17'
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot starters
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'

    // For testing
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
    testImplementation 'org.springframework.security:spring-security-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

## Step 3: Configure OAuth 2.0 Resource Server

Configure Spring Boot to validate JWT tokens from MS Entra ID.

**`src/main/resources/application.yml`**
```yaml
spring:
  application:
    name: my-protected-api

  security:
    oauth2:
      resourceserver:
        jwt:
          # MS Entra ID issues JWTs - configure the issuer URI
          issuer-uri: https://login.microsoftonline.com/${AZURE_TENANT_ID}/v2.0
          # Optionally specify the JWK Set URI directly
          jwk-set-uri: https://login.microsoftonline.com/${AZURE_TENANT_ID}/discovery/v2.0/keys

          # Configure audience validation
          audiences:
            - api://my-protected-api

server:
  port: 8080

# Logging for debugging (optional)
logging:
  level:
    org.springframework.security: DEBUG
```

**Environment Variables:**
Set the `AZURE_TENANT_ID` environment variable with your tenant ID from Terraform output.

## Step 4: Configure Security

Create a security configuration class to protect your endpoints.

**`src/main/java/com/example/api/config/SecurityConfig.java`**
```java
package com.example.api.config;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.oauth2.core.DelegatingOAuth2TokenValidator;
import org.springframework.security.oauth2.core.OAuth2TokenValidator;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.security.oauth2.jwt.JwtDecoder;
import org.springframework.security.oauth2.jwt.JwtDecoders;
import org.springframework.security.oauth2.jwt.JwtValidators;
import org.springframework.security.oauth2.jwt.NimbusJwtDecoder;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Value("${spring.security.oauth2.resourceserver.jwt.issuer-uri}")
    private String issuerUri;

    @Value("${spring.security.oauth2.resourceserver.jwt.audiences}")
    private String audience;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                // Allow health check endpoints without authentication
                .requestMatchers("/actuator/health").permitAll()

                // Require authentication for all other endpoints
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt.decoder(jwtDecoder()))
            );

        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        NimbusJwtDecoder jwtDecoder = JwtDecoders.fromIssuerLocation(issuerUri);

        // Add audience validation
        OAuth2TokenValidator<Jwt> audienceValidator = new AudienceValidator(audience);
        OAuth2TokenValidator<Jwt> withIssuer = JwtValidators.createDefaultWithIssuer(issuerUri);
        OAuth2TokenValidator<Jwt> withAudience = new DelegatingOAuth2TokenValidator<>(withIssuer, audienceValidator);

        jwtDecoder.setJwtValidator(withAudience);

        return jwtDecoder;
    }
}
```

**Custom Audience Validator:**

**`src/main/java/com/example/api/config/AudienceValidator.java`**
```java
package com.example.api.config;

import org.springframework.security.oauth2.core.OAuth2Error;
import org.springframework.security.oauth2.core.OAuth2TokenValidator;
import org.springframework.security.oauth2.core.OAuth2TokenValidatorResult;
import org.springframework.security.oauth2.jwt.Jwt;

public class AudienceValidator implements OAuth2TokenValidator<Jwt> {

    private final String audience;

    public AudienceValidator(String audience) {
        this.audience = audience;
    }

    @Override
    public OAuth2TokenValidatorResult validate(Jwt jwt) {
        if (jwt.getAudience().contains(audience)) {
            return OAuth2TokenValidatorResult.success();
        }

        OAuth2Error error = new OAuth2Error(
            "invalid_token",
            "The required audience is missing",
            null
        );
        return OAuth2TokenValidatorResult.failure(error);
    }
}
```

## Step 5: Create Protected API Endpoints

Create REST controllers with protected endpoints.

**`src/main/java/com/example/api/controller/ApiController.java`**
```java
package com.example.api.controller;

import org.springframework.security.core.annotation.AuthenticationPrincipal;
import org.springframework.security.oauth2.jwt.Jwt;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Map;

@RestController
@RequestMapping("/api")
public class ApiController {

    @GetMapping("/public-info")
    public Map<String, String> getPublicInfo() {
        return Map.of(
            "message", "This endpoint is protected by OAuth 2.0",
            "status", "authenticated"
        );
    }

    @GetMapping("/user-info")
    public Map<String, Object> getUserInfo(@AuthenticationPrincipal Jwt jwt) {
        return Map.of(
            "subject", jwt.getSubject(),
            "claims", jwt.getClaims(),
            "scopes", jwt.getClaimAsStringList("scp")
        );
    }

    @GetMapping("/secure-data")
    public Map<String, Object> getSecureData(@AuthenticationPrincipal Jwt jwt) {
        // Extract user information from JWT
        String userId = jwt.getSubject();
        String userName = jwt.getClaimAsString("name");
        String email = jwt.getClaimAsString("preferred_username");

        return Map.of(
            "userId", userId,
            "userName", userName != null ? userName : "Unknown",
            "email", email != null ? email : "Unknown",
            "data", "This is protected data for authenticated user"
        );
    }
}
```

## Step 6: Testing Your Protected API

### 6.1 Obtain an Access Token

To test your API, you need a valid access token. You can use the Azure CLI:

```bash
# Login to Azure
az login

# Get an access token for your API
az account get-access-token \
  --resource api://my-protected-api \
  --query accessToken \
  --output tsv
```

Alternatively, use the Microsoft Authentication Library (MSAL) or Postman with OAuth 2.0 configuration.

### 6.2 Test Without Token (Should Fail)

```bash
curl -X GET http://localhost:8080/api/public-info
```

**Expected Response:** `401 Unauthorized`

### 6.3 Test With Token (Should Succeed)

```bash
TOKEN="<your-access-token>"

curl -X GET http://localhost:8080/api/public-info \
  -H "Authorization: Bearer ${TOKEN}"
```

**Expected Response:**
```json
{
  "message": "This endpoint is protected by OAuth 2.0",
  "status": "authenticated"
}
```

### 6.4 Test User Info Endpoint

```bash
curl -X GET http://localhost:8080/api/user-info \
  -H "Authorization: Bearer ${TOKEN}"
```

**Expected Response:**
```json
{
  "subject": "00000000-0000-0000-0000-000000000000",
  "claims": {
    "aud": "api://my-protected-api",
    "iss": "https://login.microsoftonline.com/{tenant-id}/v2.0",
    "exp": 1706889600,
    "nbf": 1706886000,
    "sub": "...",
    "name": "John Doe",
    "preferred_username": "john.doe@example.com"
  },
  "scopes": ["API.Access"]
}
```

## Common Issues and Troubleshooting

### Issue 1: "Invalid Issuer" Error

**Cause:** The issuer in your JWT doesn't match the configured issuer URI.

**Solution:** Ensure your `issuer-uri` includes the correct tenant ID and version (`/v2.0`):
```yaml
issuer-uri: https://login.microsoftonline.com/${AZURE_TENANT_ID}/v2.0
```

### Issue 2: "Invalid Audience" Error

**Cause:** The `aud` claim in the JWT doesn't match your configured audience.

**Solution:**
1. Verify the `identifier_uris` in Terraform matches your `application.yml`
2. Ensure you're requesting a token for the correct resource: `--resource api://my-protected-api`

### Issue 3: "Invalid Signature" Error

**Cause:** Spring Boot can't verify the JWT signature using MS Entra's public keys.

**Solution:**
1. Ensure your application can reach `https://login.microsoftonline.com/{tenant-id}/discovery/v2.0/keys`
2. Check for network/proxy issues
3. Verify the tenant ID is correct

### Issue 4: Token Expires Immediately

**Cause:** System clock skew or expired token.

**Solution:**
1. Synchronize your system clock
2. Request a fresh token
3. Check the `exp` claim in your JWT (use [jwt.io](https://jwt.io) to decode)

## Security Best Practices

### 1. Always Validate Audience
Never skip audience validation. It prevents tokens issued for other APIs from being accepted.

### 2. Use HTTPS in Production
Always use HTTPS to prevent token interception. Configure your server with TLS certificates.

### 3. Implement Proper Logging
Log authentication failures for security monitoring:

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests(authorize -> authorize
            .anyRequest().authenticated()
        )
        .oauth2ResourceServer(oauth2 -> oauth2
            .jwt(Customizer.withDefaults())
            .authenticationEntryPoint((request, response, authException) -> {
                log.warn("Authentication failed: {}", authException.getMessage());
                response.sendError(HttpServletResponse.SC_UNAUTHORIZED, "Unauthorized");
            })
        );

    return http.build();
}
```

### 4. Set Token Expiration Appropriately
Configure token lifetimes in MS Entra ID based on your security requirements. Shorter is generally better (1 hour is typical).

### 5. Use Environment Variables for Secrets
Never hardcode tenant IDs, client IDs, or other sensitive information. Use environment variables or Azure Key Vault.

### 6. Enable Actuator Health Checks Carefully
Only expose necessary actuator endpoints and consider securing them:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info
  endpoint:
    health:
      show-details: when-authorized
```

## Real-World Example: Multi-Tenant API

In production, you might need to support multiple tenants. Here's how to configure it:

**`application.yml`**
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          # Use a custom validator for multi-tenant scenarios
          issuer-uri: https://login.microsoftonline.com/common/v2.0
```

**`SecurityConfig.java`**
```java
@Bean
public JwtDecoder jwtDecoder() {
    // For multi-tenant, validate against allowed tenant list
    NimbusJwtDecoder jwtDecoder = JwtDecoders.fromIssuerLocation(
        "https://login.microsoftonline.com/common/v2.0"
    );

    OAuth2TokenValidator<Jwt> tenantValidator = new MultiTenantValidator(allowedTenants);
    OAuth2TokenValidator<Jwt> audienceValidator = new AudienceValidator(audience);

    OAuth2TokenValidator<Jwt> validators = new DelegatingOAuth2TokenValidator<>(
        tenantValidator,
        audienceValidator,
        JwtValidators.createDefault()
    );

    jwtDecoder.setJwtValidator(validators);
    return jwtDecoder;
}
```

## Next Steps

You now have a Spring Boot API protected by MS Entra ID OAuth 2.0 authentication. All endpoints require a valid JWT access token, which Spring Security validates automatically.

In the next article, we'll cover:
- **Implementing Role-Based Access Control (RBAC)** using MS Entra App Roles
- Defining custom roles in your app registration
- Enforcing role-based authorization at the endpoint level
- Testing with different user roles

## Further Reading

- [Spring Security OAuth 2.0 Resource Server Documentation](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [Microsoft Identity Platform Documentation](https://learn.microsoft.com/en-us/azure/active-directory/develop/)
- [OAuth 2.0 and OIDC Fundamentals]({{< ref "oauth2-oidc-saml" >}})

---

**Questions or improvements?** Leave a comment below or open an issue on the blog repository.
