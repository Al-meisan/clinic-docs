# PROMPTS: TASK-005 - AWS Cognito Integration

# ~~PROMPT 1: AWS Cognito User Pool Configuration and Service Setup~~

**Create AWS Cognito service infrastructure for healthcare user management, including user pool configuration with custom healthcare attributes, authentication methods, and the foundational NestJS service for Cognito integration.**

## PROJECT CONTEXT
**Tech Stack Requirements:**
- Backend: TypeScript with NestJS framework
- Infrastructure: AWS services with cost-effective approach
- Authentication: AWS Cognito with JWT tokens
- Architecture: Domain-driven design with feature-based modules

**Coding Standards:**
- Folder/file names: lowercase with dashes (e.g., `aws-cognito`)
- Class names: PascalCase (e.g., `CognitoService`)
- File suffixes: `*.service.ts`, `*.module.ts`, `*.config.ts`
- Location: `src/infrastructure/aws/cognito/`

**Error Handling:**
- Use NestJS exception filters for normalized errors
- Log critical operations without exposing sensitive data
- Provide clear error messages for authentication failures

## EPIC CONTEXT
**Project Setup Epic Requirements:**
- Foundation for all user management and access control
- Required for frontend authentication and protected routes
- Enables role-based feature access across all application areas

**Integration Dependencies:**
- Must integrate with NestJS application framework
- Will be consumed by authentication guards and user services
- Foundation for OAuth2 scope-based authorization system

## TASK CONTEXT
**Goal:** Configure AWS Cognito user pool with custom attributes for healthcare roles and create the foundational service for backend integration.

**Healthcare-Specific Requirements:**
- Custom attributes for medical licenses, specialties, clinic association
- Support for healthcare provider roles with credential validation
- Multi-language support (Arabic, French) for user profiles
- HIPAA-compliant user management with audit logging

**User Pool Requirements:**
- MFA configuration: OPTIONAL
- Password policy: Strong passwords with 12+ characters
- Account recovery: Email verification only
- Username attributes: Email addresses
- Auto-verified attributes: Email

## BUSINESS DOMAIN CONTEXT
**Healthcare User Management Rules:**
- Medical license validation required for healthcare providers
- Clinic association determines data access boundaries
- Cross-clinic data access prohibited unless explicitly authorized
- All authentication events must be logged for audit compliance

**Role-Based Access Requirements:**
- System Administrators: `admin:full` scope
- Clinic Managers: `clinic:manage` + operational scopes
- Healthcare Providers: clinical + prescription scopes
- Clinical Staff: clinical scopes, limited patient write access
- Administrative Staff: patient + appointment scopes

## FUNCTIONAL REQUIREMENTS
1. **AWS Cognito User Pool Configuration**
   - Configure user pool with healthcare-specific custom attributes
   - Set up appropriate password policies and MFA options
   - Configure email-based account recovery and verification

2. **Custom Healthcare Attributes**
   - medical_license: string (required for providers)
   - license_state: string (license jurisdiction)
   - specialties: string (JSON array of medical specialties)
   - clinic_id: string (associated clinic UUID)
   - user_role: string (system role assignment)
   - preferred_language: string (ar/fr language preference)
   - npi_number: string (National Provider Identifier)

3. **Cognito Service Implementation**
   - Create NestJS service for Cognito operations
   - Implement user authentication methods
   - Provide token validation functionality
   - Handle user registration and profile management

## TECHNICAL REQUIREMENTS
**Performance Requirements:**
- Token validation response time: < 100ms
- Service initialization without blocking application startup
- Efficient AWS SDK configuration with connection pooling

**Security Requirements:**
- Secure storage of AWS credentials via environment variables
- No sensitive data in logs or error messages
- Proper error handling without information leakage
- HTTPS-only communication with AWS services

**Configuration Management:**
- Use NestJS ConfigModule for AWS configuration
- Validate configuration schema on startup
- Support for different environments (dev, staging, production)

## TESTING REQUIREMENTS
**Unit Tests:**
- CognitoService method testing with mocked AWS SDK
- Configuration validation testing
- Error handling scenarios for authentication failures
- Token parsing and validation logic

**Integration Tests:**
- Authentication flow with test user pool
- User registration and profile update workflows
- Custom attribute storage and retrieval
- MFA and password reset flows

**Test Coverage Target:** 90% for service methods and configuration

## DOCUMENTATION REQUIREMENTS
- JSDoc documentation for all public service methods
- Configuration guide for AWS Cognito setup
- Custom attribute definitions and usage guidelines
- Environment variable configuration documentation

## VALIDATION CRITERIA
- [ ] AWS Cognito user pool configured with all custom healthcare attributes
- [ ] CognitoService class implemented with authentication and token validation methods
- [ ] Configuration module properly validates AWS settings on startup
- [ ] Custom attributes can be set and retrieved for healthcare users
- [ ] Error handling provides clear messages without exposing sensitive information
- [ ] Unit tests achieve 90% coverage for service methods
- [ ] Integration tests validate user registration and authentication flows
- [ ] Documentation covers configuration setup and custom attribute usage
- [ ] Service integrates with NestJS dependency injection system
- [ ] Logging captures authentication events without exposing credentials

# PROMPT 2: JWT Authentication Guard and Token Validation System

**Implement NestJS authentication guard for JWT token validation, user context extraction, and request authentication middleware that integrates with the Cognito service.**

## PROJECT CONTEXT
**Backend Architecture:**
- NestJS framework with guard-based authentication
- JWT tokens from AWS Cognito for user authentication
- Request context injection for user information
- Global authentication enforcement with public route exceptions

**File Structure Standards:**
- Guards location: `src/common/guards/`
- Decorators location: `src/common/decorators/`
- Interfaces location: `src/common/interfaces/`
- Follow lowercase-with-dashes naming for files

## EPIC CONTEXT
**Authentication System Requirements:**
- Foundation for protecting all API endpoints
- Must extract user context for business logic services
- Support for public routes that bypass authentication
- Integration point for authorization and scope validation

## TASK CONTEXT
**JWT Integration Requirements:**
- Validate JWT tokens from AWS Cognito
- Extract user information and custom attributes from tokens
- Handle token expiration and refresh scenarios
- Provide clean user context to controllers and services

**Authentication Flow:**
1. Extract token from Authorization header
2. Validate token signature and expiration with Cognito
3. Parse user information and custom attributes
4. Attach user context to request object
5. Allow access to protected resources

## BUSINESS DOMAIN CONTEXT
**Healthcare Security Requirements:**
- All API endpoints require authentication by default
- User context must include clinic association for data isolation
- Medical license and role information needed for authorization
- Audit trail required for all authenticated requests

**User Context Requirements:**
- User ID, email, and clinic association
- Healthcare role and custom attributes
- OAuth2 scopes for authorization decisions
- Preferred language for internationalized responses

## FUNCTIONAL REQUIREMENTS
1. **JWT Authentication Guard**
   - Extract and validate JWT tokens from request headers
   - Integrate with CognitoService for token validation
   - Attach user context to request for downstream services
   - Handle authentication errors with appropriate HTTP status codes

2. **Public Route Support**
   - Decorator to mark routes as public (bypass authentication)
   - Health check and documentation endpoints should be public
   - Login and registration endpoints are public by default

3. **User Context Injection**
   - Custom decorator to inject current user into controller methods
   - User object with healthcare-specific attributes
   - Clinic context for multi-tenant data isolation

## TECHNICAL REQUIREMENTS
**Performance Requirements:**
- Token validation < 100ms per request
- Minimal performance impact on API response times
- Efficient token parsing and user context building

**Error Handling:**
- Clear error messages for missing or invalid tokens
- Proper HTTP status codes (401 for unauthorized, 403 for forbidden)
- Security-conscious error messages (no information leakage)

**Integration Requirements:**
- Must work with NestJS global guards
- Compatible with existing CognitoService
- Support for future authorization guards

## TESTING REQUIREMENTS
**Unit Tests:**
- Guard behavior with valid and invalid tokens
- Public route bypass functionality
- Error handling for various authentication failure scenarios
- User context extraction and decoration

**Integration Tests:**
- End-to-end authentication flow with test tokens
- Public and protected endpoint access patterns
- Token expiration and refresh handling
- Multi-tenant context validation

**Test Scenarios:**
- Valid authentication with healthcare user
- Missing token rejection
- Expired token handling
- Invalid token format rejection
- Public route access without authentication

## DOCUMENTATION REQUIREMENTS
- Usage examples for authentication guard
- Public route decorator documentation
- User context object structure definition
- Error handling and status code reference

## VALIDATION CRITERIA
- [ ] JwtAuthGuard class implements CanActivate interface correctly
- [ ] Token extraction from Authorization header works reliably
- [ ] Integration with CognitoService for token validation is functional
- [ ] User context extraction includes all healthcare-specific attributes
- [ ] Public route decorator allows bypass of authentication
- [ ] CurrentUser decorator injects user context into controllers
- [ ] Error handling provides appropriate HTTP status codes
- [ ] Unit tests cover all authentication scenarios with 90%+ coverage
- [ ] Integration tests validate complete authentication flow
- [ ] Performance requirements met for token validation timing
- [ ] Security requirements met with no information leakage in errors
- [ ] Documentation covers usage patterns and error handling

# PROMPT 3: OAuth2 Scope-Based Authorization System

**Implement OAuth2 scope-based authorization system with NestJS guards and decorators that enforce healthcare workflow permissions based on user roles and scopes.**

## PROJECT CONTEXT
**Authorization Architecture:**
- OAuth2 scope-based access control system
- Role-based permissions aligned with healthcare workflows  
- Granular endpoint protection with scope requirements
- Integration with JWT authentication system

**Healthcare Permission Model:**
- Scopes follow pattern: `resource:action` (e.g., `patient:write`, `clinical:read`)
- Role hierarchy with scope inheritance
- Workflow-specific permissions for patient care, appointments, billing

## EPIC CONTEXT
**Workflow Role Standardization:**
- System Administrator: `admin:full` - Complete system access
- Clinic Manager: `clinic:manage` + operational scopes
- Healthcare Provider: clinical + prescription scopes  
- Clinical Staff: clinical scopes, limited patient write
- Administrative Staff: patient + appointment scopes, limited clinical read

**Scope Definitions:**
- `admin:full` - Complete system administration
- `clinic:manage` - Clinic operations management
- `patient:write` - Create and modify patient records
- `patient:read` - View patient information
- `appointment:write` - Schedule and modify appointments
- `appointment:read` - View appointment information
- `clinical:write` - Document clinical information
- `clinical:read` - Access clinical records
- `billing:write` - Process payments and billing
- `billing:read` - View financial information
- `prescription:write` - Create and manage prescriptions
- `prescription:read` - View prescription information

## TASK CONTEXT
**Authorization Requirements:**
- Scope-based access control for all protected endpoints
- Flexible scope checking with AND/OR logic support
- Clear error messages for insufficient permissions
- Integration with existing JWT authentication guard

**Implementation Approach:**
- Authorization guard that checks required scopes against user scopes
- Decorator to specify endpoint scope requirements
- Support for multiple scopes with different validation logic
- Proper error handling for authorization failures

## BUSINESS DOMAIN CONTEXT
**Healthcare Authorization Rules:**
- Clinical scopes restricted to licensed/trained personnel only
- Prescription write access limited to licensed prescribers
- Patient data access controlled by clinic association
- Administrative functions separated from clinical functions

**Security Requirements:**
- Minimum necessary access principle
- Audit trail for authorization decisions
- No scope elevation without proper role assignment
- Clear separation between operational and clinical permissions

## FUNCTIONAL REQUIREMENTS
1. **Scopes Authorization Guard**
   - Extract required scopes from route metadata
   - Compare against user's assigned scopes
   - Support for multiple scope validation patterns
   - Proper error handling for insufficient permissions

2. **Scope Requirement Decorator**
   - Decorator to specify required scopes on controllers/methods
   - Support for single and multiple scope requirements
   - Clear syntax for complex scope requirements

3. **Scope Validation Logic**
   - AND logic: user must have ALL specified scopes
   - OR logic: user must have AT LEAST ONE of specified scopes
   - Hierarchical scope checking (admin:full includes all scopes)

## TECHNICAL REQUIREMENTS
**Performance Requirements:**
- Scope validation < 50ms per request
- Efficient scope comparison algorithms
- Minimal impact on API response times

**Error Handling:**
- HTTP 403 (Forbidden) for insufficient scopes
- Clear error messages indicating required scopes
- Security-conscious responses (no scope enumeration)

**Integration Requirements:**
- Must work after JWT authentication guard
- Compatible with existing NestJS guard system
- Support for controller and method-level scope requirements

## TESTING REQUIREMENTS
**Unit Tests:**
- Scope comparison logic with various scope combinations
- Guard behavior with different user scope assignments
- Error handling for missing or insufficient scopes
- Decorator metadata extraction and processing

**Integration Tests:**
- End-to-end authorization flow with different user roles
- Multi-scope endpoint protection scenarios
- Hierarchical scope validation (admin access)
- Error response format validation

**Test Scenarios:**
- Healthcare provider accessing clinical endpoints
- Administrative staff blocked from clinical data
- System admin accessing all endpoints
- Insufficient scope rejection scenarios

## DOCUMENTATION REQUIREMENTS
- Scope definitions and healthcare workflow alignment
- Usage examples for scope decorators
- Authorization error handling guide
- Role-to-scope mapping documentation

## VALIDATION CRITERIA
- [ ] ScopesGuard class implements CanActivate interface correctly
- [ ] RequireScopes decorator properly sets metadata for scope requirements
- [ ] Scope validation logic correctly handles AND/OR patterns
- [ ] Healthcare-specific scopes aligned with workflow requirements
- [ ] Hierarchical scope checking works (admin:full access)
- [ ] Error handling provides clear 403 responses for insufficient permissions
- [ ] Integration with JWT authentication guard works seamlessly
- [ ] Unit tests cover all scope validation scenarios with 90%+ coverage
- [ ] Integration tests validate role-based access patterns
- [ ] Performance requirements met for authorization checking
- [ ] Documentation covers all scope definitions and usage patterns
- [ ] Security requirements met with appropriate error responses

# PROMPT 4: User Context Integration and Authentication Module Setup

**Complete the authentication system by implementing user context services, authentication module configuration, and integration with the broader NestJS application architecture.**

## PROJECT CONTEXT
**Module Architecture:**
- Feature-based module organization with shared authentication module
- Dependency injection for authentication services across application
- Global guard configuration with module-level setup
- Integration with existing application modules

**Service Integration:**
- User context service for profile management
- Authentication service orchestration
- Module exports for application-wide usage
- Configuration service integration

## EPIC CONTEXT
**Authentication Foundation:**
- Complete authentication system ready for frontend integration
- User management capabilities for admin interfaces
- Foundation for all subsequent feature development
- Audit and logging system integration

## TASK CONTEXT
**Integration Requirements:**
- Authentication module that exports all authentication services
- Global guard configuration for application-wide protection
- User context service for profile and clinic management
- Complete authentication flow from login to protected resource access

**Module Dependencies:**
- Config module for AWS and application settings
- Logger module for audit trail
- Database module for user profile storage (future integration)

## BUSINESS DOMAIN CONTEXT
**Healthcare User Management:**
- User profiles with healthcare-specific attributes
- Clinic association and multi-tenant data isolation
- Role management and permission assignment
- Audit logging for compliance requirements

**Integration Requirements:**
- Must support user registration and profile updates
- Handle clinic association and role assignment
- Provide user context for business logic services
- Enable audit logging for healthcare compliance

## FUNCTIONAL REQUIREMENTS
1. **Authentication Module**
   - Export all authentication services and guards
   - Configure global authentication guards
   - Module imports and dependency resolution
   - Integration with application root module

2. **User Context Service**
   - User profile retrieval and caching
   - Clinic association management
   - Role and scope resolution
   - Profile update capabilities

3. **Application Integration**
   - Global guard configuration
   - Exception filter integration
   - Audit logging setup
   - Configuration validation

## TECHNICAL REQUIREMENTS
**Module Configuration:**
- Proper dependency injection setup
- Global guard registration
- Provider exports for application use
- Configuration module integration

**Performance Requirements:**
- Efficient user context caching
- Minimal application startup impact
- Optimized guard execution order

**Error Handling:**
- Global exception filter for authentication errors
- Consistent error response format
- Proper logging for debugging and auditing

## TESTING REQUIREMENTS
**Unit Tests:**
- Module configuration and dependency injection
- User context service methods
- Authentication service orchestration
- Guard integration and execution order

**Integration Tests:**
- Complete authentication flow from login to resource access
- User context retrieval and caching
- Multi-tenant data isolation validation
- Audit logging verification

**Test Coverage:**
- All authentication module components
- User context service functionality
- Global guard behavior
- Error handling scenarios

## DOCUMENTATION REQUIREMENTS
- Authentication module setup and configuration guide
- User context service API documentation
- Global guard configuration reference
- Troubleshooting guide for common authentication issues

## VALIDATION CRITERIA
- [ ] AuthenticationModule properly exports all required services and guards
- [ ] Global guard configuration protects all non-public endpoints
- [ ] User context service provides complete user profile information
- [ ] Clinic association enables proper multi-tenant data isolation
- [ ] Role and scope resolution works correctly for all user types
- [ ] Audit logging captures all authentication and authorization events
- [ ] Module integrates properly with NestJS application architecture
- [ ] Performance requirements met for user context operations
- [ ] Unit tests achieve 90%+ coverage for all authentication components
- [ ] Integration tests validate complete authentication workflows
- [ ] Error handling provides consistent responses across all authentication scenarios
- [ ] Documentation enables easy setup and troubleshooting of authentication system