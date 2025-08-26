# PROMPTS: TASK-006 - Backend Authentication System

# PROMPT 1: JWT Authentication Guard and User Context Injection

**Create a comprehensive JWT authentication guard that validates AWS Cognito tokens, extracts user information, and injects user context into all authenticated requests. The guard must handle token validation, error scenarios, and provide a foundation for authorization.**

## PROJECT CONTEXT

**Technology Stack:**
- Backend: TypeScript with NestJS framework
- Authentication: AWS Cognito JWT token validation  
- Architecture: Domain-driven design with guard pattern for access control
- Database: PostgreSQL with TypeORM for user entity management

**Code Standards:**
- Use NestJS decorators and guard patterns
- Implement proper error handling with appropriate HTTP status codes
- Follow dependency injection patterns
- Maintain clean separation between authentication and authorization logic

## EPIC CONTEXT

**Authentication Epic Requirements:**
- OAuth2 scope-based authorization system foundation
- Integration with AWS Cognito for user management
- Multi-tenant architecture with clinic boundaries
- Audit trail for authentication events
- Required by all business feature modules for access control

## TASK CONTEXT

**Primary Objective:** 
Implement JWT token validation and user context injection as the foundation for all authentication in the MedFlow backend system.

**Related Requirements:**
- JWT tokens validated on every protected request
- User context available in request objects throughout the application lifecycle
- Automatic handling of token expiration and refresh scenarios
- Integration with existing AWS Cognito service
- Support for public endpoints that don't require authentication

## BUSINESS DOMAIN CONTEXT

**Healthcare Authentication Requirements:**
- Healthcare provider roles require medical license validation through custom Cognito attributes
- Multi-language support (Arabic/French) for user profiles  
- HIPAA-compliant user management with audit logging
- Cross-clinic data access is prohibited unless explicitly authorized

**Role Definitions:**
- System Administrator: `admin:full` scope with complete system access
- Clinic Manager: `clinic:manage` plus operational scopes  
- Healthcare Provider: Clinical and prescription scopes
- Clinical Staff: Clinical scopes with limited patient write access
- Administrative Staff: Patient and appointment scopes with limited clinical read access

## FUNCTIONAL REQUIREMENTS

1. **JWT Token Validation**
   - Validate JWT signature against AWS Cognito public keys
   - Check token expiration and ensure token is not expired
   - Extract user payload including user_id, clinic_id, scopes, and role information
   - Handle malformed or invalid tokens with appropriate error responses

2. **User Context Construction**
   - Build complete user context object from Cognito token payload and database user record
   - Include user ID, Cognito ID, email, name, role, scopes, clinic association, and active status
   - Inject user context into request object for use by controllers and services
   - Support optional authentication for endpoints marked as public

3. **Public Endpoint Support**
   - Allow endpoints decorated with `@Public()` to bypass authentication requirements
   - Ensure public endpoints are accessible without tokens
   - Maintain consistent guard behavior across all endpoint types

## TECHNICAL REQUIREMENTS

1. **Guard Implementation**
   - Create `JwtAuthGuard` implementing NestJS `CanActivate` interface
   - Use dependency injection for Cognito service and user repository
   - Implement proper logging for authentication events and failures
   - Handle edge cases like missing Authorization header gracefully

2. **Token Extraction and Processing**
   - Extract Bearer token from Authorization header
   - Validate token format and structure before processing
   - Parse token payload and extract required user attributes
   - Map Cognito user data to internal user context structure

3. **Error Handling**
   - Return 401 Unauthorized for invalid or missing tokens
   - Return appropriate error messages without exposing sensitive information
   - Log authentication failures for security monitoring
   - Handle network issues with Cognito validation service

4. **Performance Requirements**
   - Authentication check must add < 50ms latency to requests
   - Implement efficient token validation without excessive Cognito API calls
   - Cache validation results where appropriate without compromising security

## TESTING REQUIREMENTS

**Unit Tests:**
- Test successful token validation with valid JWT
- Test failure scenarios with invalid, expired, and malformed tokens
- Test user context construction with various user types and roles
- Test public endpoint bypass functionality
- Test error handling and appropriate HTTP status codes

**Integration Tests:**
- Test end-to-end authentication flow with real AWS Cognito tokens
- Test integration with user service for context building
- Test request flow through authentication guard to protected endpoints
- Test performance under load with multiple concurrent requests

**Security Tests:**
- Test token tampering detection and rejection
- Test unauthorized access prevention
- Test information disclosure prevention in error messages
- Test authentication bypass attempts

## DOCUMENTATION REQUIREMENTS

- JSDoc comments for all guard methods explaining purpose and parameters
- Code comments explaining complex token validation logic
- Example usage documentation showing how to apply the guard to controllers
- Error handling documentation for different failure scenarios

## VALIDATION CRITERIA

- [ ] JwtAuthGuard successfully validates AWS Cognito JWT tokens
- [ ] User context is properly injected into authenticated requests
- [ ] Invalid tokens are rejected with appropriate 401 errors
- [ ] Public endpoints bypass authentication without issues
- [ ] Authentication adds minimal performance overhead (< 50ms)
- [ ] All edge cases are handled gracefully with proper error responses
- [ ] Unit tests achieve 95%+ code coverage for guard logic
- [ ] Integration tests verify end-to-end authentication flow
- [ ] Documentation includes complete usage examples and error scenarios

---

# PROMPT 2: OAuth2 Scope-Based Authorization Guard

**Implement a scope-based authorization guard that enforces healthcare-specific permissions using OAuth2 scopes. The guard must validate user permissions against required scopes, implement role hierarchy, and prevent unauthorized access to protected endpoints.**

## PROJECT CONTEXT

**Authentication Foundation:**
- JWT Authentication Guard from Prompt 1 provides user context injection
- User context includes scopes array extracted from Cognito tokens
- NestJS guard pattern for consistent permission checking across all endpoints

**Authorization Architecture:**
- Decorator pattern for specifying required scopes on endpoints
- Guard pattern for runtime permission validation  
- Exception filter pattern for consistent authorization error handling

## EPIC CONTEXT

**Authorization Requirements:**
- OAuth2 scope-based authorization system aligned with healthcare workflows
- Role hierarchy where higher-level roles inherit lower-level permissions
- Healthcare-specific permission scopes for clinical operations
- Granular access control for different types of healthcare data and operations

## TASK CONTEXT

**Primary Objective:**
Create a comprehensive authorization system that enforces permission-based access control using OAuth2 scopes aligned with healthcare workflow requirements.

**Dependencies:**
- Requires JWT Authentication Guard (Prompt 1) for user context
- Integrates with role definitions from AWS Cognito configuration
- Builds foundation for endpoint protection across all feature modules

## BUSINESS DOMAIN CONTEXT

**Healthcare Permission Scopes:**
- `admin:full` - Complete system administration access
- `clinic:manage` - Clinic operations management permissions  
- `patient:write` - Create and modify patient records
- `patient:read` - View patient information
- `appointment:write` - Schedule and modify appointments
- `appointment:read` - View appointment information
- `clinical:write` - Document clinical information (requires clinical training)
- `clinical:read` - Access clinical records
- `billing:write` - Process payments and billing
- `billing:read` - View financial information
- `prescription:write` - Create prescriptions (restricted to licensed prescribers)
- `prescription:read` - View prescription information

**Role Hierarchy Rules:**
- System Administrator (`admin:full`) inherits all lower-level scopes
- Clinic Manager (`clinic:manage`) inherits operational read scopes
- Higher-level roles automatically have permissions for lower-level operations
- Clinical scopes restricted to users with appropriate medical credentials

## FUNCTIONAL REQUIREMENTS

1. **Scope Validation**
   - Extract required scopes from endpoint metadata using Reflector
   - Compare required scopes against user's available scopes from JWT token
   - Implement scope hierarchy where higher roles inherit lower permissions
   - Allow endpoints without required scopes to be accessible to authenticated users

2. **Permission Hierarchy Implementation**
   - `admin:full` provides access to all endpoints regardless of specific scope requirements
   - `clinic:manage` provides read access to patient, appointment, and clinical data
   - Clinical scopes require specific role validation for healthcare-specific operations
   - Implement logical scope groupings for efficient permission checking

3. **Authorization Error Handling**
   - Return 403 Forbidden for insufficient permissions with clear error messages
   - Include required scopes information in error response for debugging
   - Log authorization failures for security monitoring and audit trails
   - Provide actionable error messages without exposing sensitive system information

## TECHNICAL REQUIREMENTS

1. **ScopesGuard Implementation**
   - Create `ScopesGuard` implementing NestJS `CanActivate` interface
   - Use Reflector to extract required scopes from endpoint metadata
   - Implement efficient scope checking algorithms avoiding complex nested loops
   - Handle missing user context gracefully with appropriate error responses

2. **RequireScopes Decorator**
   - Create `@RequireScopes(...scopes: string[])` decorator for endpoint protection
   - Use SetMetadata to store required scopes for guard access
   - Support multiple scopes per endpoint with AND logic (user must have all)
   - Allow flexible scope specification for different endpoint requirements

3. **Performance Optimization**
   - Implement efficient scope checking without database queries during authorization
   - Use Set operations for fast scope membership testing
   - Cache role hierarchy mappings to avoid repeated calculations
   - Minimize authorization overhead to maintain API response times

4. **Integration Requirements**
   - Work seamlessly with JWT Authentication Guard user context
   - Support combination with other guards (clinic isolation, rate limiting)
   - Integrate with audit logging system for authorization events
   - Maintain compatibility with existing controller patterns

## TESTING REQUIREMENTS

**Unit Tests:**
- Test scope validation with various user role and scope combinations
- Test role hierarchy enforcement with admin and clinic manager roles
- Test authorization failure scenarios with insufficient permissions
- Test missing scopes metadata handling (should allow access)
- Test edge cases like empty scopes arrays and undefined user context

**Integration Tests:**
- Test end-to-end authorization flow from JWT token to endpoint access
- Test authorization with different healthcare roles and real scope combinations
- Test combined authentication and authorization guard behavior
- Test authorization logging and audit trail generation

**Security Tests:**
- Test scope tampering prevention (scopes come from validated JWT)
- Test privilege escalation prevention across role boundaries
- Test unauthorized access prevention for clinical and prescription operations
- Test information disclosure prevention in authorization error messages

## DOCUMENTATION REQUIREMENTS

- JSDoc documentation for all guard methods and decorator functions
- Code comments explaining scope hierarchy and permission inheritance logic
- Usage examples showing how to protect different types of healthcare endpoints
- Security documentation explaining scope-based access control principles

## VALIDATION CRITERIA

- [ ] ScopesGuard correctly validates user permissions against required scopes
- [ ] Role hierarchy properly implemented with admin and clinic manager privileges
- [ ] Authorization failures return 403 Forbidden with clear error messages
- [ ] Required scopes decorator works correctly with various scope combinations
- [ ] Performance requirements met with minimal authorization overhead
- [ ] Healthcare-specific scope validation works for clinical and prescription operations
- [ ] Unit tests cover all permission scenarios and edge cases
- [ ] Integration tests verify complete authorization workflow
- [ ] Security testing confirms proper access control enforcement

---

# PROMPT 3: Multi-Tenant Clinic Isolation Guard

**Implement a clinic isolation guard that enforces multi-tenant data separation, ensuring users can only access data from their associated clinic. The guard must prevent cross-clinic data leakage while supporting system administrator override capabilities.**

## PROJECT CONTEXT

**Multi-Tenant Architecture:**
- Each user is associated with a specific clinic through clinic_id in user context
- Data access must be restricted to user's clinic unless explicitly authorized
- System administrators have override capabilities for cross-clinic management
- Database queries must include clinic_id filtering for data isolation

**Existing Authentication Context:**
- JWT Authentication Guard provides user context with clinic_id
- Scope-based authorization guard handles permission validation
- Clinic isolation guard adds additional layer of tenant-specific security

## EPIC CONTEXT

**Multi-Tenant Security Requirements:**
- Cross-clinic data access is prohibited unless explicitly authorized
- Healthcare data isolation ensures HIPAA compliance and patient privacy
- System administrators need controlled access to multiple clinics for management
- Audit trail required for any cross-clinic data access attempts

## TASK CONTEXT

**Primary Objective:**
Implement comprehensive clinic isolation to ensure multi-tenant data security and prevent unauthorized access to healthcare data across clinic boundaries.

**Integration Requirements:**
- Works with existing authentication and authorization guards
- Integrates with request parameter extraction for clinic_id validation
- Supports both API parameters and request body clinic_id references
- Maintains high performance for all data access operations

## BUSINESS DOMAIN CONTEXT

**Healthcare Multi-Tenancy Rules:**
- Patients belong to specific clinics and cannot be accessed by other clinics
- Provider schedules and appointments are clinic-specific
- Clinical documentation and prescriptions are isolated per clinic
- Billing information and financial data must remain clinic-isolated

**Access Control Exceptions:**
- System administrators (`admin:full` scope) can access all clinic data
- Specific cross-clinic permissions may be granted for certain operations
- Emergency access protocols may override standard isolation rules
- Audit logging required for all cross-clinic access scenarios

## FUNCTIONAL REQUIREMENTS

1. **Clinic ID Extraction and Validation**
   - Extract clinic_id from request parameters, body, and query parameters
   - Compare extracted clinic_id against user's associated clinic from JWT context
   - Handle requests without explicit clinic_id (default to user's clinic)
   - Support multiple clinic_id sources in complex requests

2. **Multi-Tenant Access Control**
   - Prevent access to data from clinics other than user's assigned clinic
   - Allow system administrators to access any clinic data with proper logging
   - Handle special cases for shared resources and system-wide operations
   - Implement graceful handling of invalid or non-existent clinic_id values

3. **Cross-Clinic Access Prevention**
   - Block unauthorized cross-clinic data access attempts immediately
   - Return 403 Forbidden with clear error message for clinic isolation violations
   - Log all cross-clinic access attempts for security monitoring
   - Prevent information disclosure about other clinics through error messages

## TECHNICAL REQUIREMENTS

1. **ClinicGuard Implementation**
   - Create `ClinicGuard` implementing NestJS `CanActivate` interface
   - Extract clinic_id from multiple request sources (params, body, query)
   - Implement efficient clinic validation without database queries
   - Handle missing or undefined clinic_id values appropriately

2. **Request Context Analysis**
   - Parse request parameters for clinic_id references
   - Check request body for nested clinic_id values in complex payloads
   - Extract clinic_id from query parameters for filtering operations
   - Handle array operations where multiple clinic_ids might be present

3. **System Administrator Override**
   - Detect `admin:full` scope for cross-clinic access permissions
   - Log all administrator cross-clinic access for audit purposes
   - Maintain security while providing necessary administrative flexibility
   - Implement configurable override policies for specific operations

4. **Performance Requirements**
   - Clinic validation must add minimal latency to API requests
   - Avoid database queries during clinic isolation checks
   - Use efficient string comparison and context validation methods
   - Support high-throughput operations with clinic filtering

## TESTING REQUIREMENTS

**Unit Tests:**
- Test clinic isolation enforcement for different user roles
- Test system administrator override functionality
- Test clinic_id extraction from various request sources
- Test error handling for missing or invalid clinic_id values
- Test edge cases with undefined user context or clinic associations

**Integration Tests:**
- Test complete multi-tenant workflow with real user contexts and requests
- Test cross-clinic access prevention in typical healthcare workflows
- Test system administrator cross-clinic operations with audit logging
- Test performance impact on high-volume clinic-specific operations

**Security Tests:**
- Test cross-clinic data leakage prevention
- Test unauthorized clinic access attempts and proper rejection
- Test information disclosure prevention in error responses
- Test clinic_id manipulation attempts and validation

## DOCUMENTATION REQUIREMENTS

- JSDoc comments explaining multi-tenant isolation principles
- Code documentation for clinic_id extraction and validation logic
- Security documentation outlining clinic isolation enforcement
- Usage examples showing proper clinic-aware endpoint implementation

## VALIDATION CRITERIA

- [ ] Clinic isolation guard prevents unauthorized cross-clinic data access
- [ ] System administrators can access multiple clinics with proper audit logging
- [ ] Clinic_id extraction works from request parameters, body, and query sources
- [ ] Cross-clinic access attempts return appropriate 403 Forbidden errors
- [ ] Performance requirements met with minimal latency impact
- [ ] Security testing confirms complete tenant isolation
- [ ] Unit tests cover all clinic validation scenarios and edge cases
- [ ] Integration tests verify multi-tenant workflow protection
- [ ] Documentation includes comprehensive multi-tenant security guidance

---

# PROMPT 4: Authentication Decorators and User Context Utilities

**Create a comprehensive set of NestJS decorators for authentication and user context management, including parameter decorators for user injection, metadata decorators for endpoint configuration, and utility functions for common authentication patterns.**

## PROJECT CONTEXT

**Decorator Architecture:**
- NestJS parameter decorators for injecting user context into controller methods
- Metadata decorators for configuring authentication and authorization requirements
- Consistent patterns for accessing user information across all controllers
- Integration with existing guard system for seamless authentication flow

**Code Reusability:**
- Standardized decorators reduce boilerplate code in controllers
- Type-safe user context access with TypeScript interfaces
- Consistent error handling patterns across all authenticated endpoints
- Simplified controller implementation with declarative authentication

## EPIC CONTEXT

**Developer Experience Goals:**
- Simple, intuitive API for controllers to access authenticated user information
- Declarative approach to endpoint security configuration
- Consistent patterns across all healthcare workflow implementations
- Reduced cognitive load for developers implementing new features

## TASK CONTEXT

**Primary Objective:**
Create a comprehensive decorator system that simplifies controller implementation while maintaining type safety and security for user context access.

**Integration Dependencies:**
- Requires JWT Authentication Guard for user context population
- Works with Scope-based Authorization Guard for permission validation
- Supports Clinic Isolation Guard for multi-tenant operations
- Provides foundation for all controller authentication patterns

## BUSINESS DOMAIN CONTEXT

**Healthcare Context Requirements:**
- Controllers need access to current user's role, clinic, and permissions
- Healthcare workflows require user information for audit trails
- Provider-specific operations need access to user's medical credentials
- Patient operations require clinic context for data isolation

**User Context Properties:**
- Basic user information: id, email, firstName, lastName
- Authentication context: cognitoId, isActive status
- Authorization context: role, scopes array
- Healthcare context: clinicId for multi-tenant operations

## FUNCTIONAL REQUIREMENTS

1. **User Context Injection Decorators**
   - `@CurrentUser()` decorator to inject complete user context
   - `@CurrentUser('property')` decorator for specific user properties
   - `@UserClinic()` decorator to inject user's clinic context
   - Type-safe parameter injection with proper TypeScript interfaces

2. **Authentication Configuration Decorators**
   - `@Public()` decorator to mark endpoints as publicly accessible
   - `@RequireScopes(...scopes)` decorator for permission-based access control
   - `@OptionalAuth()` decorator for endpoints with conditional authentication
   - Clean metadata-based configuration without complex guard setup

3. **Utility Functions**
   - Helper functions for common user context operations
   - Type guards for checking user roles and permissions
   - Context validation utilities for edge case handling
   - User context transformation utilities for different use cases

## TECHNICAL REQUIREMENTS

1. **Parameter Decorator Implementation**
   - Create `CurrentUser` parameter decorator using `createParamDecorator`
   - Support both full user context and specific property injection
   - Implement proper TypeScript typing for parameter injection
   - Handle missing user context gracefully with appropriate error messages

2. **Metadata Decorator Implementation**
   - Create `Public` decorator using `SetMetadata` for guard configuration
   - Create `RequireScopes` decorator for authorization metadata
   - Ensure consistent metadata key naming across all decorators
   - Support decorator composition for complex endpoint requirements

3. **Type Safety and Interfaces**
   - Define comprehensive `UserContext` interface with all user properties
   - Create utility types for different user context scenarios
   - Implement proper generic typing for property-specific injection
   - Ensure compile-time type checking for user context access

4. **Error Handling Integration**
   - Graceful handling of missing user context in decorators
   - Appropriate error messages for development and debugging
   - Integration with existing exception handling patterns
   - Runtime validation of user context completeness

## TESTING REQUIREMENTS

**Unit Tests:**
- Test CurrentUser decorator with complete user context injection
- Test property-specific user context injection with various properties
- Test Public decorator metadata setting and guard integration
- Test RequireScopes decorator with different scope combinations
- Test edge cases with undefined user context and missing properties

**Integration Tests:**
- Test decorators integration with authentication guards and real requests
- Test type safety and compile-time error detection for incorrect usage
- Test decorator behavior in complex controller scenarios
- Test performance impact of decorator overhead in high-volume scenarios

**Developer Experience Tests:**
- Test IntelliSense and TypeScript completion with decorator usage
- Test error messages clarity for common developer mistakes
- Test documentation accuracy and completeness for decorator usage

## DOCUMENTATION REQUIREMENTS

- Comprehensive JSDoc for all decorators with usage examples
- TypeScript interface documentation for UserContext and related types
- Code examples showing common authentication patterns with decorators
- Best practices guide for using decorators in healthcare controllers
- Migration guide from manual user context handling to decorator patterns

## VALIDATION CRITERIA

- [ ] CurrentUser decorator successfully injects complete user context
- [ ] Property-specific user injection works with type safety
- [ ] Public decorator properly configures endpoints for public access
- [ ] RequireScopes decorator integrates with authorization guard
- [ ] All decorators maintain type safety with proper TypeScript interfaces
- [ ] Error handling provides clear messages for development scenarios
- [ ] Performance impact is minimal for decorator overhead
- [ ] Unit tests achieve complete coverage for all decorator functionality
- [ ] Integration tests verify decorator behavior in realistic scenarios
- [ ] Documentation provides clear usage guidance and examples

---

# PROMPT 5: Audit Logging and Security Event Tracking

**Implement comprehensive audit logging for all authentication and authorization events, including successful logins, authorization failures, cross-clinic access attempts, and security violations. The system must provide detailed tracking for healthcare compliance and security monitoring.**

## PROJECT CONTEXT

**Security and Compliance Context:**
- Healthcare applications require comprehensive audit trails for HIPAA compliance
- Authentication and authorization events must be logged for security monitoring
- Audit logs provide forensic information for security incident investigation
- Logging system must handle high-volume operations without performance degradation

**Integration with Existing Components:**
- Audit logging integrates with JWT Authentication Guard
- Authorization events tracked through Scope-based Authorization Guard
- Multi-tenant events logged through Clinic Isolation Guard
- Centralized logging approach for consistency across all authentication components

## EPIC CONTEXT

**Healthcare Compliance Requirements:**
- HIPAA audit requirements for access to protected health information
- Security event logging for unauthorized access attempts
- User activity tracking for clinical decision accountability
- Regulatory compliance reporting through comprehensive audit trails

**Operational Security:**
- Real-time security monitoring and alerting capabilities
- Forensic analysis support for security incident response
- User behavior analytics for anomaly detection
- Integration with security information and event management (SIEM) systems

## TASK CONTEXT

**Primary Objective:**
Implement a robust audit logging system that captures all security-relevant events while maintaining high performance and providing actionable security intelligence.

**Integration Requirements:**
- Seamless integration with existing authentication and authorization guards
- Structured logging format compatible with log aggregation systems
- Configurable logging levels for different environments
- Async logging to prevent performance impact on API operations

## BUSINESS DOMAIN CONTEXT

**Healthcare Audit Requirements:**
- Track all access to patient information with user and timestamp details
- Log clinical decision points and user actions for medical liability
- Monitor prescription access and controlled substance operations
- Record multi-clinic access patterns and potential policy violations

**Security Event Categories:**
- Authentication events: login attempts, token validation, session management
- Authorization events: permission checks, scope validation, role enforcement
- Data access events: patient record access, clinical data viewing
- Security violations: unauthorized access attempts, policy violations

## FUNCTIONAL REQUIREMENTS

1. **Authentication Event Logging**
   - Log successful authentication with user, timestamp, and source information
   - Record failed authentication attempts with failure reasons and source IPs
   - Track token refresh operations and session management events
   - Monitor unusual authentication patterns and potential security threats

2. **Authorization Event Logging**
   - Log all authorization decisions with user, requested resource, and outcome
   - Record permission failures with required vs. available scopes
   - Track role-based access decisions and hierarchy evaluations
   - Monitor privilege escalation attempts and unusual permission patterns

3. **Security Violation Tracking**
   - Log cross-clinic access attempts with user and target clinic information
   - Record data access policy violations and unauthorized operation attempts
   - Track suspicious user behavior patterns and anomaly detection triggers
   - Monitor administrative override usage and emergency access scenarios

4. **Structured Audit Trail**
   - Consistent log format with standard fields across all security events
   - Searchable and filterable audit records for compliance reporting
   - Integration with centralized logging infrastructure and SIEM systems
   - Long-term audit record retention for regulatory compliance requirements

## TECHNICAL REQUIREMENTS

1. **Audit Logger Service**
   - Create dedicated `AuditLoggerService` for security event logging
   - Implement structured logging with consistent field naming and formats
   - Support multiple log destinations (console, file, remote logging services)
   - Async logging implementation to prevent API performance impact

2. **Event Categories and Schemas**
   - Define comprehensive event schemas for different security event types
   - Implement event severity levels (INFO, WARNING, ERROR, CRITICAL)
   - Create event correlation IDs for tracking related security events
   - Support custom event metadata for healthcare-specific audit requirements

3. **Integration with Guards**
   - Modify JWT Authentication Guard to log authentication events
   - Update Scope-based Authorization Guard for permission event logging
   - Enhance Clinic Isolation Guard with multi-tenant access logging
   - Implement middleware for automatic request/response audit logging

4. **Performance and Scalability**
   - Implement batched logging for high-volume environments
   - Use async logging patterns to prevent blocking API operations
   - Configure log level filtering based on environment requirements
   - Support log rotation and archival for long-term storage management

## TESTING REQUIREMENTS

**Unit Tests:**
- Test audit logger service with different event types and severity levels
- Test event schema validation and structured logging format consistency
- Test async logging behavior and error handling scenarios
- Test integration with authentication and authorization guards

**Integration Tests:**
- Test complete audit trail generation through authentication workflows
- Test log aggregation and filtering for security event analysis
- Test performance impact of audit logging on API response times
- Test audit log retention and archival functionality

**Security Tests:**
- Test audit log integrity and tamper prevention
- Test security event detection accuracy and completeness
- Test log access controls and audit administrator permissions
- Test compliance reporting and audit trail completeness

## DOCUMENTATION REQUIREMENTS

- Comprehensive documentation of audit event schemas and field definitions
- Security monitoring playbook for analyzing audit logs and detecting threats
- Compliance reporting guide for generating HIPAA and regulatory audit reports
- Integration documentation for connecting audit logs to SIEM and monitoring systems

## VALIDATION CRITERIA

- [ ] All authentication events properly logged with complete context information
- [ ] Authorization decisions tracked with user, resource, and outcome details
- [ ] Security violations detected and logged with appropriate severity levels
- [ ] Audit logging adds minimal performance overhead to API operations
- [ ] Structured log format supports efficient searching and analysis
- [ ] Integration with authentication guards captures all security-relevant events
- [ ] Long-term audit retention supports regulatory compliance requirements
- [ ] Unit tests verify audit logging functionality and event schema validation
- [ ] Performance testing confirms minimal impact on API response times
- [ ] Security testing validates audit log integrity and access controls

---

# PROMPT 6: Authentication System Integration and Global Configuration

**Integrate all authentication components (guards, decorators, audit logging) into a cohesive system with global configuration, application-level setup, and comprehensive error handling. Establish the complete authentication infrastructure for the MedFlow healthcare application.**

## PROJECT CONTEXT

**System Integration Context:**
- Combine all authentication components into unified NestJS module structure
- Configure global guards and interceptors for application-wide authentication
- Establish consistent error handling patterns across all authentication scenarios
- Integrate with existing NestJS application structure and AWS infrastructure

**Configuration Management:**
- Environment-specific authentication configuration (development, staging, production)
- AWS Cognito connection configuration with proper credential management
- Logging configuration for different deployment environments
- Security configuration for production healthcare deployments

## EPIC CONTEXT

**Complete Authentication Infrastructure:**
- Foundation for all healthcare workflow authentication and authorization
- Integration point for frontend authentication and protected routes
- Security infrastructure supporting HIPAA compliance and audit requirements
- Scalable authentication system supporting multi-clinic healthcare operations

## TASK CONTEXT

**Primary Objective:**
Create a complete, production-ready authentication system that integrates all components and provides a solid foundation for all MedFlow healthcare applications.

**Final Integration:**
- All authentication components from previous prompts work together seamlessly
- Global application configuration enables authentication across all controllers
- Comprehensive error handling provides consistent security behavior
- Performance optimization ensures minimal impact on healthcare workflow operations

## BUSINESS DOMAIN CONTEXT

**Healthcare Production Requirements:**
- 24/7 availability requirements for healthcare applications
- High-performance authentication for busy clinical environments
- Robust error handling that doesn't disrupt patient care workflows
- Comprehensive security monitoring for healthcare compliance

**Operational Integration:**
- Easy deployment and configuration for different clinic environments
- Clear monitoring and troubleshooting capabilities for healthcare IT teams
- Integration with existing clinic infrastructure and security policies
- Support for emergency access and disaster recovery scenarios

## FUNCTIONAL REQUIREMENTS

1. **Authentication Module Integration**
   - Create comprehensive `AuthModule` that exports all authentication services and guards
   - Configure global guards at application level for automatic protection
   - Establish authentication module dependencies and provider configuration
   - Integrate with main application module structure

2. **Global Configuration Management**
   - Environment-based configuration for AWS Cognito connection parameters
   - Authentication behavior configuration (token expiration, refresh policies)
   - Audit logging configuration for different deployment environments  
   - Error handling configuration and customization options

3. **Application-Level Integration**
   - Configure global authentication guards for automatic endpoint protection
   - Set up global exception filters for consistent authentication error handling
   - Integrate authentication interceptors for request/response processing
   - Configure application-wide authentication middleware

4. **Production Readiness**
   - Health check endpoints for authentication service monitoring
   - Performance monitoring and metrics collection for authentication operations
   - Error recovery and resilience patterns for authentication failures
   - Documentation and deployment guides for production healthcare environments

## TECHNICAL REQUIREMENTS

1. **AuthModule Implementation**
   - Create comprehensive NestJS module with all authentication providers
   - Configure module imports for AWS services and database connections
   - Export guards, decorators, and services for use across application
   - Implement proper dependency injection configuration

2. **Global Guards Configuration**
   - Configure `APP_GUARD` providers for automatic authentication protection
   - Set up guard precedence and execution order for multiple authentication layers
   - Implement global exception filters for authentication error handling
   - Configure interceptors for audit logging and request processing

3. **Configuration Service Integration**
   - Create `AuthConfig` service for environment-specific authentication settings
   - Integrate with NestJS ConfigModule for secure configuration management
   - Implement configuration validation for required authentication parameters
   - Support runtime configuration updates for operational flexibility

4. **Error Handling and Resilience**
   - Create global exception filter for authentication and authorization errors
   - Implement retry policies for transient authentication service failures
   - Configure circuit breaker patterns for AWS Cognito service integration
   - Establish error recovery procedures for authentication system failures

5. **Performance Optimization**
   - Implement caching strategies for token validation and user context
   - Configure connection pooling for AWS Cognito service requests
   - Optimize guard execution order to minimize authentication overhead
   - Implement performance monitoring and alerting for authentication operations

## TESTING REQUIREMENTS

**Integration Tests:**
- Test complete authentication system integration with real NestJS application
- Test authentication flow through multiple guards and interceptors
- Test global configuration in different environment scenarios
- Test error handling and recovery across all authentication components

**Performance Tests:**
- Load testing to ensure authentication system handles high-volume clinic operations
- Stress testing for peak usage scenarios in busy healthcare environments
- Memory and CPU usage profiling for authentication system components
- Latency testing to ensure minimal impact on healthcare workflow operations

**End-to-End Tests:**
- Complete user authentication journey from login to protected endpoint access
- Multi-user scenarios with different roles and clinic associations
- Error scenarios and recovery testing for production resilience
- Security testing for complete system attack surface analysis

## DOCUMENTATION REQUIREMENTS

- Complete authentication system architecture documentation
- Deployment and configuration guide for healthcare IT teams
- Troubleshooting guide for common authentication issues
- Security configuration best practices for healthcare compliance
- API documentation for authentication endpoints and error responses

## VALIDATION CRITERIA

- [ ] Complete authentication system properly integrated with NestJS application
- [ ] All authentication components work together without conflicts
- [ ] Global guards provide automatic protection for all application endpoints
- [ ] Configuration system supports different deployment environments
- [ ] Error handling provides consistent behavior across all authentication scenarios
- [ ] Performance requirements met for healthcare workflow operations
- [ ] Security testing confirms complete system protection
- [ ] Integration tests verify end-to-end authentication functionality
- [ ] Documentation provides complete deployment and operational guidance
- [ ] Production deployment successful with proper monitoring and alerting