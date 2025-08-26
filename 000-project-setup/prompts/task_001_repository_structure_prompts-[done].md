# PROMPTS: TASK-001 - Repository Structure Setup

# PROMPT 1: Create Clinic-Backend Repository Foundation (

Create the initial folder structure, configuration files, and basic setup for the clinic-backend repository following NestJS and TypeScript standards with proper gitignore and environment configuration.

## PROJECT CONTEXT
**Tech Stack**: TypeScript with NestJS for backend development, domain-driven design with feature modules, PostgreSQL database with TypeORM, AWS infrastructure deployment.

**Architecture Patterns**: 
- Feature-based folder structure under `src/features/`
- Shared modules for cross-cutting concerns in `src/shared/`
- Infrastructure layer for external systems in `src/infrastructure/`
- Configuration management with environment variables and validation

## EPIC CONTEXT
**Business Goal**: Establish foundational repository structure and development standards for the MedFlow project enabling efficient team collaboration and standardized development practices across all repositories.

## TASK CONTEXT
**Primary Objective**: Create clinic-backend repository with standardized folder structure, development tool configuration, and environment setup that enables new developers to clone and run basic setup within 30 minutes.

**Repository Naming**: Follow `clinic-backend` naming convention with lowercase and hyphens.

## TECHNICAL REQUIREMENTS
**Backend Folder Structure**: 
```
clinic-backend/src/
                ├── main.ts                    # Application entry point
                ├── app.module.ts              # Root module
                ├── common/                    # Reusable code across app
                ├── config/                    # Configuration files
                ├── features/                  # Feature modules
                ├── shared/                    # Shared modules/services
                └── infrastructure/            # External systems
```

**Configuration Files Required**:
- `.gitignore` with Node.js and NestJS specific exclusions
- `.env.example` template with all required environment variables
- `package.json` with NestJS dependencies and proper scripts
- `tsconfig.json` configured for NestJS development
- `docker-compose.yml` for local development environment

## FUNCTIONAL REQUIREMENTS
- Repository structure must support domain-driven design approach
- All configuration files must be present and properly configured
- Environment variables must be documented in .env.example
- Docker compose setup for local PostgreSQL development

## VALIDATION CRITERIA
- [ ] Repository contains proper folder structure with all required directories
- [ ] .gitignore excludes node_modules, .env, dist, and other build artifacts
- [ ] .env.example contains all necessary environment variables for development
- [ ] package.json includes correct NestJS dependencies and development scripts
- [ ] tsconfig.json configured for TypeScript with proper paths and options
- [ ] docker-compose.yml provides PostgreSQL service for local development

---

# PROMPT 2: Create Clinic-Portal Repository Foundation

Create the initial folder structure, configuration files, and basic setup for the clinic-portal repository following React, TypeScript, and Vite standards with feature-based architecture.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript for frontend, shadcn/ui for UI components, Vite for build tool, feature-based folder structure with domain-driven approach.

**Architecture Patterns**:
- Feature-based modules under `src/features/`
- Shared UI components in `src/components/`
- Core utilities and services in `src/lib/`
- Routing configuration in `src/routes/`

## EPIC CONTEXT
**Business Goal**: Establish foundational repository structure and development standards for the MedFlow project enabling efficient team collaboration and standardized development practices across all repositories.

## TASK CONTEXT
**Primary Objective**: Create clinic-portal repository with React and TypeScript foundation, Vite build system, and feature-based architecture that supports rapid frontend development.

**Repository Naming**: Follow `clinic-portal` naming convention with lowercase and hyphens.

## TECHNICAL REQUIREMENTS
**Frontend Folder Structure**:
```
clinic-portal/src/
              ├── assets/                    # Static assets
              ├── components/                # Shared UI components
              ├── config/                    # Configuration files
              ├── hooks/                     # Shared custom hooks
              ├── lib/                       # Core utilities and services
              ├── providers/                 # Context providers
              ├── routes/                    # Routing setup
              ├── features/                  # Feature-based modules
              └── types/                     # Global TypeScript types
```

**Configuration Files Required**:
- `vite.config.ts` with React plugin and path aliases
- `tsconfig.json` configured for React and strict TypeScript
- `.gitignore` with React and Vite specific exclusions
- `.env.example` with frontend environment variables template

## FUNCTIONAL REQUIREMENTS
- Repository structure must support feature-based development
- Vite configuration must include path aliases for easy imports
- TypeScript configuration must enforce strict typing
- Environment variable template for API endpoints and configuration

## VALIDATION CRITERIA
- [ ] Repository contains proper feature-based folder structure
- [ ] vite.config.ts includes React plugin and path aliases (@, @/components, @/features, etc.)
- [ ] tsconfig.json configured for React with strict TypeScript settings
- [ ] .gitignore excludes node_modules, dist, .env, and Vite build artifacts
- [ ] .env.example contains API_BASE_URL and other necessary frontend variables
- [ ] package.json includes React, TypeScript, Vite, and necessary development dependencies

---

# PROMPT 3: Create Clinic-Infra Repository Foundation

Create the initial folder structure and configuration files for the clinic-infra repository following Terraform best practices and Infrastructure as Code standards for AWS deployment.

## PROJECT CONTEXT
**Tech Stack**: Terraform for Infrastructure as Code, AWS as cloud provider, cost-effective architecture approach, environment-specific configurations for dev/staging/prod.

**Architecture Patterns**:
- Environment-based folder structure for different deployment stages
- Reusable Terraform modules for common infrastructure patterns
- Centralized variable management and state configuration

## EPIC CONTEXT
**Business Goal**: Establish foundational repository structure and development standards for the MedFlow project enabling efficient team collaboration and standardized development practices across all repositories.

## TASK CONTEXT
**Primary Objective**: Create clinic-infra repository with Terraform modules, environment-specific configurations, and Infrastructure as Code structure that supports AWS deployment across multiple environments.

**Repository Naming**: Follow `clinic-infra` naming convention with lowercase and hyphens.

## TECHNICAL REQUIREMENTS
**Infrastructure Folder Structure**:
```
├── README.md
├── .gitignore
├── environments/              # Environment-specific configs
│   ├── dev/
│   ├── staging/
│   └── prod/
├── modules/                   # Reusable Terraform modules
└── scripts/                   # Deployment and utility scripts
```

**Configuration Files Required**:
- `.gitignore` with Terraform-specific exclusions (state files, .terraform/, etc.)
- Environment folders with main.tf, variables.tf, outputs.tf structure
- Module templates for common infrastructure patterns

## FUNCTIONAL REQUIREMENTS
- Repository structure must support multiple environment deployments
- Terraform modules must be reusable across different environments
- State management and variable configuration must be environment-specific
- Scripts for common deployment tasks

## VALIDATION CRITERIA
- [ ] Repository contains environment folders (dev, staging, prod) with proper Terraform structure
- [ ] .gitignore excludes .terraform/, *.tfstate, *.tfvars, and other Terraform artifacts
- [ ] modules/ directory prepared for reusable infrastructure components
- [ ] scripts/ directory contains deployment and utility scripts
- [ ] Each environment folder has placeholder main.tf, variables.tf, and outputs.tf files
- [ ] README.md includes basic Terraform usage instructions and prerequisites

---

# PROMPT 4: Create Comprehensive Repository Documentation

Create detailed README.md files for all three repositories (clinic-backend, clinic-portal, clinic-infra) with setup instructions, development workflows, and environment configuration that enables new developers to get started within 30 minutes.

## PROJECT CONTEXT
**Development Standards**: New developer onboarding must be completed within 30 minutes, documentation must include prerequisites, setup instructions, development workflow, and deployment instructions using markdown formatting without emoji.

**Cross-Platform Support**: Documentation must support Windows, macOS, and Linux development environments with environment-specific instructions where necessary.

## EPIC CONTEXT
**Business Goal**: Establish foundational repository structure and development standards for the MedFlow project enabling efficient team collaboration and standardized development practices across all repositories.

## TASK CONTEXT
**Primary Objective**: Create comprehensive README documentation that allows any new team member to clone repositories and get development environment running within 30 minutes.

**Documentation Standards**: Use markdown formatting with proper headers and code blocks, document all environment variables and configuration options, include troubleshooting sections.

## FUNCTIONAL REQUIREMENTS
**README Structure for Each Repository**:
- Project description and overview
- Prerequisites and system requirements
- Step-by-step setup instructions
- Development workflow and common commands
- Environment variable documentation
- Deployment instructions
- Troubleshooting section

## TECHNICAL REQUIREMENTS
**Backend README (clinic-backend)**:
- Node.js and npm version requirements
- PostgreSQL setup instructions (local and Docker)
- Environment variables configuration
- Database migration commands
- Development server startup
- API testing instructions

**Frontend README (clinic-portal)**:
- Node.js and npm version requirements
- Vite development server setup
- Environment variables for API endpoints
- Build and preview commands
- Component development guidelines

**Infrastructure README (clinic-infra)**:
- Terraform installation requirements
- AWS CLI configuration steps
- Environment setup for dev/staging/prod
- Deployment commands and workflow
- State management instructions

## VALIDATION CRITERIA
- [ ] Each repository has comprehensive README.md with all required sections
- [ ] Prerequisites section lists exact version requirements for all tools
- [ ] Setup instructions are step-by-step and testable by new developers
- [ ] Environment variables are fully documented with example values
- [ ] Development workflow includes common commands and debugging tips
- [ ] Troubleshooting section addresses common setup issues
- [ ] All code blocks are properly formatted with language specification

---

# PROMPT 5: Configure Development Tools and Standards

Set up ESLint, Prettier, and development tool configurations across all three repositories with consistent code quality standards, conventional commit message format, and development workflow automation.

## PROJECT CONTEXT
**Code Quality Standards**: ESLint and Prettier for consistent code formatting and quality enforcement, conventional commit messages with format `<type>(<scope>): <description>`, shared configuration across repositories for consistency.

**Development Workflow**: Git hooks for pre-commit validation, automated code formatting, and commit message validation to enforce standards across all repositories.

## EPIC CONTEXT
**Business Goal**: Establish foundational repository structure and development standards for the MedFlow project enabling efficient team collaboration and standardized development practices across all repositories.

## TASK CONTEXT
**Primary Objective**: Configure development tools (ESLint, Prettier) and Git standards that enforce code quality and consistency across all three repositories with automated validation.

**Configuration Consistency**: All repositories should share similar ESLint and Prettier configurations with technology-specific adjustments (React rules for frontend, NestJS rules for backend).

## TECHNICAL REQUIREMENTS
**ESLint Configuration**:
- TypeScript-specific rules for all repositories
- React-specific rules for clinic-portal
- NestJS-specific rules for clinic-backend
- Shared base configuration for consistency

**Prettier Configuration**:
- Consistent formatting rules across all repositories
- Integration with ESLint to avoid conflicts
- Automatic formatting on save configuration

**Git Configuration**:
- Pre-commit hooks for code quality validation
- Commit message format validation
- Branch naming conventions
- Git ignore patterns specific to each technology

## FUNCTIONAL REQUIREMENTS
**Backend Development Tools (clinic-backend)**:
- ESLint with @typescript-eslint and @nestjs/eslint-plugin
- Prettier integration with ESLint
- Pre-commit hooks for TypeScript and test validation
- Package.json scripts for linting and formatting

**Frontend Development Tools (clinic-portal)**:
- ESLint with @typescript-eslint and React rules
- Prettier with React-specific formatting
- Pre-commit hooks for TypeScript and component validation
- Vite integration with linting during development

**Infrastructure Development Tools (clinic-infra)**:
- Terraform formatting with `terraform fmt`
- Validation scripts for Terraform configuration
- Pre-commit hooks for infrastructure code quality
- Documentation validation scripts

## VALIDATION CRITERIA
- [ ] All repositories have properly configured .eslintrc.js files
- [ ] All repositories have consistent .prettierrc configuration
- [ ] ESLint rules are appropriate for each technology stack
- [ ] Prettier and ESLint configurations don't conflict
- [ ] Package.json scripts include lint, format, and validation commands
- [ ] Pre-commit hooks prevent commits that fail quality checks
- [ ] Commit message format is validated and enforced
- [ ] Development servers integrate linting feedback during development
