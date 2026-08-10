# User Authentication & Authorization Service

A Spring Boot–based authentication and authorization microservice responsible for **user registration, login, JWT-based authentication, role management, session tracking, token validation, and user retrieval**.

The service is designed as an independent authentication component that can participate in a larger microservice architecture through **Eureka service discovery** and can publish asynchronous events using **Apache Kafka**.

---

## 🚀 Features

* User registration / signup
* User login
* JWT token generation
* JWT signature verification
* JWT expiration validation
* Role-based authorization data embedded in JWT claims
* Persistent user sessions
* Token validation through stored sessions
* User lookup by ID
* User status validation
* Duplicate email detection
* Automatic default role assignment
* Kafka event publishing after successful signup
* MySQL persistence using Spring Data JPA
* Eureka service registration
* DTO-based API responses
* Layered Spring Boot architecture

---

# 🏗️ Architecture

The service follows a layered architecture:

```text
                         ┌──────────────────────┐
                         │       Client         │
                         │ Web / Mobile / API   │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │    REST Controllers  │
                         │                      │
                         │   AuthController     │
                         │   UserController     │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │       Services       │
                         │                      │
                         │     AuthService      │
                         │     UserService      │
                         └───────┬───────┬──────┘
                                 │       │
                 ┌───────────────┘       └─────────────────┐
                 ▼                                         ▼
        ┌──────────────────┐                     ┌─────────────────┐
        │ Spring Data JPA  │                     │     JWT         │
        │    Repositories  │                     │ Generation &    │
        │                  │                     │   Validation    │
        └────────┬─────────┘                     └─────────────────┘
                 │
                 ▼
        ┌──────────────────┐
        │      MySQL       │
        │                  │
        │ Users            │
        │ Roles            │
        │ Sessions         │
        └──────────────────┘

                         ┌──────────────────┐
                         │      Kafka       │
                         │                  │
                         │ Signup Events    │
                         └──────────────────┘

                         ┌──────────────────┐
                         │      Eureka      │
                         │ Service Discovery│
                         └──────────────────┘
```

---

# 🛠️ Tech Stack

| Technology                  | Purpose                         |
| --------------------------- | ------------------------------- |
| Java 17                     | Programming language            |
| Spring Boot                 | Backend framework               |
| Spring Web                  | REST API development            |
| Spring Data JPA             | Persistence layer               |
| Hibernate                   | ORM                             |
| MySQL                       | Relational database             |
| JJWT                        | JWT creation and validation     |
| Apache Kafka                | Asynchronous event publishing   |
| Spring Cloud Netflix Eureka | Service discovery               |
| Lombok                      | Boilerplate reduction           |
| Maven                       | Build and dependency management |
| JUnit / Spring Boot Test    | Testing                         |

The project uses Java 17 and includes Spring Web, Spring Data JPA, MySQL Connector, JJWT, Kafka, Eureka Client, Lombok, and Spring Boot Test dependencies.

---

# 📁 Project Structure

```text
UserAuthService/
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── org/example/userauthservice/
│   │   │       │
│   │   │       ├── config/
│   │   │       │   └── AuthConfig.java
│   │   │       │
│   │   │       ├── constants/
│   │   │       │   └── RoleValues.java
│   │   │       │
│   │   │       ├── controllers/
│   │   │       │   ├── AuthController.java
│   │   │       │   └── UserController.java
│   │   │       │
│   │   │       ├── dtos/
│   │   │       │   ├── LoginRequestDto.java
│   │   │       │   ├── SignupRequestDto.java
│   │   │       │   ├── UserDto.java
│   │   │       │   ├── ValidateTokenRequestDto.java
│   │   │       │   └── EmailDto.java
│   │   │       │
│   │   │       ├── models/
│   │   │       │   ├── BaseModel.java
│   │   │       │   ├── User.java
│   │   │       │   ├── Role.java
│   │   │       │   ├── UserSession.java
│   │   │       │   └── Status.java
│   │   │       │
│   │   │       ├── repos/
│   │   │       │   ├── UserRepo.java
│   │   │       │   ├── RoleRepo.java
│   │   │       │   └── SessionRepo.java
│   │   │       │
│   │   │       ├── services/
│   │   │       │   ├── AuthService.java
│   │   │       │   └── UserService.java
│   │   │       │
│   │   │       └── UserAuthServiceApplication.java
│   │   │
│   │   └── resources/
│   │       └── application.properties
│   │
│   └── test/
│       └── java/
│           └── org/example/userauthservice/
│
├── pom.xml
├── mvnw
├── mvnw.cmd
└── README.md
```

The repository is organized into configuration, constants, controllers, DTOs, models, repositories, and services.

---

# 🔐 Authentication Flow

## Signup Flow

```text
Client
  │
  │ POST /auth/signup
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├── Check existing email
  │
  ├── Create User
  │
  ├── Assign default role
  │
  ├── Persist User
  │
  └── Publish signup event
          │
          ▼
        Kafka
```

During signup, the service:

1. Checks whether the email already exists.
2. Creates a new `User`.
3. Finds or creates the default non-admin role.
4. Associates the role with the user.
5. Saves the user using JPA.
6. Publishes a signup event to Kafka.

The current implementation publishes an email-oriented event to the Kafka topic `signup`.

---

# 🔑 Login Flow

```text
Client
  │
  │ email + password
  ▼
AuthController
  │
  ▼
AuthService
  │
  ├── Find user
  │
  ├── Check account status
  │
  ├── Verify credentials
  │
  ├── Build JWT claims
  │
  ├── Sign JWT
  │
  ├── Persist session
  │
  └── Return user + token
```

The login process validates:

* User existence
* User account status
* Credentials

After successful authentication, the service creates a signed JWT and persists a corresponding `UserSession`.

---

# 🎟️ JWT Authentication

The service uses **JSON Web Tokens (JWT)** for stateless authentication.

The token contains claims including:

```json
{
  "user_id": 123,
  "access": [
    "NON_ADMIN"
  ],
  "iat": 1720000000000,
  "exp": 1720000100000,
  "issued_by": "scaler",
  "type": "auth"
}
```

The actual role values depend on the roles assigned to the user.

The service generates the token using an HMAC SHA-256 signing key:

```text
HS256
```

The JWT implementation uses the JJWT library and a `SecretKey` configured through `AuthConfig`.

---

# ⏱️ Token Expiration

Tokens are issued with:

```text
iat = issued-at timestamp
exp = issued-at timestamp + 100000 ms
```

The current implementation therefore gives a token a lifetime of approximately:

```text
100 seconds
```

The expiration is checked during token validation.

If the token has expired, the associated session is removed from the database.

---

# 🗂️ Session Management

JWTs are not the only authentication state maintained by the application.

After login:

```text
JWT
 │
 ▼
UserSession
 │
 ├── token
 ├── user
 └── status
```

The token is persisted in the `UserSession` entity.

During validation:

```text
Incoming Token
      │
      ▼
Find UserSession
      │
      ├── Not found → false
      │
      ▼
Verify JWT Signature
      │
      ▼
Check Expiration
      │
      ├── Expired → Delete Session → false
      │
      └── Valid → true
```

This provides a database-backed mechanism for determining whether a token is still recognized by the service.

---

# 👥 Role-Based Authorization

Users can have multiple roles.

The relationship is:

```text
User
 │
 │ Many-to-Many
 ▼
Role
```

The `User` entity contains:

```java
@ManyToMany
private List<Role> roles;
```

During signup, a default non-admin role is assigned to the new user.

Role values are also embedded inside the JWT:

```json
{
  "access": [
    "NON_ADMIN"
  ]
}
```

This allows downstream services to determine the user's authorization level from the authentication token.

---

# 📡 API Endpoints

Base URL:

```text
http://localhost:8080
```

---

## 1. Signup

```http
POST /auth/signup
```

### Request

```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123",
  "phoneNumber": "9876543210"
}
```

### Response

```http
201 Created
```

Example response:

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "roles": [
    "NON_ADMIN"
  ]
}
```

The password is not included in the returned `UserDto`.

---

## 2. Login

```http
POST /auth/login
```

### Request

```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

### Response

```http
200 OK
```

The service returns user information and places the generated authentication token in the response's `Set-Cookie` header under `auth_session_id`.

Example:

```text
Set-Cookie: auth_session_id=<JWT>
```

---

## 3. Validate Token

```http
POST /auth/validateToken
```

### Request

```json
{
  "token": "<JWT_TOKEN>"
}
```

### Response

```json
true
```

or:

```json
false
```

The service first checks whether the token exists in a persisted session and then verifies its JWT signature and expiration.

---

## 4. Get User by ID

```http
GET /users/{id}
```

### Example

```bash
curl http://localhost:8080/users/1
```

### Response

```json
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "roles": [
    "NON_ADMIN"
  ]
}
```

The endpoint retrieves the user through `UserService` and maps the entity to `UserDto`.

---

# 🗄️ Data Model

The primary entities are:

```text
                 ┌─────────────┐
                 │    User     │
                 ├─────────────┤
                 │ id          │
                 │ name        │
                 │ email       │
                 │ password    │
                 │ phoneNumber │
                 └──────┬──────┘
                        │
                    M : M
                        │
                        ▼
                 ┌─────────────┐
                 │    Role     │
                 ├─────────────┤
                 │ id          │
                 │ value       │
                 └─────────────┘


                 ┌─────────────┐
                 │ UserSession │
                 ├─────────────┤
                 │ id          │
                 │ token       │
                 │ user        │
                 │ status      │
                 └──────┬──────┘
                        │
                    M : 1
                        │
                        ▼
                     User
```

### User

Contains:

* ID
* Name
* Email
* Password
* Phone number
* Roles

### Role

Represents authorization information associated with users.

### UserSession

Stores:

* Authentication token
* Associated user
* Session status

The current entities use JPA relationships for user-role and session-user associations.

---

# 📨 Kafka Integration

The authentication service publishes a Kafka event after successful signup.

Current flow:

```text
User Signup
     │
     ▼
User persisted
     │
     ▼
Create EmailDto
     │
     ▼
Serialize to JSON
     │
     ▼
Kafka Topic: signup
```

The event contains information such as:

```json
{
  "to": "user@example.com",
  "from": "sender@example.com",
  "subject": "Welcome to Scaler",
  "body": "Have a good learning experience !!"
}
```

This demonstrates an **event-driven architecture**, where user registration can trigger downstream asynchronous processing without requiring the authentication service itself to directly send the email.

---

# 🌐 Eureka Service Discovery

The service is configured as an Eureka client.

```text
                    ┌─────────────────┐
                    │  Eureka Server  │
                    │    :8761        │
                    └────────┬────────┘
                             │
                    Service Registration
                             │
                             ▼
                    ┌─────────────────┐
                    │ UserAuthService │
                    │                 │
                    │     :PORT       │
                    └─────────────────┘
```

Configured service discovery URL:

```text
http://localhost:8761/eureka/
```

The application is configured to both register with Eureka and fetch the registry.

---

# ⚙️ Configuration

The application uses:

```text
src/main/resources/application.properties
```

Important configuration properties include:

```properties
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

spring.datasource.url=jdbc:mysql://localhost:3306/userauthservice
spring.datasource.username=<DB_USERNAME>
spring.datasource.password=<DB_PASSWORD>

eureka.client.fetch-registry=true
eureka.client.register-with-eureka=true
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/

server.port=${SERVER_PORT}
```

> **Never commit real database credentials, JWT secrets, API keys, or other credentials to GitHub.** Use environment variables or a secrets-management system instead.

The current repository's checked-in configuration contains database credentials and should be sanitized before using the repository publicly or in production.

---

# 🧰 Prerequisites

Install the following:

* Java 17+
* Maven 3.8+
* MySQL 8+
* Apache Kafka
* Eureka Server
* Git

---

# 🚀 Getting Started

## 1. Clone the repository

```bash
git clone https://github.com/vikramhankare/UserAuthService.git
```

```bash
cd UserAuthService
```

---

## 2. Create the database

Create a MySQL database:

```sql
CREATE DATABASE userauthservice;
```

---

## 3. Configure the database

Set your database configuration through environment variables or local configuration.

Example:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/userauthservice
spring.datasource.username=root
spring.datasource.password=YOUR_PASSWORD
```

---

## 4. Start Eureka

Start your Eureka Server separately.

By default, this service expects:

```text
http://localhost:8761/eureka/
```

---

## 5. Start Kafka

Start Kafka and ensure the broker is available before testing signup.

The signup flow publishes to:

```text
signup
```

---

## 6. Build the project

Using Maven:

```bash
mvn clean install
```

Or:

```bash
./mvnw clean install
```

Windows:

```cmd
mvnw.cmd clean install
```

---

## 7. Run the application

```bash
mvn spring-boot:run
```

Or:

```bash
./mvnw spring-boot:run
```

Windows:

```cmd
mvnw.cmd spring-boot:run
```

---

# 🧪 Testing the API

## Signup

```bash
curl -X POST http://localhost:8080/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "email": "john@example.com",
    "password": "password123",
    "phoneNumber": "9876543210"
  }'
```

---

## Login

```bash
curl -X POST http://localhost:8080/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john@example.com",
    "password": "password123"
  }'
```

The response contains a `Set-Cookie` header containing the authentication token.

---

## Validate Token

```bash
curl -X POST http://localhost:8080/auth/validateToken \
  -H "Content-Type: application/json" \
  -d '{
    "token": "<JWT_TOKEN>"
  }'
```

---

## Retrieve User

```bash
curl http://localhost:8080/users/1
```

---

# 🧩 Layer Responsibilities

## Controller Layer

Responsible for:

* HTTP request handling
* Request DTO consumption
* Response DTO creation
* HTTP status codes
* API routing

Controllers include:

```text
AuthController
UserController
```

---

## Service Layer

Contains business logic.

### AuthService

Responsible for:

* Signup
* Login
* Role assignment
* JWT generation
* Session creation
* Token validation
* Kafka event publishing

### UserService

Responsible for:

* User retrieval

---

## Repository Layer

Responsible for persistence using Spring Data JPA.

Repositories include:

```text
UserRepo
RoleRepo
SessionRepo
```

This keeps database access separate from business logic.

---

# 🎯 Key Backend Concepts Demonstrated

This project demonstrates several important backend engineering concepts:

### Authentication

```text
Credentials
    ↓
User verification
    ↓
JWT generation
    ↓
Session persistence
```

### Authorization

```text
User
 ↓
Roles
 ↓
JWT access claims
 ↓
Downstream authorization
```

### Persistence

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Hibernate
    ↓
MySQL
```

### Event-Driven Processing

```text
Signup
  ↓
Kafka Event
  ↓
Email/Notification Consumer
```

### Microservice Infrastructure

```text
UserAuthService
       │
       └── Eureka
       
       │
       └── Kafka
       
       │
       └── MySQL
```

---

# 🔒 Security Considerations

This project demonstrates JWT authentication, but the current implementation should be considered **educational rather than production-ready**.

## Password hashing

The current implementation stores and compares passwords directly:

```java
user.setPassword(password);
```

and later:

```java
user.getPassword().equals(password)
```

For production, passwords should never be stored as plaintext.

Recommended approach:

```text
Raw Password
     ↓
BCrypt / Argon2
     ↓
Password Hash
     ↓
Database
```

Spring Security's `PasswordEncoder` with BCrypt would be an appropriate improvement.

---

## JWT Secret Management

The current `AuthConfig` generates an HS256 key when the application starts.

For a distributed production environment, the signing key should be managed externally using:

* Environment variables
* Vault
* AWS Secrets Manager
* Azure Key Vault
* Kubernetes Secrets
* Another dedicated secret-management solution

A persistent key is required if multiple service instances need to validate tokens generated by one another.

---

## Database Credentials

Database credentials should not be committed to source control.

Use:

```properties
spring.datasource.username=${DB_USERNAME}
spring.datasource.password=${DB_PASSWORD}
```

instead.

---

## Cookie Security

The login response currently writes the JWT into a `Set-Cookie` header.

For production, authentication cookies should consider:

```text
HttpOnly
Secure
SameSite
Path
Max-Age
```

rather than constructing the cookie value manually.

---

# ⚠️ Current Limitations

The repository currently has several areas that should be improved before production use:

* Passwords are not hashed.
* Database credentials are present in `application.properties`.
* Authentication cookie configuration is minimal.
* JWT secret generation is not externally managed.
* No Spring Security filter chain is currently used for automatic endpoint authentication.
* Forgot-password functionality is not currently implemented despite being mentioned in the repository description.
* Logout/revocation API is not currently exposed.
* Token expiry is hard-coded.
* Exception handling can be improved with custom exceptions and centralized error responses.
* Request validation can be added with Bean Validation.
* More comprehensive integration tests would improve confidence.

These are intentionally listed as limitations rather than being presented as existing functionality.

---

# 📈 Recommended Future Improvements

### Security

* [ ] Add BCrypt/Argon2 password hashing
* [ ] Add Spring Security
* [ ] Add JWT authentication filter
* [ ] Move JWT secret to environment/secrets manager
* [ ] Configure secure HTTP-only cookies
* [ ] Add CSRF protection where applicable
* [ ] Add rate limiting for login
* [ ] Add account lockout / brute-force protection

### Authentication

* [ ] Add logout endpoint
* [ ] Add refresh tokens
* [ ] Add forgot-password flow
* [ ] Add email verification
* [ ] Add password reset tokens
* [ ] Add token revocation
* [ ] Make JWT expiration configurable

### API

* [ ] Add request validation with `@Valid`
* [ ] Add global exception handler
* [ ] Add standardized API error responses
* [ ] Add Swagger / OpenAPI
* [ ] Add API versioning

### Infrastructure

* [ ] Add Dockerfile
* [ ] Add Docker Compose
* [ ] Containerize MySQL
* [ ] Containerize Kafka
* [ ] Containerize Eureka
* [ ] Add GitHub Actions CI/CD
* [ ] Add Spring Boot Actuator
* [ ] Add health checks
* [ ] Add distributed tracing

### Testing

* [ ] Add controller tests
* [ ] Add service unit tests
* [ ] Add repository tests
* [ ] Add integration tests
* [ ] Add Testcontainers for MySQL and Kafka
* [ ] Add authentication flow tests

---

# 📚 What This Project Demonstrates

From a backend-engineering perspective, the project covers:

```text
Spring Boot
├── REST APIs
├── Dependency Injection
├── DTO Pattern
├── Service Layer
├── Repository Pattern
│
├── Spring Data JPA
├── Hibernate
├── MySQL
│
├── JWT Authentication
├── Role-Based Authorization
├── Session Management
│
├── Apache Kafka
├── Event-Driven Architecture
│
└── Eureka Service Discovery
```

The combination of JWT, persisted sessions, Kafka events, JPA relationships, and Eureka makes this more representative of a **microservice backend architecture** than a simple CRUD application.

---

# 👨‍💻 Author

**Vikram Hankare**

GitHub: https://github.com/vikramhankare


LinkedIn: www.linkedin.com/in/vikramhankare19

