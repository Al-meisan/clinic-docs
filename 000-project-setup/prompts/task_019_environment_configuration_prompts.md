# PROMPTS: TASK-019 - Environment Configuration Management

# PROMPT 1: Environment Configuration Foundation Setup

Implement the foundational environment configuration system with type-safe configuration loading, environment-specific settings, and basic configuration service structure for a NestJS healthcare clinic management application.

## PROJECT CONTEXT

**Tech Stack:**
- Backend: TypeScript with NestJS framework
- Infrastructure: AWS services with Terraform IaC
- Architecture: Domain-driven design principles
- Configuration: NestJS ConfigModule with validation

**Project Structure:** 
- Following feature-based folder structure with `src/config/` for configuration files
- Using lowercase with dashes naming convention for folders/files
- Configuration files should be organized by domain (app.config.ts, database.config.ts, etc.)

## EPIC CONTEXT
This is part of the Project Setup and Infrastructure epic that establishes the foundational systems needed for the healthcare clinic management platform. The configuration management system is essential for supporting multiple deployment environments (development, staging, production) with different security requirements and feature sets.

## TASK CONTEXT
**Goal:** Set up comprehensive environment-specific configuration management for development, staging, and production environments with secure secret management, feature flags, and environment-specific settings.

**Priority:** HIGH - Essential infrastructure component that all other services depend on.

**Environment Definitions:**
- `development`: Local development with debug features, relaxed security, verbose logging
- `staging`: Production-like environment for testing with production security, limited data
- `production`: Live production environment with maximum security, performance optimization

## BUSINESS DOMAIN CONTEXT
**Healthcare Compliance:** Configuration management must support healthcare compliance requirements with secure handling of sensitive data across different environments.

**Multi-Environment Support:** The clinic management system needs to operate consistently across development, staging, and production with appropriate security levels for each environment.

## FUNCTIONAL REQUIREMENTS

1. **Environment-Specific Configuration Loading**
   - Support for development, staging, and production environments
   - Automatic environment detection based on NODE_ENV
   - Type-safe configuration objects with validation

2. **Base Configuration Structure**
   - Database connection settings with environment-specific defaults
   - API configuration with timeouts and retry policies
   - Application settings (port, CORS, rate limiting)

3. **Configuration Service Architecture**
   - Injectable NestJS service for configuration access
   - Centralized configuration loading with error handling
   - Configuration caching for performance

## TECHNICAL REQUIREMENTS

```yaml
performance:
  - requirement: "Configuration loading time < 2 seconds"
    target: "Fast application startup with configuration initialization"

compatibility:
  - requirement: "Multi-environment deployment support"
    scope: "Consistent configuration management across all environments"
    
  - requirement: "Local development environment configuration"
    scope: "Easy setup for developers with minimal configuration"

architecture:
  - requirement: "Type-safe configuration with validation schemas"
    implementation: "Use Joi or class-validator for configuration validation"
    
  - requirement: "Environment-specific configuration files"
    implementation: "Separate configuration objects per environment"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- Configuration loading for each environment (dev/staging/prod)
- Configuration service initialization and dependency injection
- Error handling for invalid or missing configuration
- Type safety validation of configuration objects

**Integration Tests:**
- Application startup with valid configuration for each environment
- Configuration service integration with other NestJS modules
- Environment variable override functionality

## DOCUMENTATION REQUIREMENTS
- Configuration setup guide in README
- Environment variable documentation with examples
- Configuration service usage examples for other developers

## VALIDATION CRITERIA
- [ ] Environment-specific configuration files created (development.config.ts, staging.config.ts, production.config.ts)
- [ ] AppConfigService injectable service implemented with type-safe interfaces
- [ ] Configuration loading automatically detects environment from NODE_ENV
- [ ] Database, API, and application configuration interfaces defined
- [ ] Configuration validation prevents application startup with invalid settings
- [ ] All configuration access goes through the central service (no direct process.env usage)
- [ ] Unit tests achieve >= 90% coverage for configuration loading and service
- [ ] Integration tests verify application startup for all three environments
- [ ] Error messages are clear and actionable when configuration is invalid
- [ ] Configuration service is ready for extension with secret management and feature flags

---

# PROMPT 2: AWS Secrets Manager Integration

Implement secure secret management integration with AWS Secrets Manager for the NestJS configuration system, enabling secure storage and retrieval of sensitive configuration data like database passwords, API keys, and encryption secrets.

## PROJECT CONTEXT

**AWS Infrastructure:**
- Using AWS services with cost-effective approach
- AWS Secrets Manager for secure secret storage
- IAM-based access control for configuration management
- Regional deployment with proper AWS SDK configuration

**Security Requirements:**
- Healthcare compliance requires secure configuration management
- Secrets encrypted at rest and in transit using AWS KMS
- Role-based access control for secret management
- Configuration audit trail for compliance

## EPIC CONTEXT
Part of the Project Setup and Infrastructure epic, building on the foundational configuration system to add enterprise-grade security for sensitive configuration data. This enables the healthcare platform to securely manage database credentials, API keys, and other sensitive settings across environments.

## TASK CONTEXT
**Dependencies:** Configuration foundation from PROMPT 1 must be completed first
**Integration:** Extends the AppConfigService to retrieve secrets from AWS Secrets Manager
**Priority:** HIGH - Critical for production deployment security

## BUSINESS DOMAIN CONTEXT
**Healthcare Data Security:** Medical applications require strict security controls for all sensitive configuration data, including database connections, external API integrations, and encryption keys.

**Compliance Requirements:** Configuration audit trails and encrypted secret storage are necessary for healthcare compliance (HIPAA, SOC 2).

## FUNCTIONAL REQUIREMENTS

1. **AWS Secrets Manager Client Integration**
   - AWS SDK integration with proper error handling
   - Regional configuration based on deployment environment
   - Automatic retry logic for transient failures

2. **Secret Retrieval System**
   - Secure retrieval of database credentials
   - API key management for external integrations
   - Encryption key retrieval for data protection

3. **Secret Caching Strategy**
   - Memory-based caching with TTL for performance
   - Cache invalidation for security
   - Fallback mechanisms during AWS service outages

4. **Environment-Specific Secret Management**
   - Different secret paths per environment (dev/staging/prod)
   - Development environment fallbacks for local development
   - Production-grade secret rotation support

## TECHNICAL REQUIREMENTS

```yaml
security:
  - requirement: "Secret encryption at rest and in transit"
    implementation: "AWS KMS encryption for sensitive configuration data"
    
  - requirement: "Role-based configuration access"
    implementation: "IAM-based access control for configuration management"
    
  - requirement: "Configuration audit trail"
    implementation: "All configuration changes logged and tracked"

performance:
  - requirement: "Secret retrieval time < 500ms"
    target: "Minimal impact on application startup time"
    
  - requirement: "Secret caching with 5-minute TTL"
    target: "Balance security and performance"

reliability:
  - requirement: "Graceful fallback during AWS outages"
    implementation: "Cached secrets or default values for non-critical settings"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- AWS Secrets Manager client initialization and configuration
- Secret retrieval with mocked AWS SDK responses
- Error handling for network failures and invalid secrets
- Cache functionality with TTL behavior
- Environment-specific secret path resolution

**Integration Tests:**
- End-to-end secret retrieval from actual AWS Secrets Manager (test environment)
- Database connection using retrieved credentials
- Secret rotation handling without application restart
- IAM permission validation for different roles

**Security Tests:**
- Verify secrets are not logged or exposed in error messages
- Validate encrypted storage and transmission
- Test access control with different IAM roles

## DOCUMENTATION REQUIREMENTS
- AWS IAM policy requirements for secret access
- Secret naming conventions and path structure
- Development environment setup with local secrets
- Secret rotation procedures and best practices

## VALIDATION CRITERIA
- [ ] AWS Secrets Manager client properly configured with regional settings
- [ ] Database credentials retrieved securely from AWS Secrets Manager
- [ ] API keys and encryption secrets managed through secret service
- [ ] Secret caching implemented with appropriate TTL and security considerations
- [ ] Environment-specific secret paths working (clinic-app/dev/*, clinic-app/staging/*, clinic-app/prod/*)
- [ ] Graceful error handling for AWS service outages with fallback mechanisms
- [ ] No sensitive data exposed in logs or error messages
- [ ] Secret retrieval performance meets < 500ms requirement
- [ ] Unit tests cover secret retrieval, caching, and error scenarios with >= 90% coverage
- [ ] Integration tests validate end-to-end secret management flow
- [ ] IAM policies documented and tested for proper access control
- [ ] Development environment supports local secrets without AWS dependency

---

# PROMPT 3: Feature Flag System Implementation

Implement a comprehensive feature flag system with runtime toggles, environment-specific feature controls, and management interface for the healthcare clinic management application, enabling controlled feature rollouts and A/B testing capabilities.

## PROJECT CONTEXT

**Configuration Architecture:**
- Building on the established configuration foundation and secret management
- Integration with both NestJS backend and React frontend
- Redis-based configuration cache for dynamic updates
- Support for gradual rollouts and A/B testing

**Healthcare Domain Requirements:**
- Patient portal features can be toggled per environment
- Advanced search capabilities controlled by feature flags
- Multi-language support enablement per deployment
- Audit logging features controlled by compliance requirements

## EPIC CONTEXT
Extends the Project Setup and Infrastructure epic by adding dynamic feature control capabilities. This enables the healthcare platform to perform controlled feature rollouts, support different deployment configurations, and enable/disable features based on compliance or business requirements.

## TASK CONTEXT
**Dependencies:** 
- Configuration foundation (PROMPT 1) completed
- Secret management (PROMPT 2) integrated
- Redis infrastructure available for caching

**Integration Points:**
- Backend: NestJS service providing feature flag evaluation
- Frontend: React hooks and context for feature flag consumption
- Management: API endpoints for feature flag administration

## BUSINESS DOMAIN CONTEXT

**Healthcare Feature Management:**
- Patient portal access needs controlled rollout
- Advanced clinical features require provider certification
- Multi-language support varies by clinic location
- Compliance features mandatory in production environments

**Operational Requirements:**
- Features must be toggleable without deployment
- Different feature sets for single-doctor vs multi-provider clinics
- Emergency feature toggles for incident response

## FUNCTIONAL REQUIREMENTS

1. **Feature Flag Definition System**
   - Type-safe feature flag interfaces with boolean, string, and numeric values
   - Environment-specific default values
   - User/role-based feature targeting

2. **Runtime Feature Evaluation**
   - Real-time feature flag evaluation with < 10ms response time
   - Caching strategy with automatic cache invalidation
   - Fallback to default values during service outages

3. **Dynamic Feature Management**
   - API endpoints for feature flag administration
   - Feature flag updates without application restart
   - Audit trail for all feature flag changes

4. **Frontend Integration**
   - React hooks for feature flag consumption
   - Component-level feature toggles
   - Context provider for feature flag state management

## TECHNICAL REQUIREMENTS

```yaml
performance:
  - requirement: "Feature flag evaluation < 10ms"
    target: "Minimal performance impact from feature toggles"
    
  - requirement: "Feature flag caching with Redis"
    target: "Sub-millisecond evaluation for cached flags"

reliability:
  - requirement: "Fallback to default values"
    implementation: "Graceful degradation when feature service unavailable"
    
  - requirement: "Feature flag state persistence"
    implementation: "Redis-based storage with backup to configuration files"

security:
  - requirement: "Role-based feature access"
    implementation: "Integration with authentication system for feature targeting"
    
  - requirement: "Feature flag audit logging"
    implementation: "All changes logged with user, timestamp, and reason"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- Feature flag evaluation logic for different value types
- Cache behavior and invalidation strategies  
- Default value fallback mechanisms
- Feature flag service initialization and dependency injection

**Integration Tests:**
- End-to-end feature flag evaluation from Redis cache
- Feature flag API endpoints for administration
- React hooks integration with feature flag service
- Feature flag updates reflected in real-time

**Frontend Tests:**
- React component rendering based on feature flags
- Feature flag context provider functionality
- Conditional rendering and navigation based on flags

## DOCUMENTATION REQUIREMENTS
- Feature flag naming conventions and best practices
- Guide for adding new feature flags to the system
- React component integration examples
- Feature flag management API documentation

## VALIDATION CRITERIA
- [ ] FeatureFlags interface defined with all clinic management features (patientPortal, advancedSearch, multiLanguage, auditLogging)
- [ ] Feature flag evaluation service with Redis caching and < 10ms response time
- [ ] Environment-specific feature flag defaults configured for dev/staging/production
- [ ] API endpoints for feature flag management (GET, PUT) with proper authentication
- [ ] React useFeatureFlags hook implemented for frontend consumption
- [ ] Feature flag context provider managing state across React application
- [ ] Dynamic feature flag updates without application restart
- [ ] Audit trail logging all feature flag changes with user attribution
- [ ] Fallback to default values when Redis/feature service unavailable
- [ ] Unit tests covering feature evaluation, caching, and error scenarios >= 90% coverage
- [ ] Integration tests validating end-to-end feature flag workflow
- [ ] Frontend tests ensuring components respond correctly to feature flag changes
- [ ] Documentation for developers on adding and managing feature flags

---

# PROMPT 4: Configuration Validation and Error Handling

Implement comprehensive configuration validation with schema-based validation, startup error handling, and configuration completeness checks to ensure the healthcare clinic management application fails fast with clear error messages when configuration is invalid or incomplete.

## PROJECT CONTEXT

**Configuration Dependencies:**
- Building on configuration foundation, secret management, and feature flags
- Integration with NestJS application startup lifecycle
- Validation using Joi schema validation library
- Clear error messaging for developer troubleshooting

**Error Handling Strategy:**
- Fail-fast approach preventing application startup with invalid configuration
- Detailed error messages with remediation suggestions
- Different validation rules for different environments

## EPIC CONTEXT
Completes the configuration management foundation by ensuring configuration reliability and developer experience. This prevents configuration-related production issues and provides clear feedback during development and deployment processes.

## TASK CONTEXT
**Dependencies:** 
- Configuration foundation (PROMPT 1) completed
- Secret management (PROMPT 2) integrated
- Feature flag system (PROMPT 3) operational

**Integration:** Integrates with NestJS application bootstrap and startup lifecycle

## BUSINESS DOMAIN CONTEXT

**Healthcare System Reliability:**
- Medical applications cannot start with incomplete configuration
- Database connections must be validated before patient data access
- External API configurations must be verified before clinical integrations

**Compliance Requirements:**
- Configuration validation ensures compliance settings are properly configured
- Audit logging configuration must be validated in production environments

## FUNCTIONAL REQUIREMENTS

1. **Schema-Based Validation**
   - Joi validation schemas for all configuration categories
   - Environment-specific validation rules
   - Required vs optional configuration validation

2. **Startup Validation Process**
   - Pre-startup configuration validation
   - Application startup prevention on validation failure
   - Clear error messages with remediation steps

3. **Configuration Completeness Checks**
   - Database connection validation
   - AWS service connectivity validation
   - External API endpoint validation

4. **Developer Experience**
   - Descriptive error messages with exact missing/invalid configurations
   - Suggestions for fixing configuration issues
   - Configuration template generation

## TECHNICAL REQUIREMENTS

```yaml
reliability:
  - requirement: "Application startup fails with invalid configuration"
    implementation: "Joi schema validation during application bootstrap"
    
  - requirement: "Clear error messages with remediation steps"
    implementation: "Custom error messages with specific configuration paths"

validation:
  - requirement: "Database connection validation"
    implementation: "Test connection during startup validation"
    
  - requirement: "AWS service connectivity check"
    implementation: "Validate AWS credentials and service availability"
    
  - requirement: "Feature flag configuration consistency"
    implementation: "Validate feature flags against schema"

developer_experience:
  - requirement: "Configuration validation in < 5 seconds"
    target: "Fast feedback during development"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- Joi schema validation for valid and invalid configurations
- Error message generation for different validation failures
- Environment-specific validation rule application
- Configuration completeness checking logic

**Integration Tests:**
- Application startup success with valid configuration
- Application startup failure with invalid configuration
- Database connection validation during startup
- AWS service validation during startup

**Error Scenario Tests:**
- Missing required environment variables
- Invalid database connection strings
- Unreachable AWS services
- Invalid feature flag configurations

## DOCUMENTATION REQUIREMENTS
- Configuration troubleshooting guide
- Required environment variables reference
- Common configuration errors and solutions
- Configuration validation setup for CI/CD

## VALIDATION CRITERIA
- [ ] Joi validation schemas defined for database, API, security, and feature flag configurations
- [ ] Environment-specific validation rules (stricter for production)
- [ ] Application startup validation prevents launch with invalid configuration
- [ ] Database connection tested and validated during startup
- [ ] AWS service connectivity verified during configuration validation
- [ ] Feature flag configuration validated against defined schema
- [ ] Error messages include specific configuration paths and remediation suggestions
- [ ] Configuration validation completes in < 5 seconds
- [ ] Unit tests cover all validation scenarios with >= 90% coverage
- [ ] Integration tests verify application startup behavior with valid/invalid configurations
- [ ] Error scenario tests validate all common configuration failure modes
- [ ] Documentation provides clear troubleshooting guidance for configuration issues
- [ ] CI/CD pipeline integration ready for automated configuration validation

---

# PROMPT 5: Configuration Management CLI Tool

Develop a command-line interface tool for managing application configuration, secrets, and feature flags across different environments, providing developers and DevOps teams with efficient tools for configuration administration and deployment automation.

## PROJECT CONTEXT

**Configuration Management Ecosystem:**
- CLI tool integrates with established configuration, secret management, and feature flag systems
- AWS SDK integration for secret management operations
- Support for all three environments (development, staging, production)
- Integration with CI/CD pipeline for automated configuration deployment

**Developer Workflow:**
- Local development configuration setup
- Environment-specific secret management
- Feature flag administration without direct database access
- Configuration validation and troubleshooting tools

## EPIC CONTEXT
Provides the operational tooling needed to manage the configuration system established in the previous prompts. This CLI tool enables efficient configuration management workflows and supports the DevOps processes needed for the healthcare clinic management platform.

## TASK CONTEXT
**Dependencies:** 
- All previous configuration prompts (1-4) completed
- AWS credentials and permissions configured
- Node.js CLI development environment available

**Integration:** Standalone CLI tool that operates on the same configuration system used by the applications

## BUSINESS DOMAIN CONTEXT

**Operational Efficiency:**
- DevOps teams need tools for managing configuration across multiple environments
- Healthcare compliance requires audit trails for all configuration changes
- Secure secret management workflow for production deployments

**Development Support:**
- Developers need easy configuration setup for local development
- Feature flag management for testing and rollout processes

## FUNCTIONAL REQUIREMENTS

1. **Secret Management Commands**
   - Set secrets in AWS Secrets Manager with environment targeting
   - Retrieve secrets for verification and troubleshooting
   - List available secrets per environment
   - Secret validation and format checking

2. **Configuration Validation Commands**
   - Validate configuration for specific environments
   - Check configuration completeness
   - Test database and service connectivity
   - Generate configuration templates

3. **Feature Flag Management Commands**
   - Enable/disable feature flags per environment
   - List current feature flag states
   - Bulk feature flag updates
   - Feature flag deployment verification

4. **Environment Management Commands**
   - Environment configuration comparison
   - Configuration sync between environments (non-sensitive data)
   - Environment setup automation
   - Configuration backup and restore

## TECHNICAL REQUIREMENTS

```yaml
usability:
  - requirement: "Clear command structure with help documentation"
    implementation: "Commander.js with comprehensive help system"
    
  - requirement: "Interactive prompts for complex operations"
    implementation: "Inquirer.js for guided configuration workflows"

security:
  - requirement: "Secure secret handling in CLI operations"
    implementation: "AWS SDK with proper credential management"
    
  - requirement: "Audit logging for all CLI operations"
    implementation: "Configuration change logging with user attribution"

reliability:
  - requirement: "Robust error handling and recovery"
    implementation: "Clear error messages with suggested remediation"
    
  - requirement: "Configuration validation before applying changes"
    implementation: "Dry-run mode and confirmation prompts"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- CLI command parsing and argument validation
- AWS SDK integration for secret operations
- Configuration validation logic
- Error handling and user feedback

**Integration Tests:**
- End-to-end secret management workflow
- Feature flag management operations
- Configuration validation against actual environments
- CLI tool integration with existing configuration system

**Manual Testing:**
- CLI usability and developer experience
- Help documentation completeness
- Error message clarity and actionability
- Interactive prompt functionality

## DOCUMENTATION REQUIREMENTS
- CLI command reference with examples
- Configuration management workflow guide
- Troubleshooting guide for common CLI issues
- Integration guide for CI/CD pipeline usage

## VALIDATION CRITERIA
- [ ] CLI tool (`config-manager`) executable with clear command structure
- [ ] Secret management commands (set-secret, get-secret, list-secrets) operational
- [ ] Configuration validation command with environment targeting
- [ ] Feature flag management commands (set-flag, get-flags, list-flags) functional
- [ ] Environment-specific operations with proper AWS credentials handling
- [ ] Interactive prompts for complex operations with confirmation steps
- [ ] Comprehensive help system with command usage examples
- [ ] Error handling provides clear, actionable error messages
- [ ] Audit logging for all configuration changes with user and timestamp
- [ ] Unit tests covering CLI parsing, AWS operations, and error scenarios >= 85% coverage
- [ ] Integration tests validating end-to-end configuration management workflows
- [ ] Documentation includes command reference and workflow examples
- [ ] CI/CD integration ready for automated configuration deployment
- [ ] Developer onboarding workflow simplified with CLI-based setup

---

# PROMPT 6: Frontend Configuration Integration

Implement React frontend integration with the configuration management system, providing type-safe configuration access, feature flag hooks, and environment-specific settings for the healthcare clinic management application's user interface.

## PROJECT CONTEXT

**Frontend Architecture:**
- React with TypeScript using Vite build system
- Feature-based folder structure with shared configuration utilities
- Integration with backend configuration API
- TanStack Query for configuration data fetching and caching

**UI Framework:**
- Tailwind CSS with shadcn/ui components
- Zustand for client-side configuration state management
- React Router for feature flag-based navigation
- i18next for multi-language support controlled by feature flags

## EPIC CONTEXT
Completes the configuration management system by integrating it with the React frontend, enabling the user interface to respond to configuration changes, feature flags, and environment-specific settings. This ensures consistent configuration management across the entire healthcare platform.

## TASK CONTEXT
**Dependencies:** 
- All backend configuration prompts (1-5) completed
- React application setup with TypeScript
- State management and routing systems available
- API client infrastructure established

**Integration:** Bridges backend configuration system with frontend React components

## BUSINESS DOMAIN CONTEXT

**Healthcare UI Configuration:**
- Patient portal features controlled by feature flags
- Multi-language support based on clinic configuration
- Clinical features enabled based on provider licenses
- Mobile responsive behavior controlled by deployment environment

**User Experience Considerations:**
- Seamless feature toggles without page refresh
- Graceful degradation when features are disabled
- Consistent UI behavior across different environments

## FUNCTIONAL REQUIREMENTS

1. **Configuration Context Provider**
   - React context for global configuration access
   - Integration with backend configuration API
   - Real-time updates for feature flag changes
   - Error boundary for configuration loading failures

2. **Feature Flag Hooks System**
   - `useFeatureFlag` hook for individual feature checks
   - `useFeatureFlags` hook for multiple feature access
   - Component-level conditional rendering
   - Navigation control based on feature availability

3. **Environment Configuration Access**
   - Frontend API configuration from environment settings
   - Theme and styling configuration per environment
   - Development vs production behavior toggles
   - Configuration-driven component behavior

4. **Configuration-Driven Components**
   - Conditional component rendering based on feature flags
   - Navigation menu items controlled by feature availability
   - Form fields enabled/disabled by configuration
   - Language selector based on multi-language feature flag

## TECHNICAL REQUIREMENTS

```yaml
performance:
  - requirement: "Configuration loading time < 1 second"
    target: "Fast initial page load with configuration"
    
  - requirement: "Feature flag evaluation < 5ms in React components"
    target: "Negligible impact on component render performance"

usability:
  - requirement: "Graceful degradation when features disabled"
    implementation: "Components hide/show smoothly based on feature flags"
    
  - requirement: "Real-time feature flag updates"
    implementation: "Configuration updates without page refresh"

developer_experience:
  - requirement: "Type-safe configuration access"
    implementation: "TypeScript interfaces for all configuration options"
    
  - requirement: "Easy feature flag integration"
    implementation: "Simple hooks for component-level feature checking"
```

## TESTING REQUIREMENTS

**Unit Tests:**
- Configuration context provider with different config states
- Feature flag hooks with various flag combinations
- Component conditional rendering based on feature flags
- Error handling for configuration loading failures

**Integration Tests:**
- End-to-end configuration loading and feature flag evaluation
- Navigation behavior based on feature flag states
- Component integration with backend configuration API
- Real-time configuration updates in React components

**Component Tests:**
- Feature-flagged component rendering in different states
- Navigation menu behavior with different feature combinations
- Form behavior based on configuration settings

## DOCUMENTATION REQUIREMENTS
- React hooks usage guide for configuration and feature flags
- Component integration examples
- Configuration-driven component development patterns
- Troubleshooting guide for frontend configuration issues

## VALIDATION CRITERIA
- [ ] ConfigurationProvider React context managing global configuration state
- [ ] useFeatureFlag hook for individual feature flag evaluation
- [ ] useFeatureFlags hook providing access to all feature flags
- [ ] Configuration API client integrated with TanStack Query for caching
- [ ] Feature flag-based conditional component rendering working correctly
- [ ] Navigation menu items show/hide based on feature flag availability
- [ ] Environment-specific API configuration properly loaded in frontend
- [ ] Real-time feature flag updates reflected in UI without page refresh
- [ ] Type-safe configuration interfaces for all frontend configuration options
- [ ] Error boundaries handle configuration loading failures gracefully
- [ ] Unit tests for configuration context, hooks, and error scenarios >= 90% coverage
- [ ] Integration tests validate end-to-end configuration flow from backend to frontend
- [ ] Component tests verify feature flag-based rendering behavior
- [ ] Documentation provides clear examples for developers integrating configuration
- [ ] Development experience optimized with fast configuration loading and clear error messages