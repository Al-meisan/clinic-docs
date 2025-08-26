# EPIC: EPIC-000 - Project Setup and Infrastructure

## Epic Overview

### Goals & Value

**Business Goal:** Establish the foundational technical infrastructure and development environment for MedFlow, enabling rapid development of healthcare management features for Algerian single-doctor practices and future multi-provider clinics.

**User Value:** 
- Development team can efficiently build and deploy healthcare features
- Secure, scalable, and cost-effective infrastructure supports clinic operations
- Modern development stack ensures maintainable and performant application
- Standardized setup enables consistent development practices across team

**Success Criteria:** 
- All three repositories (clinic-backend, clinic-portal, clinic-infra) are created and properly configured
- CI/CD pipelines are functional and deploying to development environment
- Authentication system is working with AWS Cognito integration
- Frontend routing and layout system supports future feature development
- All development tools and standards are documented and working
- Development environment can be set up by new developers in under 30 minutes
- Infrastructure provisions successfully in AWS with proper security configurations

### Scope

**In Scope:**
- Repository structure setup for backend, frontend, and infrastructure
- Backend NestJS application with TypeScript, TypeORM, and AWS integrations
- Frontend React application with TypeScript, Tailwind CSS, and shadcn/ui
- AWS infrastructure setup with Terraform (VPC, RDS, ECS, Cognito, S3, CloudWatch)
- Authentication system implementation with AWS Cognito
- Frontend routing configuration with protected routes
- State management setup with Zustand and TanStack Query
- Layout system supporting single-doctor and multi-provider modes
- Development environment configuration (ESLint, Prettier, Jest)
- Basic error handling and logging infrastructure
- Multi-language support foundation (Arabic/French)
- Development documentation and coding standards

**Out of Scope:**
- Business feature implementation (patients, appointments, etc.)
- Production deployment and monitoring setup
- Third-party integrations (payment gateways, SMS services)
- Advanced performance optimizations
- Comprehensive test coverage (beyond basic setup)
- Mobile application development
- Advanced security configurations beyond basics

## User Stories & Requirements

### Primary User Stories

```yaml
story_1:
  as_a: "Software Developer"
  i_want: "to set up the development environment quickly with all required dependencies"
  so_that: "I can start building healthcare features without infrastructure blockers"
  acceptance_criteria:
    - "Backend repository with NestJS, TypeORM, and AWS SDK configured"
    - "Frontend repository with React, TypeScript, and UI framework ready"
    - "All development tools (linting, formatting, testing) working"
    - "Documentation for local development setup available"
    - "Environment variables and configuration properly structured"

story_2:
  as_a: "DevOps Engineer"
  i_want: "to provision AWS infrastructure using Infrastructure as Code"
  so_that: "environments are reproducible, scalable, and properly secured"
  acceptance_criteria:
    - "Terraform modules for all AWS services (VPC, RDS, ECS, Cognito)"
    - "Separate configurations for dev, staging, and production environments"
    - "Infrastructure can be provisioned and destroyed reliably"
    - "Security groups and IAM policies follow least privilege principle"
    - "Resource tagging and cost monitoring configured"

story_3:
  as_a: "Healthcare Provider"
  i_want: "to securely log into the clinic management system"
  so_that: "I can access patient information and clinic features safely"
  acceptance_criteria:
    - "Secure login form with email/password authentication"
    - "AWS Cognito integration for user management"
    - "Role-based access control with healthcare-specific scopes"
    - "Automatic token refresh and session management"
    - "Secure logout that clears all session data"

story_4:
  as_a: "Healthcare Staff"
  i_want: "to navigate through different sections of the application easily"
  so_that: "I can efficiently access the features I need for patient care"
  acceptance_criteria:
    - "Responsive layout supporting desktop and tablet use"
    - "Navigation menu with proper healthcare workflow organization"
    - "Protected routes based on user permissions"
    - "Breadcrumb navigation for complex workflows"
    - "Single-doctor mode with simplified interface"
```

### Detailed Requirements

1. **Backend Infrastructure Setup**
   - Description: Complete NestJS application with database, authentication, and AWS service integrations
   - Priority: HIGH
   - Dependencies: AWS account, Terraform infrastructure

2. **Frontend Application Foundation**
   - Description: React application with routing, authentication, and UI framework
   - Priority: HIGH
   - Dependencies: Backend API endpoints, authentication service

3. **AWS Infrastructure Provisioning**
   - Description: Terraform-managed AWS resources for hosting and services
   - Priority: HIGH
   - Dependencies: AWS account with appropriate permissions

4. **Authentication System Implementation**
   - Description: AWS Cognito integration with OAuth2 scopes and role management
   - Priority: HIGH
   - Dependencies: AWS Cognito service, backend API

5. **Development Environment Configuration**
   - Description: Standardized development tools and workflows
   - Priority: MEDIUM
   - Dependencies: Repository structure, package managers

6. **Multi-language Foundation**
   - Description: Internationalization framework supporting Arabic and French
   - Priority: MEDIUM
   - Dependencies: Frontend application setup

7. **Layout and Navigation System**
   - Description: Responsive layout with healthcare-specific navigation patterns
   - Priority: MEDIUM
   - Dependencies: Frontend routing, authentication

## Domain Model (Epic-Specific)

### Entities

```yaml
User:
  attributes:
    - id: UUID
    - cognitoId: string (AWS Cognito user ID)
    - email: string
    - firstName: string
    - lastName: string
    - role: UserRole
    - scopes: Scope[]
    - isActive: boolean
    - clinicId: UUID
    - providerId: UUID (optional, if user is a provider)
    - lastLoginAt: DateTime
    - preferences: UserPreferences
    - createdAt: DateTime
    - updatedAt: DateTime
  relationships:
    - relates_to: "Clinic"
      type: "many-to-one"
    - relates_to: "Provider"
      type: "one-to-one" (optional)
  business_rules:
    - "Email must be unique across the system"
    - "Cognito ID must match AWS Cognito user pool"
    - "User role determines available scopes"
    - "Clinic ID determines data access boundary"

Clinic:
  attributes:
    - id: UUID
    - name: string
    - mode: ClinicMode (single_doctor/multi_provider)
    - address: Address
    - contactInfo: ContactInfo
    - license: string
    - taxId: string
    - settings: ClinicSettings
    - subscriptionPlan: SubscriptionPlan
    - isActive: boolean
    - createdAt: DateTime
    - updatedAt: DateTime
  business_rules:
    - "License number must be valid and unique"
    - "Clinic mode determines available features"
    - "Single doctor clinics have simplified interface"

UserPreferences:
  attributes:
    - language: string (ar/fr)
    - theme: string (light/dark)
    - timezone: string
    - dateFormat: string
    - timeFormat: string (12h/24h)
    - notifications: NotificationPreferences
  business_rules:
    - "Language must be supported (ar/fr)"
    - "Timezone must be valid IANA timezone"

ClinicSettings:
  attributes:
    - operatingHours: WorkingHours
    - timeZone: string
    - defaultAppointmentDuration: number (minutes)
    - bufferTime: number (minutes)
    - maxAdvanceBooking: number (days)
    - allowWalkIns: boolean
    - reminderSettings: ReminderSettings
  business_rules:
    - "Operating hours must be logical (start < end)"
    - "Default duration must be between 15-480 minutes"
```

### Business Rules

```yaml
rules:
  - rule_id: "BR-S001"
    rule: "User access is controlled through OAuth2 scopes based on roles and permissions"
    applies_to: "Authentication and authorization system"
    validation: "Scope validation on all protected endpoints"
    
  - rule_id: "BR-S002"
    rule: "System Administrator has admin:full scope with complete system access"
    applies_to: "User role assignment and scope management"
    validation: "Role-based scope assignment in Cognito"
    
  - rule_id: "BR-S009"
    rule: "Cross-clinic data access is prohibited unless explicitly authorized"
    applies_to: "All data access operations"
    validation: "Clinic ID validation on all data queries"
    
  - rule_id: "BR-MT001"
    rule: "Single-doctor practices operate in simplified mode with reduced UI complexity"
    applies_to: "Frontend interface and navigation"
    validation: "Clinic mode determines available UI components"
    
  - rule_id: "BR-DI014"
    rule: "User interface language must be selectable (Arabic/French)"
    applies_to: "Frontend internationalization"
    validation: "Language preference stored and applied consistently"
```

## Integration Points

### Dependencies on Other Epics

```yaml
dependencies:
  - epic: "EPIC-001 (Core Patient Management)"
    dependency_type: "HARD"
    what_we_need: "Authentication system, database infrastructure, routing framework"
    
  - epic: "EPIC-002 (Appointment Scheduling)"
    dependency_type: "HARD"
    what_we_need: "Backend API structure, frontend layout system, user permissions"
    
  - epic: "All Future Epics"
    dependency_type: "HARD"
    what_we_need: "Complete infrastructure foundation and development standards"
```

### APIs This Epic Provides

```yaml
provided_apis:
  - endpoint: "POST /auth/login"
    purpose: "User authentication with AWS Cognito"
    consumers: "Frontend login component, mobile applications"
    
  - endpoint: "POST /auth/refresh"
    purpose: "JWT token refresh for session management"
    consumers: "Frontend authentication service"
    
  - endpoint: "POST /auth/logout"
    purpose: "Secure session termination"
    consumers: "Frontend logout functionality"
    
  - endpoint: "GET /health"
    purpose: "System health check for monitoring"
    consumers: "Load balancers, monitoring systems"
    
  - endpoint: "GET /users/profile"
    purpose: "Current user profile and preferences"
    consumers: "Frontend user interface, settings management"
```

### APIs This Epic Consumes

```yaml
consumed_apis:
  - source: "AWS Cognito"
    endpoints: "Authentication, user management, token validation"
    purpose: "User authentication and session management"
    
  - source: "AWS S3"
    endpoints: "File upload, retrieval, and management"
    purpose: "Document storage and file management"
```

## Technical Approach

### Architecture Decisions

```yaml
decisions:
  - decision: "Use NestJS with TypeScript for backend development"
    rationale: "Provides enterprise-grade architecture with excellent TypeScript support and built-in dependency injection"
    impact: "Faster development, better maintainability, strong typing throughout"
    
  - decision: "Implement Infrastructure as Code with Terraform"
    rationale: "Ensures reproducible, version-controlled infrastructure with multi-environment support"
    impact: "Consistent deployments, easier scaling, infrastructure versioning"
    
  - decision: "Use AWS Cognito for authentication and user management"
    rationale: "Provides enterprise-grade security with OAuth2/OIDC compliance and reduces custom auth development"
    impact: "Reduced security risk, faster implementation, scalable user management"
    
  - decision: "Implement React with TypeScript and shadcn/ui for frontend"
    rationale: "Modern, performant UI with excellent TypeScript support and consistent design system"
    impact: "Faster UI development, consistent design, better developer experience"
    
  - decision: "Use Zustand + TanStack Query for state management"
    rationale: "Lightweight, performant state management with excellent server state handling"
    impact: "Simpler state management, better performance, automatic caching"
```

### Performance Requirements

```yaml
performance:
  - metric: "Application startup time"
    target: "< 3 seconds for initial page load"
    measurement: "Web vitals monitoring and performance testing"
    
  - metric: "API response time"
    target: "< 500ms for authentication endpoints"
    measurement: "Application performance monitoring (APM)"
    
  - metric: "Infrastructure provisioning time"
    target: "< 15 minutes for complete environment setup"
    measurement: "Terraform execution time tracking"
    
  - metric: "Development environment setup"
    target: "< 30 minutes from repository clone to running application"
    measurement: "Documentation testing and developer onboarding time"
```

### Security Considerations

```yaml
security:
  - consideration: "Secure secret management for all environments"
    implementation: "AWS Secrets Manager for production, environment variables for development"
    validation: "Security audit of secret storage and access patterns"
    
  - consideration: "Network security with proper VPC configuration"
    implementation: "Private subnets for databases, public subnets for load balancers, security groups with minimal access"
    validation: "Network penetration testing and security group audit"
    
  - consideration: "Application security with input validation and HTTPS"
    implementation: "class-validator for input validation, HTTPS everywhere, CORS configuration"
    validation: "Security scanning and vulnerability assessment"
    
  - consideration: "Authentication security with secure token handling"
    implementation: "JWT tokens with secure storage, automatic refresh, secure logout"
    validation: "Authentication flow security testing"
```

## Current Tasks

- [X] **[TASK-001]:** Repository Structure Setup - Create and configure clinic-backend, clinic-portal, and clinic-infra repositories with proper README and gitignore files
- [ ] **[TASK-002]:** Terraform Infrastructure Base - Create AWS infrastructure modules for VPC, security groups, and basic networking
- [ ] **[TASK-003]:** Backend NestJS Application Setup - Initialize NestJS application with TypeScript, TypeORM, and basic folder structure
- [ ] **[TASK-004]:** Database Infrastructure - Set up PostgreSQL with TypeORM, migrations, and connection configuration
- [ ] **[TASK-005]:** AWS Cognito Integration - Configure AWS Cognito user pool and integrate with backend authentication
- [ ] **[TASK-006]:** Backend Authentication System - Implement JWT handling, guards, and role-based access control
- [ ] **[TASK-007]:** Frontend React Application Setup - Initialize React application with TypeScript, Vite, and development tools
- [ ] **[TASK-008]:** Frontend UI Framework Integration - Set up Tailwind CSS, shadcn/ui components, and design system
- [ ] **[TASK-009]:** Frontend Routing System - Implement React Router with protected routes and navigation structure
- [ ] **[TASK-010]:** Frontend Authentication Implementation - Create login/logout components with Cognito integration
- [ ] **[TASK-011]:** State Management Setup - Configure Zustand stores and TanStack Query for data fetching
- [ ] **[TASK-012]:** Frontend Layout System - Create responsive layouts supporting single-doctor and multi-provider modes
- [ ] **[TASK-013]:** Internationalization Foundation - Set up i18next with Arabic and French language support
- [ ] **[TASK-014]:** Development Environment Configuration - Configure ESLint, Prettier, Jest, and development scripts
- [ ] **[TASK-015]:** CI/CD Pipeline Setup - Create GitHub Actions for testing, building, and deployment automation
- [ ] **[TASK-016]:** Monitoring and Logging Setup - Configure CloudWatch, error tracking, and application monitoring
- [ ] **[TASK-017]:** Documentation and Developer Guide - Create comprehensive setup documentation and coding standards
- [ ] **[TASK-018]:** Security Configuration - Implement security headers, CORS, rate limiting, and security best practices
- [ ] **[TASK-019]:** Environment Configuration Management - Set up environment-specific configurations for dev, staging, and production
- [ ] **[TASK-020]:** Integration Testing Framework - Set up end-to-end testing with proper test data and environments
