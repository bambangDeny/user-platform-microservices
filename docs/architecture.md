# System Architecture

This system is designed as a microservices-based backend platform
built with Go, focusing on scalability, clear service boundaries,
and maintainability.

## Services
- API Gateway: single entry point for all client requests, handling
  authentication validation and request routing
- Auth Service: responsible for login, authentication, and JWT
  (RSA) token management
- User Service: manages user profile, user data, and user roles
- OTP Service: handles OTP generation, verification, and rate limiting
- Notification Service: processes asynchronous notification events

## Service Communication

### Registration Flow
- Client sends registration request to API Gateway
- API Gateway forwards request to User Service
- User Service creates user account in database
- User Service publishes `user.created` event to Kafka
- OTP Service consumes event and generates verification OTP
- OTP Service publishes `otp.verification_required` event to Kafka
- Notification Service consumes event and sends verification notification

### Login Flow
- Client sends login request to API Gateway
- API Gateway forwards request to Auth Service
- Auth Service validates credentials with User Service
- Auth Service generates and returns JWT (RSA signed)
- Auth Service publishes `user.logged_in` event to Kafka
- Notification Service consumes event and sends login notification (optional)

### OTP Request Flow
- Client requests OTP via API Gateway
- API Gateway validates JWT token with Auth Service
- API Gateway forwards request to OTP Service
- OTP Service generates OTP and stores in Redis with TTL
- OTP Service publishes `otp.sent` event to Kafka
- Notification Service consumes event and sends OTP via email/SMS

### OTP Verification Flow
- Client submits OTP via API Gateway
- API Gateway forwards to OTP Service
- OTP Service validates OTP against Redis cache
- OTP Service publishes `otp.verified` or `otp.failed` event to Kafka
- User Service consumes `otp.verified` event and updates user status

### Protected Resource Access Flow
- Client sends request with JWT to API Gateway
- API Gateway validates JWT signature using Auth Service public key
- API Gateway forwards request to appropriate service (User/OTP/etc)
- Target service processes request and returns response
- API Gateway returns response to client

### User Profile Update Flow
- Client sends update request to API Gateway
- API Gateway validates JWT and forwards to User Service
- User Service updates user data in database
- User Service publishes `user.updated` event to Kafka
- Audit Service consumes event for logging (future service)

### Password Reset Flow
- Client requests password reset via API Gateway
- API Gateway forwards to Auth Service
- Auth Service verifies user exists via User Service
- Auth Service publishes `password.reset_requested` event to Kafka
- OTP Service consumes event and generates reset OTP
- OTP Service publishes `otp.password_reset` event to Kafka
- Notification Service sends reset OTP to user
- Client submits new password with OTP
- Auth Service validates OTP with OTP Service
- Auth Service updates password and publishes `password.reset_completed` event

## Data Ownership & Service Boundaries

| Service | Data Owned | Storage | Notes |
|---------|------------|---------|-------|
| Auth Service | username, password hash, refresh token | Postgres | JWT signed with RSA |
| User Service | user profile, email, role, status | Postgres | Emits `user.created/updated` |
| OTP Service | OTP code, TTL, OTP status | Redis | Event-driven, TTL for expiration |
| Notification Service | - | - | Consumes events only, no DB |
| Audit Service | Logs of user actions | Postgres | Consume events for audit & analytics |


