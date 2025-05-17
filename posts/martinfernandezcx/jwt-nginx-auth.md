---
title: 'Securing Microservices with JWT Validation at the Nginx Proxy Layer'
published: false
description: 'Learn how to implement JWT validation at the Nginx proxy layer to secure your microservices architecture, with detailed implementation steps and security considerations.'
tags: 'authentication, nginx, jwt, microservices, security'
cover_image: ./assets/jwt-nginx-auth/cover.png
---

# Securing Microservices with JWT Validation at the Nginx Proxy Layer

In a microservices architecture, separating concerns is critical for maintainability, scalability, and security. One key decision when building APIs is how and where to handle authentication. A common pattern is to delegate authentication to a dedicated **authentication microservice**, which issues tokens (e.g., JWTs), and use those tokens to access protected resources on **independent backend APIs**. When working in an infrastructure change we faced the callenge of iether integrating the authentication on the node backend (withouth the proper libraries) or keep a single backend just for authorization.

The optios we had then were:
*having the go backend validate the token and proxy to the node backend over authenticated routes.(we tried implementing this, but to go proxy ended up being messi and hard to maintain).
*doing the authentication in node (infrastructure restrictions made us walk away from this)
*Implement another way of authenticating using the existing infrastructure

And this third one is what we came up with after investigating.

This post demonstrates how to validate JWT tokens directly in **Nginx** before routing requests to your protected **Node.js API**, centralizing authorization enforcement at the gateway layer.
This keeps the authentication within the infrasctrucuter boundaries and allows us to simplify both the GO backend and the Node bakend by relying on the NGINX layer.

## Why JWT at the Proxy?

- **Decouples concerns**: Authentication logic doesn't pollute your API code.
- **Consistent enforcement**: All routes must pass the same token checks before hitting backend services.
- **Performance**: Nginx (especially via OpenResty) is efficient and fast at handling token validation.

## Options for JWT Validation

1. **Validate JWT in each backend service**
   - Pros: Full control per service.
   - Cons: Repeated logic, potential for inconsistency.
2. **Use Nginx with a third-party JWT module**
   - Commercial option with NGINX Plus.
3. **Use OpenResty (Nginx + Lua) with `lua-resty-jwt`**
   - Open-source, flexible, and efficient.

## OpenResty + Lua

We use OpenResty and the `lua-resty-jwt` library to inspect JWTs in the Nginx layer. If valid, we forward requests to the backend. Otherwise, Nginx returns a 401 response.

## Architecture

- `auth-api`: issues JWTs via login endpoint.
- `node-api`: protected and public routes.
- `nginx`: gateway with Lua-based JWT validation.

## Security Considerations

Some of these concerns  were left out of this POC but we would like to mention for a proper production implementation. Please read through and evaluate wether it fits to your scenario or not.

### Protection Against Common Attacks

1. **Replay Attacks**
   - Implement token expiration (exp claim)
   - Use short-lived tokens (15-60 minutes)
   - Consider implementing a token blacklist for revoked tokens
   - Use nonce values in token claims
   - Implement request timestamp validation

2. **Token Theft Prevention**
   - Always use HTTPS for token transmission
   - Implement secure cookie attributes (HttpOnly, Secure, SameSite)
   - Use token binding to prevent token reuse
   - Implement rate limiting on authentication endpoints
   - Monitor for suspicious patterns (multiple failed validations)

### Token Expiration Best Practices

1. **Short-lived Access Tokens**
   - Set expiration time between 15-60 minutes
   - Use refresh tokens for longer sessions
   - Implement sliding expiration for active users

2. **Refresh Token Strategy**
   - Longer expiration (days/weeks)
   - Store refresh tokens securely
   - Implement refresh token rotation
   - Maintain a refresh token family tree

3. **Expiration Implementation**
   ```lua
   -- Example of expiration check in Lua
   local jwt = require "resty.jwt"
   local validators = require "resty.jwt-validators"
   
   validators.set_system_leeway(0) -- Strict time validation
   validators.register_validator("exp", validators.opt_is_not_expired())
   ```

4. **Grace Period Considerations**
   - Implement a small grace period (30 seconds) for clock skew
   - Handle token expiration gracefully
   - Provide clear error messages for expired tokens
   - Implement automatic token refresh when possible

## Project Layout

You can find the full source here:  
**GitHub Repo: [martinfernandezcx/NGINXAUTH](https://github.com/martinfernandezcx/NGINXAUTH)**

## How It Works

1. Client logs in via `/api/auth/login`, receives JWT.
2. Client sends `Authorization: Bearer <token>` on protected requests.
3. Nginx runs a Lua script to:
   - Check token structure.
   - Validate signature and expiration.
   - Inject user ID into a request header.
4. Validated requests reach the Node.js service with identity attached.

## Testing with Postman

The project includes a comprehensive Postman test suite to verify the JWT authentication flow and API endpoints. The test suite covers authentication, public routes, and protected routes with various scenarios.

### Test Suite Structure

The Postman collection (`postman/jwt-nginx-auth-tests.json`) includes:

1. **Authentication Tests**
   - Login endpoint validation
   - Token format verification
   - Automatic token storage for subsequent requests

2. **Public Endpoint Tests**
   - Access to public routes
   - Response format validation

3. **Protected Endpoint Tests**
   - Access without token (401)
   - Access with invalid token (401)
   - Access with valid token (200)
   - Response payload validation

### Running the Tests

1. **Prerequisites**
   - Install [Postman](https://www.postman.com/downloads/)
   - Start the application:
     ```bash
     docker-compose up --build
     ```

2. **Import the Collection**
   - Open Postman
   - Click "Import" button
   - Select the `postman/jwt-nginx-auth-tests.json` file
   - select the `postman\environment.json` file
   - The collection will be imported with all test cases

3. **Run the Tests**
   - Select the "JWT Nginx Auth Tests" collection
   - Click the "Run" button
   - Postman will execute all tests in sequence
   - View test results in the Postman console

4. **Test Flow**
   - Tests run in a specific order to ensure proper token handling
   - Login test stores the token for subsequent requests
   - Protected route tests verify token validation
   - Each test includes assertions for status codes and response formats

### Test Cases break down

1. **Login Test**
   ```javascript
   pm.test("Status code is 200", function () {
       pm.response.to.have.status(200);
   });
   pm.test("Response has token", function () {
       var jsonData = pm.response.json();
       pm.expect(jsonData).to.have.property('token');
   });
   ```

2. **Protected Route Test**
   ```javascript
   pm.test("Status code is 200", function () {
       pm.response.to.have.status(200);
   });
   pm.test("Response contains protected data", function () {
       var jsonData = pm.response.json();
       pm.expect(jsonData).to.have.property('message');
   });
   ```

### Environment Variables

The test suite uses Postman environment variables:
- `auth_token`: Automatically set after successful login
- Used in subsequent requests to protected routes

### Continuous Integration

The Postman collection can be integrated into CI/CD pipelines using:
- [Newman](https://github.com/postmanlabs/newman) CLI tool
- Postman's CI/CD integrations
- Custom test runners

Example Newman command:
```bash
newman run postman/jwt-nginx-auth-tests.json -e postman/environment.json
```

## Conclusion

Centralizing JWT validation in the proxy simplifies backend services, enforces uniform security, and keeps authentication logic out of each microservice. This pattern is ideal for architectures using distinct auth and business logic APIs.

In contrast, validating tokens in the Node.js API itself might allow greater control over roles or context-based access logic but at the cost of duplication and potential inconsistency.

OpenResty strikes a solid balance between **performance**, **flexibility**, and **maintainability** in JWT-based authentication.


## Apendix-A: Problems Found and Solutions

During the implementation of this JWT authentication system, we encountered several issues that required specific solutions:

1. **OpenResty Dependencies**
   - Problem: Missing Perl and curl in the OpenResty Alpine image
   - Solution: Added required packages in Dockerfile:
     ```dockerfile
     RUN apk add --no-cache perl curl
     ```

2. **Nginx User Configuration**
   - Problem: Missing nginx user in the container
   - Solution: Created nginx user and group:
     ```dockerfile
     RUN addgroup -S nginx && adduser -S -G nginx nginx
     ```

3. **MIME Types Configuration**
   - Problem: Missing mime.types file in OpenResty Alpine image
   - Solution: Created custom mime.types file and copied it to the correct location:
     ```dockerfile
     COPY mime.types /etc/nginx/mime.types
     ```

4. **Lua Package Path**
   - Problem: Lua package path directive in wrong context
   - Solution: Moved lua_package_path to http context in nginx.conf:
     ```nginx
     http {
         lua_package_path "/usr/local/openresty/lualib/?.lua;;";
         lua_package_cpath "/usr/local/openresty/lualib/?.so;;";
     }
     ```

5. **Log Directory Permissions**
   - Problem: Nginx couldn't write to log directory
   - Solution: Created log directory and set proper permissions:
     ```dockerfile
     RUN mkdir -p /var/log/nginx && \
         chown -R nginx:nginx /var/log/nginx
     ```

These solutions ensure proper functionality of the JWT authentication system while maintaining security and following best practices for containerized applications.
