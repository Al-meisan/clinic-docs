# PROMPTS: TASK-020 - Integration Testing Framework

# PROMPT 1: E2E Testing Framework Foundation Setup

Implement the foundational end-to-end testing framework using Playwright with TypeScript, including base page objects, test utilities, and essential test infrastructure for the clinic management system.

## PROJECT CONTEXT

**Tech Stack:**
- Frontend: React with TypeScript and shadcn/ui components
- Backend: NestJS with TypeScript
- Database: PostgreSQL with TypeORM
- Authentication: AWS Cognito integration
- Build System: Vite for frontend, standard NestJS for backend

**Project Structure:**
- Feature-based folder structure for frontend (`src/features/`)
- Domain-driven design for backend
- Standardized URL patterns and navigation structure
- Role-based access control with defined scopes

## EPIC CONTEXT

This is part of the Project Setup Epic focusing on establishing development infrastructure and quality assurance capabilities for the healthcare clinic management system.

## TASK CONTEXT

**Task Goal:** Set up comprehensive end-to-end testing framework with proper test data management, environment isolation, and automated testing capabilities.

**Current Dependencies Available:**
- Backend API Implementation (TASK-003)
- Frontend Application (TASK-007) 
- Database Infrastructure (TASK-004)
- CI/CD Pipeline (TASK-015)

## BUSINESS DOMAIN CONTEXT

**Healthcare Domain Requirements:**
- Patient data privacy and security maintained in test environments
- Healthcare workflow testing covers compliance requirements
- Critical patient care processes thoroughly tested
- No real patient data in test environments

**Key Workflows to Support:**
- Patient registration and management
- Provider-patient relationship testing
- Appointment scheduling and clinical documentation flows
- Billing and payment processing

## FUNCTIONAL REQUIREMENTS

1. **Playwright Framework Setup**
   - Configure Playwright with TypeScript support
   - Set up cross-browser testing (Chrome, Firefox, Safari, Edge)
   - Configure mobile device testing capabilities
   - Implement screenshot and video recording on failures

2. **Base Page Object Architecture**
   - Create BasePage class with common navigation and utility methods
   - Implement authentication helpers (login/logout)
   - Add waiting utilities and loading state handling
   - Create standardized selectors using data-testid attributes

3. **Test Organization Structure**
   - Organize tests by feature modules (patients, appointments, clinical, billing)
   - Implement test categorization (smoke, regression, critical path)
   - Set up test configuration for different environments

## TECHNICAL REQUIREMENTS

**Performance:**
- Test execution setup time < 2 minutes
- Individual test execution < 30 seconds
- Parallel test execution capability

**Security:**
- Secure test user credentials management
- Test environment isolation from production
- Automated cleanup of sensitive test data

**Compatibility:**
- Support for headless and headed browser modes
- CI/CD pipeline integration ready
- Local development environment support

## TESTING REQUIREMENTS

**Unit Tests:**
- Test utility functions and helper methods
- Page object method validation
- Configuration validation

**Integration Tests:**
- Browser automation functionality
- Authentication flow testing
- Cross-page navigation testing

## DOCUMENTATION REQUIREMENTS

- Playwright setup and configuration guide
- Page object pattern documentation
- Test writing guidelines and best practices
- Troubleshooting guide for common issues

## VALIDATION CRITERIA

- [ ] Playwright configured with TypeScript and cross-browser support
- [ ] BasePage class implemented with authentication and navigation utilities
- [ ] Page object pattern established for main application areas
- [ ] Test structure organized by feature modules
- [ ] Sample smoke tests passing for critical workflows
- [ ] Configuration supports local development and CI/CD environments
- [ ] Documentation covers setup and test writing guidelines

---

# PROMPT 2: Test Data Management System Implementation

Implement comprehensive test data management system using factory pattern for entity generation, database seeding capabilities, and automated cleanup utilities to ensure consistent and isolated testing environments.

## PROJECT CONTEXT

**Database Schema:**
- Patient entities with personal information, medical history, medications
- Provider entities with schedules and specializations
- Appointment entities with status workflow and documentation
- Billing entities with payment processing and insurance

**ORM Configuration:**
- TypeORM with PostgreSQL
- Entity relationships and foreign key constraints
- Migration system for schema changes
- Soft delete implementation for data integrity

## EPIC CONTEXT

Building robust testing infrastructure that supports reliable, repeatable test execution across development, staging, and CI/CD environments while maintaining data privacy and security standards.

## TASK CONTEXT

**Implementation Focus:** Test data generation, database seeding, cleanup automation, and test environment isolation to support comprehensive end-to-end testing scenarios.

**Integration Dependencies:**
- Database infrastructure with test database configuration
- TypeORM entity definitions and relationships
- Authentication system for test user management

## BUSINESS DOMAIN CONTEXT

**Healthcare Data Requirements:**
- HIPAA-compliant test data handling (no real patient information)
- Realistic medical scenarios for workflow testing
- Insurance and billing data patterns
- Provider schedule and availability patterns

**Workflow Data Needs:**
- Patient registration with family relationships
- Appointment scheduling across multiple providers
- Clinical documentation with medical terminology
- Prescription and medication management data

## FUNCTIONAL REQUIREMENTS

1. **Factory Pattern Implementation**
   - PatientFactory with realistic demographic data generation
   - ProviderFactory with medical specializations and schedules
   - AppointmentFactory with various appointment types and statuses
   - BillingFactory with insurance and payment scenarios

2. **Database Seeding System**
   - Automated test database setup and initialization
   - Seed data for baseline scenarios (providers, insurance types, etc.)
   - Relationship management between entities
   - Configurable data volume for different test scenarios

3. **Data Cleanup Automation**
   - Per-test data isolation and cleanup
   - Soft delete handling and cascade operations
   - Database state restoration utilities
   - Test data lifecycle management

4. **Test Data Utilities**
   - Helper functions for common data operations
   - Data validation and constraint handling
   - Test-specific data queries and assertions

## TECHNICAL REQUIREMENTS

**Performance:**
- Database setup time < 30 seconds
- Test data generation < 5 seconds per entity
- Cleanup operations complete within 10 seconds

**Data Quality:**
- Realistic data patterns using Faker.js library
- Valid medical terminology and codes (ICD-10)
- Consistent data relationships and foreign key integrity
- Configurable data variations for edge case testing

**Environment Management:**
- Separate test database configuration
- Environment-specific connection strings
- Automated schema migration for test databases

## TESTING REQUIREMENTS

**Unit Tests:**
- Factory method validation and data integrity
- Database seeding and cleanup functionality
- Data generation utility functions
- Constraint validation and error handling

**Integration Tests:**
- End-to-end data lifecycle (create, use, cleanup)
- Cross-entity relationship validation
- Database transaction handling
- Concurrent test execution data isolation

## DOCUMENTATION REQUIREMENTS

- Factory pattern usage guide and examples
- Test data management best practices
- Database setup and configuration instructions
- Troubleshooting guide for data-related test failures

## VALIDATION CRITERIA

- [ ] Factory classes implemented for all major entities with realistic data generation
- [ ] Database seeding system operational with baseline test data
- [ ] Automated cleanup system ensures test isolation
- [ ] Test utilities provide easy data manipulation and validation
- [ ] Performance targets met for data operations
- [ ] Integration with existing TypeORM configuration
- [ ] Documentation covers all test data management scenarios

---

# PROMPT 3: API Integration Testing Suite Implementation

Implement comprehensive API integration testing using Supertest with NestJS, covering all endpoints, authentication flows, error scenarios, and data validation to ensure robust backend service reliability.

## PROJECT CONTEXT

**Backend Architecture:**
- NestJS application with modular feature structure
- RESTful API endpoints following standardized URL patterns
- JWT authentication with AWS Cognito integration
- Role-based access control with OAuth2 scopes
- Global exception filters and validation pipes

**API Coverage Required:**
- Patient management endpoints (CRUD operations)
- Provider management and scheduling endpoints
- Appointment lifecycle endpoints
- Clinical documentation endpoints
- Billing and payment processing endpoints

## EPIC CONTEXT

Ensuring comprehensive backend service quality through automated API testing that validates business logic, data integrity, error handling, and security controls across all service endpoints.

## TASK CONTEXT

**Implementation Focus:** Complete API endpoint testing with Supertest integration, authentication flow validation, error scenario coverage, and performance benchmarking for critical healthcare workflows.

**Dependencies:**
- NestJS backend application with all service modules
- Authentication system with test user credentials
- Database with test data management system
- Exception handling and validation systems

## BUSINESS DOMAIN CONTEXT

**Healthcare API Requirements:**
- Patient data access controls and privacy validation
- Clinical workflow state transitions and business rules
- Prescription management with safety checks and alerts
- Billing workflow integrity and payment processing validation

**Security and Compliance:**
- Authentication and authorization enforcement
- Role-based access control validation
- Data validation and sanitization testing
- Audit trail and logging verification

## FUNCTIONAL REQUIREMENTS

1. **Authentication Testing Framework**
   - JWT token generation and validation
   - Role-based access control verification
   - Session management and token refresh testing
   - Unauthorized access attempt handling

2. **CRUD Operation Testing**
   - Create operations with validation and business rules
   - Read operations with filtering, pagination, and sorting
   - Update operations with partial updates and versioning
   - Delete operations with soft delete and cascade handling

3. **Business Logic Validation**
   - Appointment scheduling conflict detection
   - Patient-provider relationship enforcement
   - Clinical documentation workflow state transitions
   - Billing calculation and payment processing logic

4. **Error Scenario Coverage**
   - Validation error responses and formatting
   - Database constraint violation handling
   - External service integration error scenarios
   - Rate limiting and security violation responses

## TECHNICAL REQUIREMENTS

**Test Organization:**
- Test suites organized by feature modules
- Setup and teardown hooks for data isolation
- Reusable test utilities and authentication helpers
- Configuration for different environments

**Performance Benchmarking:**
- Response time validation (< 500ms for standard operations)
- Concurrent request handling verification
- Database query optimization validation
- Memory usage monitoring during test execution

**API Documentation Validation:**
- OpenAPI/Swagger specification compliance
- Request/response schema validation
- HTTP status code correctness
- Error response format consistency

## TESTING REQUIREMENTS

**Integration Test Categories:**
- Authentication and authorization flows
- Complete CRUD operations for each entity
- Business workflow end-to-end scenarios
- Error handling and edge cases

**Performance Tests:**
- Endpoint response time validation
- Concurrent user simulation
- Database performance under load
- Memory leak detection

**Security Tests:**
- SQL injection protection validation
- XSS prevention verification
- Authentication bypass attempt testing
- Data access control enforcement

## DOCUMENTATION REQUIREMENTS

- API testing setup and configuration guide
- Test writing patterns and best practices
- Authentication testing documentation
- Error scenario testing guidelines

## VALIDATION CRITERIA

- [ ] All API endpoints covered with comprehensive test scenarios
- [ ] Authentication and authorization flows validated
- [ ] CRUD operations tested with business rule enforcement
- [ ] Error scenarios covered with proper response validation
- [ ] Performance benchmarks established and validated
- [ ] Security controls tested and verified
- [ ] Test suites integrated with CI/CD pipeline
- [ ] Documentation provides clear testing guidelines

---

# PROMPT 4: Multi-Environment Testing Configuration System

Implement multi-environment testing configuration supporting development, staging, and production-like environments with environment-specific settings, CI/CD integration, and automated environment provisioning.

## PROJECT CONTEXT

**Infrastructure Requirements:**
- Docker containerization for consistent environments
- AWS services integration (Cognito, RDS, S3)
- Environment-specific configuration management
- CI/CD pipeline with GitHub Actions
- Terraform infrastructure as code

**Application Configuration:**
- Environment-specific API endpoints and database connections
- AWS service configurations per environment
- Authentication and authorization settings
- Logging and monitoring configurations

## EPIC CONTEXT

Establishing reliable testing infrastructure that ensures consistent test execution across different environments while supporting continuous integration and deployment workflows for the healthcare application.

## TASK CONTEXT

**Implementation Focus:** Environment-specific test configuration, Docker containerization for test environments, CI/CD integration, and automated provisioning to support reliable testing across development lifecycle.

**Integration Points:**
- CI/CD pipeline for automated test execution
- Docker containers for environment isolation
- Environment configuration management
- Test reporting and monitoring systems

## BUSINESS DOMAIN CONTEXT

**Healthcare Environment Requirements:**
- HIPAA compliance across all testing environments
- Data isolation between environments
- Audit trail preservation for compliance testing
- Performance validation matching production requirements

**Operational Considerations:**
- Quick environment setup for development teams
- Consistent behavior across environments
- Rollback capabilities for failed deployments
- Environment monitoring and health checks

## FUNCTIONAL REQUIREMENTS

1. **Environment Configuration Management**
   - Development environment with hot reloading and debugging
   - Staging environment matching production configuration
   - Production-like testing environment for final validation
   - Local testing environment for individual developers

2. **Docker Container Configuration**
   - Multi-container setup (app, database, redis, etc.)
   - Environment-specific Docker Compose configurations
   - Volume management for persistent data and logs
   - Network configuration for service communication

3. **CI/CD Pipeline Integration**
   - Automated test execution on code changes
   - Environment provisioning and teardown
   - Test result reporting and notification
   - Deployment pipeline integration with test gates

4. **Configuration Management**
   - Environment variable management per environment
   - Secret management for credentials and API keys
   - Configuration validation and health checks
   - Dynamic configuration updates without downtime

## TECHNICAL REQUIREMENTS

**Environment Provisioning:**
- Container startup time < 2 minutes
- Environment teardown and cleanup < 1 minute
- Automated dependency management and version control
- Resource cleanup to prevent environment drift

**Configuration Validation:**
- Environment health checks and readiness probes
- Configuration drift detection and alerting
- Service availability monitoring
- Database connectivity and migration validation

**CI/CD Integration:**
- GitHub Actions workflow configuration
- Test execution parallelization
- Artifact management and storage
- Environment-specific deployment strategies

## TESTING REQUIREMENTS

**Environment Testing:**
- Configuration validation across all environments
- Service connectivity and integration testing
- Performance baseline establishment per environment
- Security configuration validation

**CI/CD Testing:**
- Pipeline execution validation
- Test result collection and reporting
- Failure notification and rollback testing
- Resource cleanup verification

## DOCUMENTATION REQUIREMENTS

- Environment setup and configuration guide
- Docker containerization documentation
- CI/CD pipeline configuration and usage
- Troubleshooting guide for environment issues

## VALIDATION CRITERIA

- [ ] Multi-environment configuration system operational
- [ ] Docker containers configured for all environments
- [ ] CI/CD pipeline integrated with automated test execution
- [ ] Environment provisioning and cleanup automated
- [ ] Configuration management system handles secrets securely
- [ ] Performance baselines established per environment
- [ ] Health checks and monitoring configured
- [ ] Documentation covers all environment management scenarios

---

# PROMPT 5: Performance Testing Framework and Load Testing Suite

Implement comprehensive performance testing framework using automated load testing tools, response time monitoring, and system performance benchmarking to validate clinic management system performance under expected user loads.

## PROJECT CONTEXT

**System Architecture:**
- NestJS backend with TypeORM and PostgreSQL database
- React frontend with client-side routing and state management
- AWS infrastructure with managed services
- Multi-user concurrent access patterns
- Real-time features for appointment scheduling and clinical workflows

**Performance Requirements:**
- Support for 100+ concurrent users
- Sub-500ms response times for critical operations
- Database query optimization for large datasets
- Frontend responsiveness under load

## EPIC CONTEXT

Establishing performance quality gates and monitoring to ensure the healthcare system maintains acceptable response times and stability under realistic user loads while supporting critical patient care operations.

## TASK CONTEXT

**Implementation Focus:** Automated performance testing suite, load testing scenarios, performance monitoring, and benchmarking system to validate system performance against defined requirements.

**Performance Targets:**
- API response times < 500ms for standard operations
- Database query performance < 100ms for simple queries
- Frontend page load times < 2 seconds
- System stability under 100+ concurrent users

## BUSINESS DOMAIN CONTEXT

**Healthcare Performance Requirements:**
- Critical patient care operations must maintain high availability
- Appointment scheduling system must handle peak booking times
- Clinical documentation must support real-time collaboration
- Emergency access must maintain performance under load

**User Load Patterns:**
- Morning appointment booking surge
- Clinical documentation concurrent usage
- End-of-day billing and administrative processing
- Emergency system access requirements

## FUNCTIONAL REQUIREMENTS

1. **Load Testing Framework**
   - Automated load testing using k6 or Artillery
   - Configurable user load profiles and scenarios
   - Gradual ramp-up and sustained load testing
   - Peak load and stress testing capabilities

2. **Performance Monitoring**
   - Real-time performance metrics collection
   - Response time percentile tracking (95th, 99th)
   - Error rate monitoring during load tests
   - Resource utilization tracking (CPU, memory, database)

3. **Test Scenario Design**
   - User registration and authentication flow
   - Patient management concurrent operations
   - Appointment scheduling under peak load
   - Clinical documentation and prescription workflows

4. **Performance Reporting**
   - Automated performance report generation
   - Historical performance trend analysis
   - Performance regression detection
   - CI/CD integration with performance gates

## TECHNICAL REQUIREMENTS

**Load Testing Configuration:**
- Configurable user load patterns (gradual ramp, spike testing)
- Test data generation for realistic scenarios
- Geographic load distribution simulation
- Mobile and desktop user simulation

**Performance Metrics:**
- Response time measurements across all endpoints
- Database query performance analysis
- Frontend rendering and interaction timing
- Memory usage and garbage collection monitoring

**Integration Requirements:**
- CI/CD pipeline integration with performance gates
- Performance monitoring integration with CloudWatch
- Test result storage and historical analysis
- Alert configuration for performance degradation

## TESTING REQUIREMENTS

**Performance Test Categories:**
- Load testing (expected user volumes)
- Stress testing (peak capacity identification)
- Spike testing (sudden load increase handling)
- Endurance testing (sustained load stability)

**Monitoring Validation:**
- Performance metrics accuracy verification
- Alert threshold validation and tuning
- Monitoring system reliability under load
- Data collection and reporting accuracy

## DOCUMENTATION REQUIREMENTS

- Performance testing setup and execution guide
- Load testing scenario documentation
- Performance monitoring and alerting configuration
- Performance optimization recommendations

## VALIDATION CRITERIA

- [ ] Load testing framework operational with configurable scenarios
- [ ] Performance monitoring system collecting comprehensive metrics
- [ ] Test scenarios cover all critical user workflows
- [ ] Performance benchmarks established and documented
- [ ] CI/CD integration with performance quality gates
- [ ] Automated reporting and alert system functional
- [ ] Performance optimization recommendations documented
- [ ] Framework supports different load patterns and stress scenarios