# PROMPTS: TASK-017 - Documentation and Developer Guide

# PROMPT 1: Developer Setup Guide Creation

Create a comprehensive developer setup guide that enables new team members to set up the complete MedFlow development environment in under 30 minutes. The guide should include step-by-step instructions, prerequisites, troubleshooting tips, and verification steps for all development tools and services.

## PROJECT CONTEXT

**MedFlow Project Overview:**
- Healthcare clinic management system for Algerian single-doctor practices
- Tech Stack: NestJS Backend with TypeScript, React Frontend with TypeScript, AWS Infrastructure
- Three main repositories: clinic-backend, clinic-portal, clinic-infra
- Uses AWS Cognito for authentication, PostgreSQL for database, Terraform for infrastructure

**Current Repository Structure:**
- Backend: NestJS with TypeScript, TypeORM, AWS SDK integration
- Frontend: React with TypeScript, Vite, Tailwind CSS, shadcn/ui components
- Infrastructure: Terraform modules for AWS services (VPC, RDS, ECS, Cognito, S3)

## EPIC CONTEXT  

**EPIC-000 Project Setup and Infrastructure:**
- Goal: Establish foundational technical infrastructure and development environment
- Success Criteria: Development environment can be set up by new developers in under 30 minutes
- All three repositories properly configured with development tools and standards

## TASK Context

**TASK-017 Documentation and Developer Guide:**
- Goal: Create comprehensive documentation and developer guide covering setup procedures, coding standards, API documentation, and operational procedures
- Priority: HIGH for Developer Setup Guide
- Success Criteria: Complete documentation suite accessible to all team members

## BUSINESS DOMAIN CONTEXT

**Healthcare Application Requirements:**
- Patient data privacy and security compliance
- Multi-language support (Arabic/French)
- Single-doctor and multi-provider clinic modes
- Professional healthcare workflow optimization

## FUNCTIONAL REQUIREMENTS

1. **Environment Prerequisites Documentation**
   - Node.js version requirements and installation instructions
   - Package manager setup (npm/yarn) with recommended versions
   - Git configuration and SSH key setup for repository access
   - AWS CLI installation and configuration for development
   - Docker setup for local database and services

2. **Repository Setup Instructions**
   - Step-by-step cloning of all three repositories
   - Environment variable configuration with examples
   - Database setup and migration procedures
   - AWS service configuration for local development
   - Verification commands to ensure proper setup

3. **Development Workflow Documentation**
   - Code editor setup with recommended extensions
   - Development server startup procedures for backend and frontend
   - Testing framework setup and execution
   - Debugging setup and common debugging scenarios
   - Git workflow and branch management practices

4. **Troubleshooting Guide**
   - Common setup issues and their solutions
   - Port conflicts and resolution steps
   - Database connection problems
   - AWS service connectivity issues
   - Performance optimization for development environment

## TECHNICAL REQUIREMENTS

- Documentation format: Markdown files in a dedicated `/docs` directory
- Interactive elements where possible (copy-paste commands, verification scripts)
- Screenshots for complex setup steps
- Version-specific instructions with update procedures
- Cross-platform support (Windows, macOS, Linux)

## TESTING REQUIREMENTS

**Manual Testing:**
- Test setup guide with a fresh development machine
- Verify 30-minute setup time target with new developer
- Test all provided commands and scripts work correctly
- Validate troubleshooting solutions resolve actual issues
- Verify cross-platform compatibility

**Documentation Testing:**
- Check all links work correctly
- Validate all code examples are syntactically correct
- Ensure environment variable examples are complete
- Test verification commands produce expected output

## DOCUMENTATIONS REQUIREMENTS

**Files to Create:**
- `/docs/development-setup.md` - Main setup guide
- `/docs/prerequisites.md` - Detailed prerequisite requirements
- `/docs/troubleshooting.md` - Common issues and solutions
- `/docs/verification.md` - Setup verification procedures
- `/scripts/verify-setup.sh` - Automated verification script
- `/scripts/install-deps.sh` - Dependency installation script

**Documentation Structure:**
- Clear headings and numbered steps
- Code blocks with syntax highlighting
- Callout boxes for important notes
- Prerequisites checklist format
- Time estimates for each major step

## VALIDATION CRITERIA

- [ ] Complete setup guide covers all three repositories (clinic-backend, clinic-portal, clinic-infra)
- [ ] Prerequisites section lists all required tools with version numbers
- [ ] Step-by-step instructions are clear and unambiguous
- [ ] Environment variable setup includes complete examples with explanations
- [ ] Database setup and migration procedures are documented
- [ ] AWS service configuration for development environment is covered
- [ ] Verification steps confirm successful setup of each component
- [ ] Troubleshooting section addresses at least 5 common setup issues
- [ ] Cross-platform instructions provided for Windows, macOS, and Linux
- [ ] Setup can be completed in under 30 minutes when following the guide
- [ ] All code examples and commands have been tested and work correctly
- [ ] Documentation includes next steps after successful setup

# PROMPT 2: API Documentation System Implementation

Implement a comprehensive API documentation system using Swagger/OpenAPI that auto-generates interactive documentation from code annotations. The system should provide complete endpoint documentation, request/response examples, authentication details, and testing capabilities directly from the documentation interface.

## PROJECT CONTEXT

**MedFlow Backend Architecture:**
- NestJS application with TypeScript
- Domain-driven design with feature modules
- JWT authentication with AWS Cognito integration
- RESTful API design with standardized response formats
- Role-based access control with OAuth2 scopes

**Current API Structure:**
- Authentication endpoints (/auth/login, /auth/refresh, /auth/logout)
- Protected endpoints with role-based access
- Standardized error responses and status codes
- Request validation using class-validator and class-transformer

## EPIC CONTEXT  

**EPIC-000 Project Setup and Infrastructure:**
- Foundation for all future API development
- Enables consistent API documentation standards
- Critical for frontend development and team collaboration

## TASK Context

**TASK-017 Documentation and Developer Guide:**
- API Documentation Priority: HIGH
- Success Criteria: All endpoints documented with request/response examples and Swagger UI
- Interactive testing capabilities for all API endpoints

## BUSINESS DOMAIN CONTEXT

**Healthcare API Requirements:**
- Patient data privacy and security considerations
- Multi-language error messages and descriptions
- Healthcare-specific status codes and error handling
- Audit trail requirements for all data modifications

## FUNCTIONAL REQUIREMENTS

1. **Swagger/OpenAPI Configuration**
   - Configure Swagger module in NestJS application
   - Set up API versioning and base URL configuration
   - Configure authentication schemes (JWT Bearer tokens)
   - Set up response schemas and error models

2. **Endpoint Documentation Standards**
   - Document all existing authentication endpoints
   - Include request/response schemas with examples
   - Document authentication requirements for protected endpoints
   - Provide error response documentation with status codes

3. **Interactive Testing Interface**
   - Configure Swagger UI for endpoint testing
   - Set up authentication flow for testing protected endpoints
   - Include example payloads for all request types
   - Configure proper CORS settings for API testing

4. **Auto-generation from Code**
   - Use decorators for endpoint documentation
   - Configure DTO classes to generate request/response schemas
   - Set up automatic schema generation from TypeScript types
   - Include validation rules in API documentation

## TECHNICAL REQUIREMENTS

**Swagger Configuration:**
- Use @nestjs/swagger package for NestJS integration
- Configure OpenAPI 3.0 specification
- Set up development and production documentation URLs
- Include API versioning in documentation structure

**Documentation Format:**
- JSDoc comments for all controller methods
- Swagger decorators for endpoints (@ApiOperation, @ApiResponse)
- Schema definitions for all DTOs and response types
- Authentication security definitions

## TESTING REQUIREMENTS

**Integration Testing:**
- Test Swagger UI loads correctly and displays all endpoints
- Verify authentication flow works in Swagger interface
- Test all documented endpoints return expected responses
- Validate schema generation matches actual API responses

**Documentation Validation:**
- Check all endpoints are documented with proper descriptions
- Verify request/response examples are accurate and complete
- Test authentication requirements are clearly documented
- Ensure error responses are properly documented

## DOCUMENTATIONS REQUIREMENTS

**Files to Create/Modify:**
- `src/main.ts` - Add Swagger configuration
- `src/config/swagger.config.ts` - Swagger setup configuration
- `src/common/decorators/api-docs.decorator.ts` - Custom documentation decorators
- `src/auth/auth.controller.ts` - Add API documentation to auth endpoints
- `docs/api-documentation.md` - API documentation overview and usage guide

**Configuration Requirements:**
- Environment-specific Swagger URLs
- Security scheme configuration for JWT authentication
- Response model definitions
- Error response standardization

## VALIDATION CRITERIA

- [ ] Swagger UI is accessible at `/api/docs` endpoint
- [ ] All authentication endpoints are fully documented with examples
- [ ] JWT authentication scheme is properly configured in Swagger
- [ ] Request/response schemas are auto-generated from DTOs
- [ ] Interactive testing works for all documented endpoints
- [ ] Error responses include proper status codes and descriptions
- [ ] API documentation includes authentication flow instructions
- [ ] All controller methods have proper JSDoc documentation
- [ ] Swagger decorators are consistently applied across all endpoints
- [ ] Development and production environments have proper Swagger configuration
- [ ] API versioning is reflected in documentation structure
- [ ] Testing interface allows authentication and protected endpoint testing

# PROMPT 3: Coding Standards Documentation Creation

Create comprehensive coding standards and guidelines documentation covering all tech stack components (NestJS backend, React frontend, TypeScript, AWS integrations). The documentation should include detailed patterns, best practices, naming conventions, file organization, and code examples that establish consistent development practices across the healthcare application.

## PROJECT CONTEXT

**MedFlow Technology Stack:**
- Backend: NestJS with TypeScript, TypeORM for database, AWS SDK
- Frontend: React with TypeScript, Tailwind CSS, shadcn/ui components
- Infrastructure: Terraform for AWS resource management
- Database: PostgreSQL with TypeORM migrations

**Established Coding Patterns:**
- Domain-driven design for backend architecture
- Feature-based folder structure for frontend
- OAuth2 scopes for role-based access control
- Standardized error handling and logging

## EPIC CONTEXT  

**EPIC-000 Project Setup and Infrastructure:**
- Standardized development practices critical for team collaboration
- Foundation for consistent code quality across all features
- Enables efficient code reviews and maintenance

## TASK Context

**TASK-017 Documentation and Developer Guide:**
- Coding Standards Priority: HIGH
- Success Criteria: Clear guidelines covering all tech stack components with examples
- Must support healthcare application requirements and compliance needs

## BUSINESS DOMAIN CONTEXT

**Healthcare Application Standards:**
- Patient data security and privacy requirements
- Audit trail and logging for all data modifications
- Multi-language support considerations (Arabic/French)
- Professional healthcare workflow optimization
- Error handling that doesn't expose sensitive information

## FUNCTIONAL REQUIREMENTS

1. **Backend Coding Standards (NestJS/TypeScript)**
   - Module organization and naming conventions
   - Controller, service, and repository patterns
   - DTO design and validation standards
   - Database entity and migration patterns
   - Error handling and logging standards
   - AWS service integration patterns

2. **Frontend Coding Standards (React/TypeScript)**
   - Component design and organization principles
   - State management patterns with Zustand
   - API integration with TanStack Query
   - UI component standards with shadcn/ui
   - Styling conventions with Tailwind CSS
   - Accessibility and internationalization standards

3. **Cross-Stack Standards**
   - TypeScript configuration and type definitions
   - File and folder naming conventions
   - Import/export patterns and organization
   - Code documentation standards (JSDoc)
   - Testing patterns and conventions
   - Git commit message and branch naming standards

4. **Healthcare-Specific Guidelines**
   - Patient data handling and security patterns
   - Audit logging requirements and implementation
   - Error messages that protect patient privacy
   - Multi-language support implementation patterns

## TECHNICAL REQUIREMENTS

**Documentation Structure:**
- Separate sections for backend and frontend standards
- Code examples with explanations for each pattern
- "Do" and "Don't" examples for clarity
- Healthcare-specific considerations highlighted
- Integration patterns between frontend and backend

**Code Quality Standards:**
- ESLint and Prettier configuration standards
- TypeScript strict mode requirements
- Code coverage expectations for testing
- Performance optimization guidelines

## TESTING REQUIREMENTS

**Standards Validation:**
- Create example code following all documented patterns
- Validate ESLint/Prettier configurations work with standards
- Test healthcare-specific patterns with sample data
- Verify TypeScript configurations support all recommended patterns

**Documentation Testing:**
- All code examples compile and run without errors
- Linting rules enforce documented standards
- Testing patterns work with established test framework
- Integration examples work with actual API endpoints

## DOCUMENTATIONS REQUIREMENTS

**Files to Create:**
- `/docs/coding-standards/overview.md` - General standards overview
- `/docs/coding-standards/backend-standards.md` - NestJS/TypeORM patterns
- `/docs/coding-standards/frontend-standards.md` - React/TypeScript patterns
- `/docs/coding-standards/typescript-standards.md` - TypeScript configuration and usage
- `/docs/coding-standards/healthcare-patterns.md` - Healthcare-specific coding patterns
- `/docs/coding-standards/testing-standards.md` - Testing patterns and requirements
- `/examples/` directory with sample code demonstrating all patterns

**Configuration Files:**
- `.eslintrc.js` - Updated with documented standards
- `.prettierrc` - Consistent formatting configuration
- `tsconfig.json` - TypeScript standards configuration

## VALIDATION CRITERIA

- [ ] Backend standards cover NestJS module organization and patterns
- [ ] Controller, service, and repository patterns are clearly documented with examples
- [ ] DTO design and validation standards include healthcare data requirements
- [ ] Frontend standards cover React component organization and state management
- [ ] TypeScript configuration standards are documented with examples
- [ ] File and folder naming conventions are consistent across stack
- [ ] Healthcare-specific patterns address patient data security requirements
- [ ] Code documentation standards include JSDoc requirements and examples
- [ ] Testing patterns cover unit, integration, and end-to-end testing approaches
- [ ] ESLint and Prettier configurations enforce documented standards
- [ ] All code examples are tested and working
- [ ] Standards address multi-language support implementation
- [ ] Error handling patterns protect patient privacy while providing useful feedback
- [ ] AWS integration patterns are documented with security considerations
- [ ] Git workflow and commit message standards are clearly defined

# PROMPT 4: Architecture Documentation Creation

Create comprehensive system architecture documentation that explains the technical design decisions, data flow diagrams, integration patterns, and system components for the MedFlow healthcare management platform. The documentation should serve as a technical reference for developers and architects working on the system.

## PROJECT CONTEXT

**MedFlow System Architecture:**
- Microservices architecture with NestJS backend and React frontend
- AWS cloud infrastructure with managed services (Cognito, RDS, ECS, S3)
- Domain-driven design with feature-based module organization
- Event-driven patterns for audit logging and notifications
- Multi-tenant architecture supporting single-doctor and multi-provider modes

**Current System Components:**
- Authentication service with AWS Cognito integration
- Backend API with role-based access control
- Frontend application with responsive layouts
- Database layer with PostgreSQL and TypeORM
- Infrastructure as Code with Terraform

## EPIC CONTEXT  

**EPIC-000 Project Setup and Infrastructure:**
- Foundation architecture established for future feature development
- Technical decisions made for scalability and healthcare compliance
- Integration patterns defined for AWS services and internal components

## TASK Context

**TASK-017 Documentation and Developer Guide:**
- Architecture Documentation Priority: MEDIUM
- Success Criteria: Technical documentation explaining system design and integration patterns
- Must enable new developers to understand system architecture quickly

## BUSINESS DOMAIN CONTEXT

**Healthcare System Requirements:**
- Patient data security and privacy compliance
- Audit trail requirements for all data modifications
- Multi-language support for Algerian healthcare providers
- Scalability from single-doctor to multi-provider clinics
- Healthcare workflow optimization and efficiency

## FUNCTIONAL REQUIREMENTS

1. **System Overview Documentation**
   - High-level architecture diagram with all major components
   - Technology stack overview and rationale for choices
   - Deployment architecture and environment configurations
   - Scalability considerations and growth patterns

2. **Component Architecture Details**
   - Backend service architecture with module boundaries
   - Frontend application architecture and state management
   - Database design patterns and data organization
   - Authentication and authorization flow documentation
   - AWS services integration and communication patterns

3. **Data Flow Documentation**
   - User authentication and session management flow
   - Request/response patterns between frontend and backend
   - Database transaction patterns and data consistency
   - Event flow for audit logging and notifications
   - Error handling and recovery patterns

4. **Integration Patterns**
   - AWS Cognito integration for user management
   - Database integration with TypeORM patterns
   - Frontend-backend API communication standards
   - Third-party service integration patterns
   - Monitoring and logging integration

## TECHNICAL REQUIREMENTS

**Documentation Format:**
- Architecture diagrams using Mermaid.js for version control compatibility
- Component interaction diagrams with clear boundaries
- Sequence diagrams for complex workflows
- Entity relationship diagrams for database design
- Deployment diagrams for infrastructure understanding

**Technical Depth:**
- Detailed explanation of design decisions and trade-offs
- Performance characteristics and bottlenecks identification
- Security architecture and threat model considerations
- Scalability patterns and horizontal scaling approaches

## TESTING REQUIREMENTS

**Architecture Validation:**
- Verify architecture diagrams match actual implementation
- Test integration patterns work as documented
- Validate security flows match documented patterns
- Confirm scalability assumptions with performance testing

**Documentation Testing:**
- All diagrams render correctly in documentation system
- Links between architecture components are accurate
- Code examples in architecture documentation are functional
- Integration patterns can be followed by developers

## DOCUMENTATIONS REQUIREMENTS

**Files to Create:**
- `/docs/architecture/system-overview.md` - High-level system architecture
- `/docs/architecture/backend-architecture.md` - Backend service design
- `/docs/architecture/frontend-architecture.md` - Frontend application design
- `/docs/architecture/database-design.md` - Data architecture and patterns
- `/docs/architecture/security-architecture.md` - Security design and patterns
- `/docs/architecture/deployment-architecture.md` - Infrastructure and deployment
- `/docs/architecture/integration-patterns.md` - Service integration documentation
- `/docs/diagrams/` directory with Mermaid diagram source files

**Diagram Requirements:**
- System context diagram showing external dependencies
- Component diagrams for backend and frontend architecture
- Sequence diagrams for authentication and key workflows
- Entity relationship diagrams for database design
- Deployment diagrams for AWS infrastructure

## VALIDATION CRITERIA

- [ ] System overview provides clear understanding of overall architecture
- [ ] Backend architecture explains NestJS module organization and boundaries
- [ ] Frontend architecture documents React component hierarchy and state flow
- [ ] Database design documentation includes entity relationships and patterns
- [ ] Authentication flow is clearly documented with sequence diagrams
- [ ] AWS integration patterns are explained with security considerations
- [ ] Data flow diagrams show request/response patterns clearly
- [ ] Deployment architecture matches actual Terraform infrastructure
- [ ] Design decisions are explained with rationale and trade-offs
- [ ] Scalability patterns address growth from single-doctor to multi-provider
- [ ] Security architecture addresses healthcare data protection requirements
- [ ] Integration patterns enable consistent service communication
- [ ] All diagrams are version-controlled and render correctly
- [ ] Architecture documentation enables new developers to understand system design
- [ ] Performance characteristics and bottlenecks are identified and documented

# PROMPT 5: Operational Runbooks Creation

Create comprehensive operational runbooks and procedures for deployment, troubleshooting, maintenance, and monitoring of the MedFlow healthcare management system. The runbooks should provide step-by-step procedures for common operational tasks, issue resolution, and system maintenance to ensure reliable healthcare service delivery.

## PROJECT CONTEXT

**MedFlow Operational Environment:**
- AWS cloud infrastructure with ECS, RDS, Cognito, S3, CloudWatch
- Terraform-managed infrastructure with dev, staging, and production environments
- NestJS backend with PostgreSQL database
- React frontend with CDN distribution
- Healthcare application requiring high availability and data security

**Current Infrastructure:**
- Containerized applications running on AWS ECS
- PostgreSQL database on AWS RDS with automated backups
- CloudWatch monitoring and logging
- AWS Cognito for user authentication and management
- S3 for static asset storage and backups

## EPIC CONTEXT  

**EPIC-000 Project Setup and Infrastructure:**
- Monitoring and logging setup established with CloudWatch
- Infrastructure as Code enables consistent deployments
- Security configurations require operational procedures
- Development environment needs maintenance procedures

## TASK Context

**TASK-017 Documentation and Developer Guide:**
- Operational Runbooks Priority: MEDIUM
- Success Criteria: Step-by-step operational procedures for common tasks and issue resolution
- Must ensure reliable healthcare service delivery

## BUSINESS DOMAIN CONTEXT

**Healthcare Operations Requirements:**
- 99.9% uptime requirement for clinic operations
- Patient data backup and recovery procedures
- healthcare privacy-compliant data handling during maintenance
- Rapid issue resolution to minimize healthcare service disruption
- Audit trail maintenance for compliance requirements

## FUNCTIONAL REQUIREMENTS

1. **Deployment Procedures**
   - Step-by-step deployment process for each environment
   - Rollback procedures for failed deployments
   - Database migration execution and rollback
   - Environment-specific configuration management
   - Deployment verification and health checks

2. **Monitoring and Alerting**
   - CloudWatch dashboard setup and maintenance
   - Alert configuration for critical system metrics
   - Log analysis and troubleshooting procedures
   - Performance monitoring and optimization
   - Security event monitoring and response

3. **Troubleshooting Runbooks**
   - Common application errors and resolution steps
   - Database connectivity and performance issues
   - Authentication and authorization problems
   - AWS service integration issues
   - Network connectivity and DNS problems

4. **Maintenance Procedures**
   - Routine maintenance tasks and schedules
   - Database maintenance and optimization
   - Security updates and patching procedures
   - Backup verification and restoration testing
   - Performance optimization and scaling

## TECHNICAL REQUIREMENTS

**Runbook Format:**
- Step-by-step procedures with clear decision points
- Command-line examples with expected outputs
- Screenshot documentation for complex UI procedures
- Troubleshooting flowcharts for issue diagnosis
- Contact information and escalation procedures

**Operational Tools:**
- AWS CLI commands for infrastructure management
- Database maintenance scripts and procedures
- Monitoring query examples for CloudWatch
- Log analysis commands and patterns

## TESTING REQUIREMENTS

**Procedure Validation:**
- Test all deployment procedures in staging environment
- Validate troubleshooting steps resolve documented issues
- Test backup and recovery procedures with real data
- Verify monitoring and alerting configurations work correctly

**Runbook Testing:**
- Follow runbook procedures with different team members
- Test escalation procedures and contact information
- Validate command examples work in actual environments
- Confirm troubleshooting flowcharts lead to resolution

## DOCUMENTATIONS REQUIREMENTS

**Files to Create:**
- `/docs/operations/deployment-runbook.md` - Deployment procedures
- `/docs/operations/troubleshooting-guide.md` - Issue resolution procedures
- `/docs/operations/monitoring-playbook.md` - Monitoring and alerting procedures
- `/docs/operations/maintenance-procedures.md` - Routine maintenance tasks
- `/docs/operations/incident-response.md` - Emergency response procedures
- `/docs/operations/backup-recovery.md` - Data backup and recovery procedures
- `/scripts/operations/` directory with operational automation scripts

**Procedure Documentation:**
- Emergency contact information and escalation matrix
- Environment-specific configuration details
- Service dependencies and integration points
- Performance baselines and alert thresholds

## VALIDATION CRITERIA

- [ ] Deployment runbook covers all environments (dev, staging, production)
- [ ] Rollback procedures are clearly documented and tested
- [ ] Database migration procedures include safety checks and rollback steps
- [ ] Monitoring playbook covers all critical system metrics and alerts
- [ ] Troubleshooting guide addresses common application and infrastructure issues
- [ ] Backup and recovery procedures are documented and tested
- [ ] Security incident response procedures are established
- [ ] Maintenance procedures include schedules and safety considerations
- [ ] All command-line examples are tested and produce expected results
- [ ] Escalation procedures include contact information and severity levels
- [ ] Performance optimization procedures address scaling requirements
- [ ] Healthcare compliance requirements are addressed in operational procedures
- [ ] Audit trail maintenance procedures ensure regulatory compliance
- [ ] Emergency procedures prioritize patient care continuity
- [ ] All operational scripts are documented and version-controlled