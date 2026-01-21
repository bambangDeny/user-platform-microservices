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

## Cross-Service Communication

The platform uses a hybrid communication approach to balance performance
and reliability:

### Synchronous Communication
- **REST API**: Services expose HTTP/REST endpoints for direct
  service-to-service calls when immediate responses are required
- **Use Cases**: Auth validation, user data retrieval, OTP verification

### Asynchronous Communication
- **Message Queue**: Event-driven communication using message brokers
  (e.g., RabbitMQ, Kafka) for decoupled operations
- **Use Cases**: Notification triggers, audit logging, data synchronization

### Service Discovery
- Services register and discover each other through a service registry
  or DNS-based discovery mechanism
- Health checks ensure only healthy service instances receive traffic

### API Contracts
- Well-defined API contracts between services using OpenAPI/Swagger
- Versioned APIs to support backward compatibility during updates

