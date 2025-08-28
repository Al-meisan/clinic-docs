# PROMPTS: AWS Cognito Terraform Infrastructure Module

# ~~PROMPT 1: AWS Cognito User Pool Infrastructure Module Creation~~

**Create a comprehensive Terraform module for AWS Cognito User Pool infrastructure that supports the healthcare-specific
user management requirements of the MedFlow clinic management system, including custom healthcare attributes, security
policies, and multi-environment deployment.**

## PROJECT CONTEXT

The MedFlow healthcare management system requires AWS Cognito infrastructure to support user authentication and
authorization with healthcare-specific custom attributes. The infrastructure must be deployed using Terraform modules
for consistency across development, staging, and production environments. The Cognito User Pool must integrate
seamlessly with the NestJS backend service that has been implemented in
`src/infrastructure/aws/cognito/cognito.service.ts`.

**Technology Stack:**

- Infrastructure: Terraform modules with AWS provider
- Authentication: AWS Cognito User Pool with App Client
- Backend Integration: NestJS CognitoService expecting specific configuration
- Environments: Development, staging, production with environment-specific settings

## EPIC CONTEXT

This infrastructure module is part of the Project Setup Epic and provides the foundational authentication infrastructure
required by all subsequent feature development. The Cognito User Pool will serve as the single source of truth for user
authentication across the entire MedFlow ecosystem.

**Integration Dependencies:**

- Must provide outputs compatible with NestJS backend configuration
- Required for frontend authentication flows
- Foundation for role-based access control and OAuth2 scopes
- Integration point for audit logging and healthcare compliance

## TASK CONTEXT

**Goal:** Create production-ready Terraform infrastructure for AWS Cognito User Pool with healthcare-specific custom
attributes and security configurations.

**Backend Integration Requirements:**
The implemented NestJS CognitoService expects the following configuration parameters:

- `userPoolId`: AWS Cognito User Pool ID
- `clientId`: AWS Cognito App Client ID
- `clientSecret`: Optional App Client Secret for enhanced security
- `region`: AWS region for Cognito operations
- `authority`: Optional OIDC authority URL
- `identityPoolId`: Optional Cognito Identity Pool ID for federated identities

**Healthcare Custom Attributes Required:**
Based on the implemented backend service, the following custom attributes must be configured:

- `custom:medical_license` - Medical license number (string, mutable)
- `custom:license_state` - License jurisdiction/state (string, mutable)
- `custom:specialties` - JSON array of medical specialties (string, mutable)
- `custom:clinic_id` - Associated clinic UUID (string, mutable)
- `custom:user_role` - System role assignment (string, mutable)
- `custom:preferred_language` - User language preference: ar/fr (string, mutable)
- `custom:npi_number` - National Provider Identifier (string, mutable)

## BUSINESS DOMAIN CONTEXT

**Healthcare User Management Requirements:**

- Medical license validation for healthcare providers
- Multi-tenancy through clinic association
- Role-based access control for different healthcare workflows
- HIPAA-compliant user data handling with audit trails
- Multi-language support for Arabic and French users

**Security and Compliance Requirements:**

- Strong password policies for healthcare data protection
- MFA capability for high-privilege users
- Email verification for account security
- Proper user data encryption and secure storage
- Account recovery mechanisms without compromising security

**Cost Optimization:**

- Development environment with relaxed security for cost savings
- Production environment with full security and compliance features
- Staging environment balancing cost and production-like testing

## FUNCTIONAL REQUIREMENTS

1. **Cognito User Pool Configuration**
    - User pool with healthcare-optimized settings
    - Email-based username configuration
    - Strong password policy (12+ characters, complexity requirements)
    - MFA configuration: OPTIONAL (configurable per environment)
    - Email verification for new accounts
    - Account recovery via email verification

2. **Custom Healthcare Attributes**
    - All seven healthcare-specific custom attributes as defined in backend
    - Proper attribute types and mutability settings
    - Required vs optional attribute configuration
    - Attribute validation where applicable

3. **App Client Configuration**
    - App client for NestJS backend integration
    - Appropriate authentication flows enabled
    - Optional client secret generation for enhanced security
    - Proper token validity periods
    - Read/write permissions for custom attributes

4. **Environment-Specific Configuration**
    - Development: Relaxed security, shorter token periods, no MFA requirement
    - Staging: Production-like security with test-friendly settings
    - Production: Full security, compliance features, audit logging enabled

5. **Security Policies**
    - Password policy: minimum 12 characters, uppercase, lowercase, numbers, special characters
    - Account lockout policy: 5 failed attempts, 30-minute lockout
    - Token validity: Access tokens (1 hour), Refresh tokens (30 days)
    - Email verification required before account activation

## TECHNICAL REQUIREMENTS

**File Structure:**

```
clinic-infra/
├── modules/
│   └── cognito/
│       ├── main.tf              # Main Cognito resources
│       ├── variables.tf         # Input variables with validation
│       ├── outputs.tf           # Required outputs for backend integration
│       ├── versions.tf          # Provider requirements
│       └── README.md            # Module documentation
└── environments/
    ├── dev/main.tf              # References cognito module
    ├── staging/main.tf          # References cognito module
    └── prod/main.tf             # References cognito module
```

**Required Resources:**

- `aws_cognito_user_pool` - Main user pool with healthcare attributes
- `aws_cognito_user_pool_client` - App client for backend integration
- `aws_cognito_user_pool_domain` - Optional domain for hosted UI (if needed)

**Required Variables:**

- `environment` - Environment name (dev/staging/prod)
- `project_name` - Project name for resource naming (default: "medflow")
- `user_pool_name` - Cognito User Pool name
- `app_client_name` - App Client name
- `enable_mfa` - Boolean to enable/disable MFA (default: false for dev, true for prod)
- `password_minimum_length` - Password minimum length (default: 12)
- `token_validity_access` - Access token validity in hours (default: 1)
- `token_validity_refresh` - Refresh token validity in days (default: 30)
- `generate_client_secret` - Boolean to generate app client secret (default: true)
- `enable_username_case_sensitivity` - Boolean for case sensitivity (default: false)

**Required Outputs:**

- `user_pool_id` - Cognito User Pool ID
- `user_pool_arn` - Cognito User Pool ARN
- `app_client_id` - App Client ID
- `app_client_secret` - App Client Secret (sensitive)
- `user_pool_endpoint` - User Pool endpoint
- `user_pool_domain` - Optional domain if configured

**Resource Naming Convention:**
Format: `{environment}-{project_name}-cognito-{resource_type}`
Examples:

- `dev-medflow-cognito-user-pool`
- `prod-medflow-cognito-app-client`

## TESTING REQUIREMENTS

**Validation Tests:**

- Terraform plan execution without errors across all environments
- Resource creation with proper tags and naming conventions
- Custom attributes properly configured and accessible
- App client configuration enables required authentication flows
- Password policy enforcement validation
- MFA configuration per environment settings

**Integration Validation:**

- Backend NestJS service can connect using generated configuration
- Custom attributes can be set and retrieved via AWS SDK
- Authentication flow works end-to-end
- Token generation and validation functional

**Security Validation:**

- Password policy properly enforced
- Email verification workflow functional
- MFA enrollment available when enabled
- Account lockout policy working correctly

## DOCUMENTATION REQUIREMENTS

**Module Documentation:**

- README.md with complete usage examples
- Variable documentation with descriptions and validation
- Output documentation with usage in backend configuration
- Environment-specific configuration examples

**Integration Guide:**

- How to use module outputs in NestJS configuration
- Environment variable mapping for backend service
- Custom attribute usage patterns
- Security best practices for production deployment

## VALIDATION CRITERIA

- [ ] Terraform module creates Cognito User Pool with all healthcare custom attributes
- [ ] App client configured with appropriate authentication flows and permissions
- [ ] All required outputs provided for backend NestJS service integration
- [ ] Environment-specific configuration properly implemented (dev/staging/prod)
- [ ] Password policy enforces healthcare-grade security requirements
- [ ] MFA configuration available and properly environment-gated
- [ ] Email verification workflow configured and functional
- [ ] Custom attributes support healthcare workflow requirements
- [ ] Resource naming follows established conventions
- [ ] Terraform plan executes without errors across all environments
- [ ] Module documentation complete with usage examples
- [ ] Integration with existing NestJS CognitoService validated
- [ ] Security policies align with healthcare compliance requirements
- [ ] Cost optimization achieved through environment-specific settings

# ~~PROMPT 2: Cognito Identity Pool and Domain Configuration~~

**Extend the Cognito infrastructure with Identity Pool for federated authentication and custom domain configuration for
production-ready deployment with healthcare-specific branding and security features.**

## PROJECT CONTEXT

Building upon the User Pool infrastructure, this prompt adds Cognito Identity Pool support for federated authentication
scenarios and custom domain configuration for professional healthcare branding. The Identity Pool enables integration
with external identity providers (future requirement) and provides AWS credentials for direct AWS service access when
needed.

**Extended Integration Requirements:**

- Identity Pool for federated authentication capabilities
- Custom domain for professional appearance in production
- Enhanced security configuration for healthcare compliance
- Integration with existing User Pool infrastructure

## EPIC CONTEXT

**Advanced Authentication Features:**

- Support for future SAML/OIDC integration with hospital systems
- Direct AWS service access for advanced features
- Professional domain configuration for client-facing authentication
- Foundation for advanced security features and compliance reporting

## TASK CONTEXT

**Goal:** Enhance the Cognito infrastructure with Identity Pool and domain configuration for comprehensive
authentication infrastructure.

**Identity Pool Requirements:**

- Cognito Identity Pool linked to User Pool
- Support for authenticated and unauthenticated access patterns
- IAM roles for different user types and permission levels
- Integration with healthcare-specific AWS resource access

**Domain Configuration:**

- Custom domain for Cognito Hosted UI (optional)
- SSL certificate integration
- Professional branding configuration
- Environment-specific domain patterns

## BUSINESS DOMAIN CONTEXT

**Healthcare Authentication Extensions:**

- Future integration with hospital information systems
- Support for healthcare provider credential federation
- Advanced audit and compliance reporting capabilities
- Professional appearance for client-facing authentication interfaces

**Security Enhancements:**

- Granular AWS resource access control
- Enhanced token management with identity-based permissions
- Support for advanced MFA and security key authentication
- Integration with healthcare compliance monitoring tools

## FUNCTIONAL REQUIREMENTS

1. **Cognito Identity Pool**
    - Identity Pool linked to existing User Pool
    - Authenticated and unauthenticated role configuration
    - Role mappings for healthcare user types
    - Integration with existing custom attributes

2. **IAM Role Configuration**
    - Healthcare-specific IAM roles for different user types
    - Principle of least privilege access patterns
    - S3 access for document management
    - CloudWatch access for audit logging

3. **Custom Domain Setup**
    - Optional custom domain configuration
    - SSL certificate management
    - Domain validation and DNS configuration
    - Environment-specific domain patterns

4. **Enhanced Security Configuration**
    - Advanced token configuration
    - Enhanced attribute-based access control
    - Security monitoring integration
    - Compliance reporting setup

## TECHNICAL REQUIREMENTS

**Additional Resources:**

- `aws_cognito_identity_pool` - Identity Pool for federated authentication
- `aws_cognito_identity_pool_roles_attachment` - Role mappings
- `aws_iam_role` - Authenticated and unauthenticated user roles
- `aws_iam_role_policy` - Healthcare-specific permissions
- `aws_cognito_user_pool_domain` - Custom domain configuration
- `aws_acm_certificate` - SSL certificate for custom domain

**Additional Variables:**

- `enable_identity_pool` - Enable Identity Pool creation (default: false for dev)
- `custom_domain_name` - Custom domain name (optional)
- `certificate_arn` - ACM certificate ARN for custom domain
- `enable_hosted_ui` - Enable Cognito Hosted UI (default: false)
- `authenticated_role_name` - IAM role name for authenticated users
- `unauthenticated_role_name` - IAM role name for unauthenticated users

**Additional Outputs:**

- `identity_pool_id` - Cognito Identity Pool ID
- `authenticated_role_arn` - IAM role ARN for authenticated users
- `unauthenticated_role_arn` - IAM role ARN for unauthenticated users
- `custom_domain_url` - Custom domain URL if configured
- `hosted_ui_url` - Hosted UI URL if enabled

## VALIDATION CRITERIA

- [ ] Identity Pool properly linked to User Pool with role mappings
- [ ] IAM roles configured with healthcare-appropriate permissions
- [ ] Custom domain configuration functional with SSL certificates
- [ ] Hosted UI integration working with custom branding
- [ ] Identity Pool supports both authenticated and unauthenticated access
- [ ] Role mappings properly configured for healthcare user types
- [ ] Security policies align with healthcare compliance requirements
- [ ] Integration validated with existing User Pool infrastructure
- [ ] Environment-specific configuration properly implemented
- [ ] Documentation updated with new features and configuration options

# ~~PROMPT 3: Cognito Module Integration and Environment Configuration~~

**Complete the Cognito infrastructure by integrating the module into all environments with proper configuration
management, secrets handling, and backend service integration validation.**

## PROJECT CONTEXT

This final prompt focuses on properly integrating the Cognito module into the existing environment configurations,
establishing secure secrets management, and validating end-to-end integration with the implemented NestJS backend
service.

**Integration Focus Areas:**

- Environment-specific module usage
- Secure configuration management
- Backend service validation
- Deployment automation
- Monitoring and maintenance setup

## EPIC CONTEXT

**Complete Authentication Infrastructure:**

- Production-ready authentication system
- Automated deployment across environments
- Secure secrets and configuration management
- Monitoring and observability integration
- Documentation for operations and maintenance

## TASK CONTEXT

**Goal:** Complete production-ready deployment of Cognito infrastructure with full environment integration and
operational readiness.

**Environment Integration:**

- Update all environment configurations to use Cognito module
- Implement secure secrets management for sensitive outputs
- Configure backend service with Terraform outputs
- Establish deployment and update procedures

**Operational Readiness:**

- Monitoring and alerting configuration
- Backup and disaster recovery planning
- Cost monitoring and optimization
- Security scanning and compliance validation

## FUNCTIONAL REQUIREMENTS

1. **Environment Configuration Files**
    - Update `environments/dev/main.tf` with Cognito module usage
    - Update `environments/staging/main.tf` with production-like settings
    - Update `environments/prod/main.tf` with full security configuration

2. **Secrets Management**
    - Secure handling of Cognito App Client Secret
    - Integration with AWS Secrets Manager or Parameter Store
    - Environment variable configuration for backend service
    - Secure terraform state management for sensitive outputs

3. **Backend Integration Validation**
    - Validate NestJS service configuration with Terraform outputs
    - End-to-end authentication flow testing
    - Custom attribute functionality verification
    - Performance and reliability testing

4. **Deployment Automation**
    - Update deployment scripts with Cognito module
    - Environment-specific deployment validation
    - Rollback procedures for failed deployments
    - Blue-green deployment support for production updates

5. **Monitoring and Observability**
    - CloudWatch metrics and alarms for Cognito
    - Authentication flow monitoring
    - Error rate and performance tracking
    - Security event monitoring and alerting

## TECHNICAL REQUIREMENTS

**Environment Files Update:**

```hcl
# environments/dev/main.tf
module "cognito" {
  source = "../../modules/cognito"

  environment             = "dev"
  enable_mfa              = false
  generate_client_secret  = false
  password_minimum_length = 8
  token_validity_access   = 8
  enable_identity_pool    = false
  enable_hosted_ui        = true
}

# environments/prod/main.tf
module "cognito" {
  source = "../../modules/cognito"

  environment             = "prod"
  enable_mfa              = true
  generate_client_secret  = true
  password_minimum_length = 12
  token_validity_access   = 1
  enable_identity_pool    = true
  custom_domain_name      = "auth.medflow.com"
  certificate_arn         = var.ssl_certificate_arn
}
```

**Secrets Management Configuration:**

- AWS Systems Manager Parameter Store for non-sensitive configuration
- AWS Secrets Manager for sensitive values (client secrets, tokens)
- Proper IAM permissions for backend service to access secrets
- Terraform remote state encryption for sensitive outputs

**Backend Service Integration:**

- Environment variables mapping from Terraform outputs
- Configuration validation in NestJS service startup
- Health check endpoints for authentication service status
- Integration testing with actual Cognito resources

## TESTING REQUIREMENTS

**Infrastructure Testing:**

- Terraform plan and apply validation across all environments
- Resource dependency validation and proper creation order
- Secrets management functionality testing
- Disaster recovery and backup validation

**Integration Testing:**

- End-to-end authentication flow with backend service
- Custom attribute CRUD operations testing
- Token validation and refresh testing
- Multi-environment configuration validation

**Performance Testing:**

- Authentication performance benchmarking
- Load testing with concurrent users
- Token validation performance testing
- Resource utilization monitoring

**Security Testing:**

- Security configuration validation
- Penetration testing of authentication flows
- Secrets management security validation
- Compliance requirements verification

## OPERATIONAL REQUIREMENTS

**Monitoring Setup:**

- CloudWatch dashboards for Cognito metrics
- Authentication success/failure rate monitoring
- Custom attribute usage monitoring
- Cost monitoring and optimization alerts

**Maintenance Procedures:**

- User pool backup and restoration procedures
- Custom attribute schema update procedures
- Security policy update procedures
- Disaster recovery runbooks

**Documentation:**

- Operations runbook for Cognito management
- Troubleshooting guide for common issues
- Security incident response procedures
- Regular maintenance and update schedules

## VALIDATION CRITERIA

- [ ] All environments properly configured with Cognito module
- [ ] Secrets management implemented securely across all environments
- [ ] Backend NestJS service successfully integrates with deployed infrastructure
- [ ] End-to-end authentication flow validated in all environments
- [ ] Custom healthcare attributes functional and properly secured
- [ ] Monitoring and alerting configured for operational visibility
- [ ] Deployment automation functional with proper rollback procedures
- [ ] Security configuration meets healthcare compliance requirements
- [ ] Performance benchmarks meet or exceed requirements
- [ ] Documentation complete for operations and maintenance
- [ ] Disaster recovery procedures tested and validated
- [ ] Cost optimization implemented and monitored