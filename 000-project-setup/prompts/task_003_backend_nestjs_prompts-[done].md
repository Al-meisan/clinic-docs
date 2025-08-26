# PROMPTS: TASK-003 - Backend NestJS Application Setup

# ~~PROMPT 1: Initial NestJS Project Foundation~~

Before any change analyze the existing code base in clinic-backend and Only implement missing requirement and criteria,
execute this prompt with the minimum required changes.

Based on the existing code in clinic-backend update NestJS application foundation with TypeScript, proper folder
structure, and basic configuration following domain-driven design principles. Update the project with essential
dependencies, configure TypeScript, ESLint, and set up the main application entry points with proper module structure.

## PROJECT CONTEXT

**Tech Stack**: TypeScript with NestJS, AWS services integration, PostgreSQL database
**Architecture**: Domain-driven design with feature modules, dependency injection
**Code Standards**: ESLint and Prettier for code quality, Class-validator for input validation
**Folder Structure**: Feature-based modules with shared infrastructure components

## EPIC CONTEXT

Building the foundational backend for a healthcare clinic management system that will handle patient records,
appointments, clinical documentation, and provider workflows. This backend serves as the API layer for React frontend
applications.

## TASK CONTEXT

**Goal**: Initialize complete NestJS backend application foundation
**Why**: Creates core backend foundation for all API endpoints, business logic, and data management
**Priority**: HIGH - Foundation must be established before any other development

## BUSINESS DOMAIN CONTEXT

**Healthcare Domain**: OAuth2 scope-based authorization, multi-tenant architecture with clinic boundaries, audit trail
for all data modifications
**Compliance**: Healthcare data requires proper security, validation, and audit capabilities

## FUNCTIONAL REQUIREMENTS

1. **NestJS Application Structure**
    - Main application entry point with proper bootstrapping
    - Root module with configuration imports
    - Feature-based folder structure ready for domain modules

2. **Configuration Management**
    - Environment variable loading and validation
    - Separate configurations for different environments
    - Secure handling of sensitive configuration values

3. **Core Infrastructure Setup**
    - Common utilities, decorators, and shared interfaces
    - Exception handling foundation
    - Logging system integration points

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  core:
    - "@nestjs/core": "^10.0.0"
    - "@nestjs/common": "^10.0.0"
    - "@nestjs/config": "^3.0.0"
    - "reflect-metadata": "^0.1.13"
    - "rxjs": "^7.0.0"
  development:
    - "@nestjs/cli": "^10.0.0"
    - "typescript": "^5.0.0"
    - "@types/node": "^20.0.0"
    - "eslint": "^8.0.0"
    - "prettier": "^3.0.0"

folder_structure:
  src/:
    - main.ts (application entry point)
    - app.module.ts (root module)
    - common/ (shared utilities and infrastructure)
    - config/ (configuration management)
    - features/ (domain modules placeholder)
    - shared/ (shared modules/services)

startup_time: "< 5 seconds for development server"
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Application bootstrap process
- Configuration loading and validation
- Module imports and dependency injection

**Integration Tests**:

- Application startup and shutdown
- Health check endpoint functionality
- Configuration environment switching

## DOCUMENTATION REQUIREMENTS

- README.md with setup instructions
- Package.json with proper scripts configuration
- TypeScript configuration documentation
- Folder structure and naming conventions guide

## VALIDATION CRITERIA

- [ ] NestJS application starts successfully on port 3000
- [ ] All TypeScript code compiles without errors
- [ ] ESLint passes with zero warnings/errors
- [ ] Environment configuration loads properly
- [ ] Application follows established folder structure
- [ ] Package.json includes all required scripts (start, build, test, lint)
- [ ] Git repository is properly initialized with .gitignore
- [ ] Development server restarts automatically on file changes

only implement the missing criteria

# ~~PROMPT 2: Database Integration with TypeORM and PostgreSQL~~

Before any change analyze the existing code base in clinic-backend and Only implement missing requirement and criteria,
execute this prompt with the minimum required changes.

Set up TypeORM integration with PostgreSQL database connection, configuration management, migration system, seeding
system and base entity classes. Create database connection module, establish connection pooling, and implement the
foundation for database operations with proper error handling.

## PROJECT CONTEXT

**Database**: PostgreSQL as primary database with TypeORM ORM
**Architecture**: Repository pattern for data access, base entities for shared functionality
**Environment**: Different database configurations for development, staging, and production
**Migration**: Forward and backward migration capabilities required

## EPIC CONTEXT

Healthcare clinic management system requires robust data persistence for patient records, appointments, clinical
documentation, and provider information. Database operations must be secure, auditable, and performant.

## TASK CONTEXT

**Goal**: PostgreSQL database connection with TypeORM integration
**Why**: Foundation for all data persistence and retrieval operations
**Priority**: HIGH - Required before any feature development
**Dependencies**: PROMPT 1 (NestJS foundation) must be completed first

## BUSINESS DOMAIN CONTEXT

**Healthcare Data**: Patient information, clinical records, appointment scheduling
**Compliance**: HIPAA compliance requires audit trails, secure data handling
**Multi-tenancy**: Clinic-based data isolation and access control

## FUNCTIONAL REQUIREMENTS

1. **Database Connection Setup**
    - PostgreSQL connection with connection pooling
    - Environment-specific configuration management
    - Connection health monitoring and error handling

2. **TypeORM Integration**
    - Entity management with decorators
    - Migration system for schema changes
    - Query builder and repository patterns

3. **Base Entity Framework**
    - Common entity fields (id, createdAt, updatedAt, deletedAt)
    - Soft delete functionality
    - Audit trail foundations

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  database:
    - "@nestjs/typeorm": "^10.0.0"
    - "typeorm": "^0.3.17"
    - "pg": "^8.11.0"
    - "@types/pg": "^8.10.0"

database_config:
  connection_pooling:
    min_connections: 5
    max_connections: 20
    idle_timeout: 30000
  performance:
    query_timeout: 30000
    connection_timeout: 5000
  features:
    migrations: true
    synchronize: false (production)
    logging: true (development)
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Database configuration loading
- Connection pool management
- Base entity operations

**Integration Tests**:

- Database connection establishment
- Basic CRUD operations
- Migration execution (up/down)
- Connection error handling

## DOCUMENTATION REQUIREMENTS

- Database setup and configuration guide
- Migration workflow documentation
- Entity relationship diagrams (when entities are added)
- Connection troubleshooting guide

## VALIDATION CRITERIA

- [ ] Database connection establishes successfully
- [ ] TypeORM entities can be created and managed
- [ ] Migration system works (generate, run, revert)
- [ ] Connection pooling operates within configured limits
- [ ] Database queries log properly in development
- [ ] Base entity provides common functionality
- [ ] Soft delete functionality works correctly
- [ ] Connection errors are handled gracefully
- [ ] Different environments use appropriate database configs

# ~~PROMPT 3: Authentication Framework and JWT Integration~~

Implement authentication and authorization framework using JWT tokens, guards, and decorators for role-based access
control. Set up authentication middleware, create custom guards for route protection, and establish the foundation for
OAuth2 scope-based authorization.

## PROJECT CONTEXT

**Authentication**: JWT-based authentication with role and scope validation
**Authorization**: OAuth2 scope-based system with healthcare role hierarchy
**Security**: Route protection, token validation, and secure credential handling
**Architecture**: Guard and decorator patterns for clean authorization implementation

## EPIC CONTEXT

Healthcare clinic management requires secure access control with different permission levels for System Administrators,
Clinic Managers, Healthcare Providers, Clinical Staff, and Administrative Staff. Each role has specific scopes and
access permissions.

## TASK CONTEXT

**Goal**: JWT authentication with role-based authorization framework
**Why**: Security foundation for all protected endpoints and business logic
**Priority**: HIGH - Required before implementing any protected features
**Dependencies**: PROMPT 1 (NestJS foundation) must be completed first

## BUSINESS DOMAIN CONTEXT

**Healthcare Roles**:

- System Administrator (`admin:full`)
- Clinic Manager (`clinic:manage` + operational scopes)
- Healthcare Provider (clinical + prescription scopes)
- Clinical Staff (clinical scopes, limited patient write)
- Administrative Staff (patient + appointment scopes, limited clinical read)

**OAuth2 Scopes**: `admin:full`, `clinic:manage`, `patient:write`, `patient:read`, `appointment:write`,
`appointment:read`, `clinical:write`, `clinical:read`, `billing:write`, `billing:read`, `prescription:write`,
`prescription:read`

## FUNCTIONAL REQUIREMENTS

1. **JWT Authentication System**
    - Token generation and validation
    - Token expiration and refresh handling
    - Secure token storage and transmission

2. **Authorization Guards and Decorators**
    - Route-level protection with custom guards
    - Role-based access control decorators
    - Scope validation for specific operations

3. **Authentication Middleware**
    - Request authentication parsing
    - User context injection
    - Error handling for authentication failures

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  auth:
    - "@nestjs/jwt": "^10.0.0"
    - "@nestjs/passport": "^10.0.0"
    - "passport": "^0.6.0"
    - "passport-jwt": "^4.0.0"
    - "passport-local": "^1.0.0"
    - "@types/passport-jwt": "^3.0.0"

security_config:
  jwt:
    secret_key: "environment_variable"
    expiration: "1h"
    refresh_expiration: "7d"
  password:
    bcrypt_rounds: 12
    complexity_requirements: true
```

## TESTING REQUIREMENTS

**Unit Tests**:

- JWT token generation and validation
- Guard authorization logic
- Decorator functionality
- Role and scope validation

**Integration Tests**:

- Complete authentication flow
- Protected route access
- Invalid token handling
- Role-based endpoint access

## DOCUMENTATION REQUIREMENTS

- Authentication flow documentation
- Role and scope definition guide
- Guard and decorator usage examples
- Security best practices guide

## VALIDATION CRITERIA

- [ ] JWT tokens generate and validate correctly
- [ ] Authentication guard protects routes properly
- [ ] Role-based authorization works for different user types
- [ ] OAuth2 scopes are validated correctly
- [ ] Custom decorators (@Roles, @Scopes) function properly
- [ ] Authentication errors return appropriate HTTP status codes
- [ ] Token expiration handling works correctly
- [ ] User context is properly injected into request objects
- [ ] Password hashing and validation implemented securely

# ~~PROMPT 4: Configuration Management and Environment Setup~~

Create a comprehensive configuration management system for handling environment-specific settings, AWS service
configurations, and secure secret management. Implement configuration validation, environment detection, and proper
separation of concerns for different deployment environments.

## PROJECT CONTEXT

**Configuration**: Environment-based configuration with validation and type safety
**Security**: Secure handling of database credentials, JWT secrets, and AWS configurations
**Environments**: Development, staging, and production environment support
**Validation**: Schema validation for all configuration values

## EPIC CONTEXT

Healthcare application requires secure configuration management for database connections, AWS services, authentication
secrets, and environment-specific settings. Configuration must be validated, secure, and easily manageable across
different deployment environments.

## TASK CONTEXT

**Goal**: Comprehensive configuration management with environment validation
**Why**: Foundation for secure and maintainable environment-specific settings
**Priority**: MEDIUM - Supports all other development but not immediately blocking
**Dependencies**: PROMPT 1 (NestJS foundation) must be completed first

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance**: Secure credential management and audit trail requirements
**Multi-Environment**: Development, staging, and production configurations
**AWS Integration**: Configuration for Cognito, S3, and other AWS services

## FUNCTIONAL REQUIREMENTS

1. **Environment Configuration**
    - Environment-specific configuration files
    - Environment variable loading and validation
    - Default value handling and overrides

2. **Configuration Validation**
    - Schema validation using Joi
    - Type-safe configuration objects
    - Required vs optional setting validation

3. **Secure Secret Management**
    - Environment variable encryption for secrets
    - Secure credential handling patterns
    - No hardcoded sensitive values

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  config:
    - "@nestjs/config": "^3.0.0"
    - "joi": "^17.9.0"
    - "dotenv": "^16.0.0"

config_categories:
  database:
    - host, port, username, password, database_name
    - connection_pool_settings
    - ssl_configuration
  auth:
    - jwt_secret, jwt_expiration
    - bcrypt_rounds
    - refresh_token_settings
  aws:
    - region, access_key, secret_key
    - cognito_configuration
    - s3_bucket_settings
  application:
    - port, environment, log_level
    - cors_settings
    - rate_limiting_configuration
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Configuration loading and validation
- Environment-specific value resolution
- Schema validation with valid/invalid inputs
- Default value handling

**Integration Tests**:

- Application startup with different configurations
- Environment variable override behavior
- Configuration error handling

## DOCUMENTATION REQUIREMENTS

- Environment setup guide for developers
- Configuration variable documentation
- Security guidelines for credential management
- Environment-specific deployment notes

## VALIDATION CRITERIA

- [ ] Configuration loads successfully for all environments
- [ ] Schema validation catches invalid configuration values
- [ ] Environment variables override default values correctly
- [ ] Sensitive values are never logged or exposed
- [ ] Application fails gracefully with invalid configuration
- [ ] Configuration is type-safe with proper TypeScript interfaces
- [ ] Default values work correctly for optional settings
- [ ] Different environments load appropriate configuration
- [ ] AWS service configurations are properly structured

# ~~PROMPT 5: Health Check Endpoint and Basic Middleware~~

Implement health check endpoint for application monitoring, create basic middleware for request logging and response
formatting, and establish foundation for API documentation with OpenAPI/Swagger integration.

## PROJECT CONTEXT

**Monitoring**: Health check endpoints for application and database status monitoring
**Middleware**: Request/response processing, logging, and error handling
**Documentation**: OpenAPI/Swagger documentation foundation
**Operations**: Support for deployment and operational monitoring

## EPIC CONTEXT

Healthcare clinic management system requires reliable monitoring and operational visibility. Health checks enable
automated monitoring, while middleware provides consistent API behavior and logging for troubleshooting and audit
requirements.

## TASK CONTEXT

**Goal**: Health check endpoint and basic middleware infrastructure
**Why**: Operational foundation for monitoring, logging, and API consistency
**Priority**: MEDIUM - Important for operations but not blocking feature development
**Dependencies**: PROMPT 1 (NestJS foundation) and PROMPT 2 (Database) must be completed

## BUSINESS DOMAIN CONTEXT

**Healthcare Operations**: System uptime and reliability critical for patient care
**Audit Requirements**: Request logging for compliance and troubleshooting
**API Consistency**: Standardized response formats and error handling

## FUNCTIONAL REQUIREMENTS

1. **Health Check Endpoint**
    - Application health status reporting
    - Database connectivity verification
    - System resource and dependency checks

2. **Request/Response Middleware**
    - Request logging with correlation IDs
    - Response formatting standardization
    - Response time tracking

3. **API Documentation Foundation**
    - OpenAPI/Swagger integration
    - Endpoint documentation structure
    - API version management

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  monitoring:
    - "@nestjs/terminus": "^10.0.0"
    - "@nestjs/swagger": "^7.0.0"
    - "swagger-ui-express": "^5.0.0"

health_checks:
  application:
    - startup_time
    - memory_usage
    - uptime
  database:
    - connection_status
    - query_response_time
  external_services:
    - aws_service_connectivity

middleware_features:
  logging:
    - request_method_url_timestamp
    - response_status_duration
    - correlation_id_tracking
  formatting:
    - consistent_response_structure
    - error_message_standardization
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Health check logic for different scenarios
- Middleware request/response processing
- Response formatting functions

**Integration Tests**:

- Health endpoint returns correct status codes
- Database health check accuracy
- Middleware integration with endpoints
- OpenAPI documentation generation

## DOCUMENTATION REQUIREMENTS

- Health check endpoint documentation
- Middleware configuration guide
- API response format standards
- OpenAPI documentation setup guide

## VALIDATION CRITERIA

- [ ] Health check endpoint returns 200 when all systems healthy
- [ ] Health check returns appropriate error codes when systems unhealthy
- [ ] Database connectivity check works correctly
- [ ] Request logging middleware captures all required information
- [ ] Response formatting middleware creates consistent API responses
- [ ] Correlation IDs are generated and tracked properly
- [ ] OpenAPI documentation is generated and accessible
- [ ] Swagger UI interface displays API documentation correctly
- [ ] Health endpoint includes version and timestamp information

# ~~PROMPT 6: Error Handling and Logging Infrastructure~~

Create comprehensive error handling system with custom exception filters, structured logging implementation, and error
response standardization. Set up logging infrastructure for different environments with proper log levels and formats.

## PROJECT CONTEXT

**Error Handling**: Global exception filters and custom business exceptions
**Logging**: Structured logging with environment-specific configurations
**Monitoring**: Error tracking and performance logging for operational visibility
**Standards**: Consistent error responses and logging formats

## EPIC CONTEXT

Healthcare application requires robust error handling for patient safety and regulatory compliance. All errors must be
properly logged, tracked, and handled gracefully while providing appropriate feedback to users without exposing
sensitive information.

## TASK CONTEXT

**Goal**: Comprehensive error handling and logging infrastructure
**Why**: Foundation for application reliability, debugging, and compliance
**Priority**: MEDIUM - Important for production readiness
**Dependencies**: PROMPT 1 (NestJS foundation) must be completed first

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance**: Error logging required for audit trails and troubleshooting
**Patient Safety**: Graceful error handling prevents system failures during patient care
**Security**: Error messages must not expose sensitive healthcare information

## FUNCTIONAL REQUIREMENTS

1. **Global Exception Handling**
    - Custom exception filter for all unhandled errors
    - Business logic exception classes
    - HTTP status code standardization

2. **Structured Logging System**
    - Environment-specific log levels and formats
    - Request correlation tracking
    - Performance and error logging

3. **Error Response Standardization**
    - Consistent error response format
    - User-friendly error messages
    - Error code classification system

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  logging:
    - "winston": "^3.10.0"
    - "nest-winston": "^1.9.0"
    - "@nestjs/common": "^10.0.0"

logging_levels:
  development:
    - console_transport
    - debug_level
    - detailed_error_stacks
  production:
    - file_transport
    - info_level
    - sanitized_error_messages

error_categories:
  validation_errors: 400
  business_logic_errors: 422
  authentication_errors: 401
  authorization_errors: 403
  not_found_errors: 404
  internal_server_errors: 500
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Exception filter error handling
- Custom exception class behavior
- Logging format and level validation
- Error response structure

**Integration Tests**:

- Global error handling in API requests
- Log output verification
- Error response consistency
- Different error scenario handling

## DOCUMENTATION REQUIREMENTS

- Error handling architecture documentation
- Logging configuration and usage guide
- Error code reference documentation
- Troubleshooting and debugging guide

## VALIDATION CRITERIA

- [ ] Global exception filter catches all unhandled errors
- [ ] Custom business exceptions are properly categorized
- [ ] Error responses follow consistent format structure
- [ ] Logging works correctly for different environments
- [ ] Log correlation IDs track requests properly
- [ ] Error messages are user-friendly and secure
- [ ] Internal errors don't expose sensitive information
- [ ] Performance logging captures response times
- [ ] Log levels can be configured per environment
- [ ] Error stacks are logged in development but sanitized in production

# ~~PROMPT 7: Testing Foundation and CI/CD Preparation~~

For this prompt some part are already implement either fully or partially, review the current implementation and only
implement missing criteria and objective with the least amount of change possible

Set up comprehensive testing infrastructure with unit tests, integration tests, and end-to-end testing capabilities.
Configure testing utilities, create test database setup, and prepare foundation for CI/CD pipeline integration.

## PROJECT CONTEXT

**Testing Strategy**: Unit, integration, and e2e testing with Jest and Supertest
**Test Environment**: Isolated test database and configuration
**Coverage**: Minimum 80% code coverage requirement
**CI/CD**: Preparation for automated testing in deployment pipeline

## EPIC CONTEXT

Healthcare application requires comprehensive testing for patient safety and regulatory compliance. Testing
infrastructure must support TDD/BDD practices and ensure all business logic is properly validated before deployment.

## TASK CONTEXT

**Goal**: Complete testing infrastructure and CI/CD preparation
**Why**: Foundation for reliable, tested code deployment
**Priority**: MEDIUM - Important for development workflow
**Dependencies**: All previous prompts (1-6) should be completed first

## BUSINESS DOMAIN CONTEXT

**Healthcare Quality**: Patient safety requires thoroughly tested software
**Regulatory Compliance**: Testing documentation required for healthcare compliance
**Business Logic**: Critical healthcare workflows must be validated through testing

## FUNCTIONAL REQUIREMENTS

1. **Unit Testing Infrastructure**
    - Jest configuration for NestJS
    - Service and controller testing utilities
    - Mock and spy setup for dependencies

2. **Integration Testing Setup**
    - Test database configuration
    - API endpoint testing with Supertest
    - Database transaction rollback for test isolation

3. **Test Utilities and Helpers**
    - Test data factories and fixtures
    - Common testing utilities and matchers
    - Performance and load testing foundations

## TECHNICAL REQUIREMENTS

```yaml
dependencies:
  testing:
    - "@nestjs/testing": "^10.0.0"
    - "jest": "^29.0.0"
    - "supertest": "^6.3.0"
    - "@types/jest": "^29.0.0"
    - "@types/supertest": "^2.0.0"

test_configuration:
  unit_tests:
    - coverage_threshold: 80%
    - test_environment: "node"
    - setup_files_after_env: true
  integration_tests:
    - test_database: "separate_test_db"
    - transaction_rollback: true
    - parallel_execution: false
  performance_tests:
    - response_time_limits: "< 500ms"
    - concurrent_request_testing: true
```

## TESTING REQUIREMENTS

**Unit Tests**:

- Service layer business logic
- Controller request/response handling
- Utility function validation
- Error handling scenarios

**Integration Tests**:

- API endpoint functionality
- Database operations
- Authentication and authorization
- Health check endpoints

**End-to-End Tests**:

- Complete user workflows
- Authentication flows
- Error handling scenarios
- System integration points

## DOCUMENTATION REQUIREMENTS

- Testing strategy and guidelines
- Test setup and execution instructions
- Coverage reporting configuration
- CI/CD integration documentation

## VALIDATION CRITERIA

- [ ] Jest configuration works for unit and integration tests
- [ ] Test database setup and teardown works properly
- [ ] Unit tests achieve minimum 80% coverage
- [ ] Integration tests validate API endpoints correctly
- [ ] Test utilities and factories simplify test creation
- [ ] Test isolation prevents cross-test interference
- [ ] Performance tests validate response time requirements
- [ ] CI/CD test scripts are configured and functional
- [ ] Test documentation is complete and accurate
- [ ] All existing functionality has corresponding tests