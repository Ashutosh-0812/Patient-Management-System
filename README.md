# Patient Management System

A modern, scalable microservices-based Patient Management System built with Spring Boot, demonstrating enterprise-grade architecture patterns including API Gateway, Authentication, gRPC, Kafka event streaming, and AWS CDK infrastructure as code.

---

## 🏗️ Architecture Overview

This system implements a microservices architecture with the following components:

```
                                    ┌─────────────────┐
                                    │   API Gateway   │
                                    │   (Port 4004)   │
                                    └────────┬────────┘
                                             │
                    ┌────────────────────────┼────────────────────────┐
                    │                        │                        │
            ┌───────▼────────┐      ┌───────▼────────┐      ┌────────▼───────┐
            │ Auth Service   │      │ Patient Service│      │ Other Services │
            │  (Port 4005)   │      │  (Port 4000)   │      │                │
            └────────────────┘      └───────┬────────┘      └────────────────┘
                                            │
                            ┌───────────────┼───────────────┐
                            │               │               │
                    ┌───────▼────────┐     │      ┌────────▼────────┐
                    │ Billing Service│     │      │ Kafka (Events)  │
                    │  gRPC: 9001    │     │      └────────┬────────┘
                    │  HTTP: 4001    │     │               │
                    └────────────────┘     │      ┌────────▼────────┐
                                           │      │ Analytics Svc   │
                                           │      └─────────────────┘
                                    ┌──────▼──────┐
                                    │ PostgreSQL  │
                                    │  Databases  │
                                    └─────────────┘
```

---

## 📦 Microservices

### 1. **API Gateway** (Port: 4004)
- **Technology**: Spring Cloud Gateway
- **Purpose**: Single entry point for all client requests
- **Features**:
  - Routing to microservices
  - JWT token validation via custom filter
  - Path-based routing with predicates
  - API documentation aggregation
  - Cross-cutting concerns (authentication, rate limiting)

**Routes**:
- `/auth/**` → Auth Service
- `/api/patients/**` → Patient Service
- `/api-docs/*` → Service-specific OpenAPI docs

### 2. **Auth Service** (Port: 4005)
- **Technology**: Spring Boot, Spring Security, JWT
- **Purpose**: Authentication and authorization
- **Features**:
  - User login with JWT token generation
  - Token validation endpoint
  - PostgreSQL database for user storage
  - Swagger/OpenAPI documentation

**Endpoints**:
- `POST /login` - Authenticate user and generate JWT
- `GET /validate` - Validate JWT token

**Database**: PostgreSQL (User credentials)

### 3. **Patient Service** (Port: 4000)
- **Technology**: Spring Boot, JPA, gRPC Client, Kafka Producer
- **Purpose**: Core patient management operations
- **Features**:
  - CRUD operations for patient records
  - PostgreSQL database with JPA/Hibernate
  - Kafka event publishing for patient lifecycle events
  - gRPC client for billing service integration
  - Input validation with custom validation groups
  - RESTful API with OpenAPI documentation

**Endpoints**:
- `GET /patients` - Retrieve all patients
- `POST /patients` - Create new patient
- `PUT /patients/{id}` - Update patient
- `DELETE /patients/{id}` - Delete patient

**Integrations**:
- **Kafka**: Publishes `PatientEvent` (protobuf) to "patient" topic
- **gRPC**: Calls Billing Service to create billing accounts
- **Database**: PostgreSQL (Patient data)

**Patient Model**:
- `id` (UUID)
- `name` (String, required)
- `email` (String, unique, email validation)
- `address` (String, required)
- `dateOfBirth` (LocalDate, required)
- `registeredDate` (LocalDate, required)

### 4. **Billing Service** (Port: 4001, gRPC: 9001)
- **Technology**: Spring Boot, gRPC Server
- **Purpose**: Manage billing accounts via gRPC
- **Features**:
  - gRPC service implementation
  - Protobuf message definitions
  - Business logic for billing operations

**gRPC Service**:
- `CreateBillingAccount(BillingRequest) → BillingResponse`
  - Request: patientId, name, email
  - Response: accountId, status

**Protocol Buffers**: Located in `src/main/proto/billing_service.proto`

### 5. **Analytics Service** (Port: TBD)
- **Technology**: Spring Boot, Kafka Consumer
- **Purpose**: Consume and analyze patient events
- **Features**:
  - Kafka consumer for patient events
  - Event processing and analytics
  - Protobuf deserialization

**Kafka Consumer**:
- Topic: "patient"
- Group ID: "analytics-service"
- Processes: PatientEvent (create, update, delete)

---

## 🔧 Technologies & Patterns

### Core Technologies
- **Java 17+** - Programming language
- **Spring Boot 3.x** - Microservices framework
- **Spring Cloud Gateway** - API Gateway
- **Spring Security** - Authentication/Authorization
- **Spring Data JPA** - Database access
- **PostgreSQL** - Relational database
- **Apache Kafka** - Event streaming platform
- **gRPC** - High-performance RPC framework
- **Protocol Buffers** - Serialization format
- **Maven** - Build tool
- **Docker** - Containerization
- **AWS CDK** - Infrastructure as Code

### Architecture Patterns
- **Microservices Architecture** - Service decomposition
- **API Gateway Pattern** - Single entry point
- **Database per Service** - Data isolation
- **Event-Driven Architecture** - Kafka messaging
- **Synchronous Communication** - REST & gRPC
- **Asynchronous Communication** - Kafka events
- **JWT Authentication** - Stateless security
- **Gateway Filter Pattern** - Cross-cutting concerns

---

## 🚀 Getting Started

### Prerequisites
- Java 17 or higher
- Maven 3.8+
- Docker & Docker Compose (for databases, Kafka)
- PostgreSQL (or use Docker)
- Kafka & Zookeeper (or use Docker)

### Building the Project

Each service is a separate Maven project. Build all services:

```bash
# Build Auth Service
cd auth-service
./mvnw clean install

# Build Patient Service
cd ../patient-service
./mvnw clean install

# Build Billing Service (includes protobuf compilation)
cd ../billing-service
./mvnw clean install

# Build Analytics Service
cd ../analytics-service
./mvnw clean install

# Build API Gateway
cd ../api-gateway
./mvnw clean install
```

### Running Services Locally

**Option 1: Using Docker** (Recommended)
```bash
# Build Docker images
docker build -t auth-service ./auth-service
docker build -t patient-service ./patient-service
docker build -t billing-service ./billing-service
docker build -t analytics-service ./analytics-service
docker build -t api-gateway ./api-gateway

# Run with docker-compose (if available)
docker-compose up -d
```

**Option 2: Run Individually**

Start infrastructure first:
```bash
# PostgreSQL for Auth Service
docker run -d -p 5432:5432 \
  -e POSTGRES_DB=authdb \
  -e POSTGRES_USER=admin_user \
  -e POSTGRES_PASSWORD=password \
  --name auth-db postgres:15

# PostgreSQL for Patient Service
docker run -d -p 5433:5432 \
  -e POSTGRES_DB=patientdb \
  -e POSTGRES_USER=admin_user \
  -e POSTGRES_PASSWORD=password \
  --name patient-db postgres:15

# Kafka & Zookeeper
docker run -d -p 2181:2181 --name zookeeper zookeeper:3.8
docker run -d -p 9092:9092 \
  -e KAFKA_ZOOKEEPER_CONNECT=zookeeper:2181 \
  --name kafka wurstmeister/kafka:latest
```

Start services:
```bash
# Auth Service
cd auth-service
./mvnw spring-boot:run

# Patient Service
cd patient-service
export SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092
./mvnw spring-boot:run

# Billing Service
cd billing-service
./mvnw spring-boot:run

# Analytics Service
cd analytics-service
export SPRING_KAFKA_BOOTSTRAP_SERVERS=localhost:9092
./mvnw spring-boot:run

# API Gateway
cd api-gateway
./mvnw spring-boot:run
```

---

## 🔐 Authentication Flow

1. **Login**: Client sends credentials to `/auth/login`
2. **Token Generation**: Auth service validates and returns JWT
3. **API Access**: Client includes `Authorization: Bearer <token>` header
4. **Gateway Validation**: API Gateway validates token with Auth Service
5. **Request Forwarding**: Valid requests forwarded to target service

### Example Login Request

```http
POST http://localhost:4004/auth/login
Content-Type: application/json

{
  "email": "testuser@test.com",
  "password": "password123"
}
```

Response:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

### Example Authenticated Request

```http
GET http://localhost:4004/api/patients
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

---

## 📡 Event-Driven Communication

### Patient Events (Kafka)

**Producer**: Patient Service  
**Consumer**: Analytics Service  
**Topic**: `patient`  
**Format**: Protocol Buffers

**Event Types**:
- `PATIENT_CREATED` - New patient registered
- `PATIENT_UPDATED` - Patient information modified
- `PATIENT_DELETED` - Patient removed

**PatientEvent Schema**:
```protobuf
message PatientEvent {
  string patientId = 1;
  string name = 2;
  string email = 3;
  string event_type = 4;
}
```

---

## 🔌 gRPC Communication

### Billing Service Integration

**Client**: Patient Service  
**Server**: Billing Service  
**Port**: 9001  
**Protocol**: gRPC with Protocol Buffers

When a patient is created, Patient Service synchronously calls Billing Service via gRPC to create a billing account.

**Service Definition**:
```protobuf
service BillingService {
  rpc CreateBillingAccount (BillingRequest) returns (BillingResponse);
}

message BillingRequest {
  string patientId = 1;
  string name = 2;
  string email = 3;
}

message BillingResponse {
  string accountId = 1;
  string status = 2;
}
```

---

## 🧪 Testing

### Integration Tests

Run integration tests (requires services running):
```bash
cd integration-tests
mvn test
```

Tests include:
- `AuthIntegrationTest` - Authentication flows
- `PatientIntegrationTest` - Patient CRUD operations with auth

### Sample HTTP Requests

Located in `api-requests/` and `grpc-requests/`:
- `auth-service/login.http`
- `auth-service/validate.http`
- `patient-service/create-patient.http`
- `patient-service/get-patients.http`
- `patient-service/update-patient.http`
- `patient-service/delete-patient.http`
- `billing-service/create-billing-account.http`

---

## 📊 API Documentation

Each service exposes OpenAPI documentation:

- **Auth Service**: http://localhost:4005/v3/api-docs
- **Patient Service**: http://localhost:4000/v3/api-docs
- **Aggregated (via Gateway)**:
  - http://localhost:4004/api-docs/auth
  - http://localhost:4004/api-docs/patients

---

## 🏗️ Infrastructure as Code

### AWS CDK (LocalStack)

The `infrastructure/` module contains AWS CDK code for deploying to LocalStack or AWS.

**Features**:
- VPC and networking
- ECS Fargate clusters
- RDS PostgreSQL databases
- MSK (Managed Kafka)
- Application Load Balancers
- Service discovery with Cloud Map

**Build and Deploy**:
```bash
cd infrastructure
mvn clean package
sh localstack-deploy.sh
```

---

## 📝 Environment Variables

### Patient Service
```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://patient-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
SPRING_SQL_INIT_MODE=always
JAVA_TOOL_OPTIONS=-agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:5005
```

### Auth Service
```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/authdb
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_DATASOURCE_PASSWORD=password
```

### Billing Service
```bash
GRPC_SERVER_PORT=9001
SERVER_PORT=4001
```

### Analytics Service
```bash
SPRING_KAFKA_BOOTSTRAP_SERVERS=kafka:9092
```

### API Gateway
```bash
AUTH_SERVICE_URL=http://auth-service:4005
```

---

## 📁 Project Structure

```
patient-management-system/
├── api-gateway/              # Spring Cloud Gateway
│   ├── src/main/java/
│   │   └── com/pm/apigateway/
│   │       ├── filter/       # JWT validation filter
│   │       └── exception/    # Error handling
│   └── src/main/resources/
│       ├── application.yml   # Gateway routes
│       └── application-prod.yml
│
├── auth-service/             # Authentication & JWT
│   ├── src/main/java/
│   │   └── com/pm/authservice/
│   │       ├── controller/   # REST endpoints
│   │       ├── service/      # Business logic
│   │       ├── repository/   # Database access
│   │       ├── model/        # User entity
│   │       ├── dto/          # Request/Response DTOs
│   │       ├── util/         # JWT utilities
│   │       └── config/       # Security config
│   └── src/main/resources/
│       ├── application.properties
│       └── data.sql          # Initial user data
│
├── patient-service/          # Patient management
│   ├── src/main/java/
│   │   └── com/pm/patientservice/
│   │       ├── controller/   # REST endpoints
│   │       ├── service/      # Business logic
│   │       ├── repository/   # JPA repositories
│   │       ├── model/        # Patient entity
│   │       ├── dto/          # Request/Response DTOs
│   │       ├── mapper/       # Entity-DTO mapping
│   │       ├── kafka/        # Kafka producer
│   │       ├── grpc/         # gRPC client
│   │       └── exception/    # Custom exceptions
│   ├── src/main/proto/       # Protobuf definitions
│   └── src/main/resources/
│       ├── application.properties
│       └── data.sql          # Sample patient data
│
├── billing-service/          # Billing via gRPC
│   ├── src/main/java/
│   │   └── com/pm/billingservice/
│   │       └── grpc/         # gRPC service impl
│   └── src/main/proto/
│       └── billing_service.proto
│
├── analytics-service/        # Event processing
│   ├── src/main/java/
│   │   └── com/pm/analyticsservice/
│   │       └── kafka/        # Kafka consumer
│   └── src/main/proto/
│       └── patient_event.proto
│
├── infrastructure/           # AWS CDK
│   └── src/main/java/
│       └── com/pm/stack/
│           └── LocalStack.java  # CDK stack definition
│
├── integration-tests/        # E2E tests
│   └── src/test/java/
│       ├── AuthIntegrationTest.java
│       └── PatientIntegrationTest.java
│
├── api-requests/             # Sample HTTP requests
└── grpc-requests/            # Sample gRPC requests
```

---

## 🔍 Key Features Demonstrated

✅ **Microservices Architecture** - Service decomposition and independence  
✅ **API Gateway Pattern** - Centralized routing and security  
✅ **JWT Authentication** - Stateless token-based security  
✅ **Event-Driven Architecture** - Kafka for asynchronous messaging  
✅ **gRPC Communication** - High-performance RPC between services  
✅ **Protocol Buffers** - Efficient serialization  
✅ **Database per Service** - Data isolation pattern  
✅ **Spring Cloud Gateway** - Modern reactive gateway  
✅ **Custom Gateway Filters** - JWT validation at gateway level  
✅ **OpenAPI Documentation** - Auto-generated API docs  
✅ **Validation Groups** - Custom validation for create vs update  
✅ **Docker Support** - Containerized microservices  
✅ **Infrastructure as Code** - AWS CDK for cloud deployment  
✅ **Integration Testing** - End-to-end API testing  

---

## 🛠️ Development Notes

### gRPC Setup (Billing Service)

Add dependencies to `pom.xml`:
```xml
<!--GRPC -->
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-netty-shaded</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-protobuf</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency>
    <groupId>io.grpc</groupId>
    <artifactId>grpc-stub</artifactId>
    <version>1.69.0</version>
</dependency>
<dependency> <!-- necessary for Java 9+ -->
    <groupId>org.apache.tomcat</groupId>
    <artifactId>annotations-api</artifactId>
    <version>6.0.53</version>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>net.devh</groupId>
    <artifactId>grpc-spring-boot-starter</artifactId>
    <version>3.1.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>4.29.1</version>
</dependency>
```

Replace the `<build>` section with the following:
```xml
<build>
    <extensions>
        <!-- Ensure OS compatibility for protoc -->
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.7.0</version>
        </extension>
    </extensions>
    <plugins>
        <!-- Spring boot / maven  -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>

        <!-- PROTO -->
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.25.5:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.68.1:exe:${os.detected.classifier}</pluginArtifact>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### Debugging

Remote debugging is enabled on Patient Service (port 5005). Connect your IDE debugger to:
```
localhost:5005
```

### Common Issues

**Kafka Connection**: Ensure Kafka is running and accessible at configured bootstrap servers  
**Database Connection**: Verify PostgreSQL is running and credentials match  
**gRPC Port Conflict**: Check port 9001 is available for Billing Service  
**JWT Token Expired**: Re-login to get a fresh token  

---

## 📈 Future Enhancements

- [ ] Service mesh (Istio/Linkerd) for advanced traffic management
- [ ] Distributed tracing (Zipkin/Jaeger)
- [ ] Centralized logging (ELK Stack)
- [ ] Circuit breaker pattern (Resilience4j)
- [ ] API rate limiting and throttling
- [ ] GraphQL gateway as alternative to REST
- [ ] WebSocket support for real-time updates
- [ ] Multi-tenancy support
- [ ] Advanced analytics dashboards
- [ ] CQRS pattern implementation
- [ ] Saga pattern for distributed transactions

---

## 👥 Contributing

This is a learning project demonstrating microservices patterns. Feel free to fork and experiment!

---

## 📚 Learning Resources

This project demonstrates concepts from:
- Spring Boot Microservices
- Spring Cloud Gateway
- Event-Driven Architecture
- Domain-Driven Design
- gRPC & Protocol Buffers
- Apache Kafka
- Container Orchestration
- Infrastructure as Code

---

## 📄 License

Copyright © 2025 Code Jackal | Original Course Material by Chris Blakely

---

## 💬 Community

**Join the Discord Community**

This source code is for the Java/Spring microservices course available on YouTube.  
Join the Discord for help and discussion: https://discord.gg/nCrDnfCE

---

## 🎯 Quick Start Commands

```bash
# Clone repository
git clone <repository-url>
cd patient-management-system

# Start infrastructure (PostgreSQL, Kafka)
docker-compose up -d

# Build all services
for service in auth-service patient-service billing-service analytics-service api-gateway; do
  cd $service && ./mvnw clean install && cd ..
done

# Run API Gateway
cd api-gateway && ./mvnw spring-boot:run

# Test authentication
curl -X POST http://localhost:4004/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"testuser@test.com","password":"password123"}'

# Test patient service (replace TOKEN with actual JWT)
curl -X GET http://localhost:4004/api/patients \
  -H "Authorization: Bearer TOKEN"
```

---

**Built with ❤️ using Spring Boot and Microservices Architecture**
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.6.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.25.5:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.68.1:exe:${os.detected.classifier}</pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```


# Auth service

Dependencies (add in addition to whats there)

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-api</artifactId>
            <version>0.12.6</version>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-impl</artifactId>
            <version>0.12.6</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>io.jsonwebtoken</groupId>
            <artifactId>jjwt-jackson</artifactId>
            <version>0.12.6</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.postgresql</groupId>
            <artifactId>postgresql</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.springdoc</groupId>
            <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
            <version>2.6.0</version>
        </dependency>
        <dependency>
          <groupId>com.h2database</groupId>
          <artifactId>h2</artifactId>
        </dependency>
       
```

## Environment Variables

```
SPRING_DATASOURCE_PASSWORD=password
SPRING_DATASOURCE_URL=jdbc:postgresql://auth-service-db:5432/db
SPRING_DATASOURCE_USERNAME=admin_user
SPRING_JPA_HIBERNATE_DDL_AUTO=update
SPRING_SQL_INIT_MODE=always
```


## Data.sql

```sql
-- Ensure the 'users' table exists
CREATE TABLE IF NOT EXISTS "users" (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role VARCHAR(50) NOT NULL
);

-- Insert the user if no existing user with the same id or email exists
INSERT INTO "users" (id, email, password, role)
SELECT '223e4567-e89b-12d3-a456-426614174006', 'testuser@test.com',
       '$2b$12$7hoRZfJrRKD2nIm2vHLs7OBETy.LWenXXMLKf99W8M4PUwO6KB7fu', 'ADMIN'
WHERE NOT EXISTS (
    SELECT 1
    FROM "users"
    WHERE id = '223e4567-e89b-12d3-a456-426614174006'
       OR email = 'testuser@test.com'
);



```


# Auth Service DB

## Environment Variables

```
POSTGRES_DB=db;POSTGRES_PASSWORD=password;POSTGRES_USER=admin_user
```