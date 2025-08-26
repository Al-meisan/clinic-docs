# PROMPTS: TASK-014 - Development Environment Configuration

# PROMPT 1: ESLint and Prettier Configuration Setup

Create comprehensive ESLint and Prettier configurations optimized for the healthcare management system development, including healthcare-specific rules for medical data handling, accessibility compliance, and security best practices.

## PROJECT CONTEXT
**Tech Stack**: TypeScript with NestJS backend and React frontend, using shadcn/ui components for consistent design system. The project follows domain-driven design with feature-based folder structure.

**Code Standards**: 
- Naming conventions: camelCase for variables/functions, PascalCase for components, kebab-case for files
- Strict TypeScript configuration for type safety in medical applications
- Healthcare-specific code quality standards for patient data security
- Performance-critical code patterns for medical system responsiveness

## EPIC CONTEXT  
This is part of the **Project Setup Epic (EPIC-000)** which establishes foundational development infrastructure. This task provides the code quality foundation that all other development tasks will depend on, ensuring consistent development practices across the healthcare application.

## TASK CONTEXT
**Goal**: Configure ESLint and Prettier with healthcare-specific rules and consistent formatting that enforces medical software development standards.

**Priority**: HIGH - Critical for establishing code quality standards before feature development begins.

**Healthcare-Specific Requirements**:
- Medical data handling rules for sensitive healthcare data
- WCAG accessibility compliance enforcement  
- Performance rules for critical healthcare workflows
- Security best practices for medical applications

## TECHNICAL REQUIREMENTS

**ESLint Configuration Must Include**:
- TypeScript strict mode enforcement
- React and React Hooks rules
- Accessibility rules (@eslint-plugin/jsx-a11y)
- Security rules (eslint-plugin-security)
- Performance rules (eslint-plugin-react-perf)
- Custom rules for medical data handling patterns
- Import ordering and absolute path enforcement

**Prettier Configuration Must Include**:
- Consistent formatting across TypeScript/JSX files
- Integration with ESLint to avoid conflicts
- Healthcare documentation formatting standards
- Code organization that enhances medical code readability

**File Structure to Create**:
```
.eslintrc.js
.prettierrc
.eslintignore
.prettierignore
```

## FUNCTIONAL REQUIREMENTS

1. **Medical Data Security Rules**
   - Enforce secure patterns for patient data handling
   - Prevent logging of sensitive medical information
   - Require proper data sanitization patterns

2. **Accessibility Enforcement**
   - WCAG 2.1 AA compliance rules
   - Medical form accessibility patterns
   - Screen reader compatibility enforcement

3. **Performance Rules**
   - Optimize critical healthcare workflow code paths
   - Prevent performance anti-patterns in medical UI
   - Enforce efficient data fetching patterns

4. **Code Organization Rules**
   - Consistent import ordering (React, third-party, internal)
   - Proper component structure for medical interfaces
   - Clear separation of clinical logic and UI

## TESTING REQUIREMENTS

**Integration Tests**:
- Verify ESLint catches healthcare-specific rule violations
- Test Prettier formatting consistency across file types
- Validate configuration works with TypeScript strict mode
- Ensure no conflicts between ESLint and Prettier rules

**Manual Tests**:
- Test linting on sample medical data handling code
- Verify accessibility rules catch common violations
- Test formatting preservation of medical documentation
- Validate performance rules identify optimization opportunities

## VALIDATION CRITERIA
- [ ] ESLint configuration enforces all healthcare-specific rules
- [ ] Prettier formats TypeScript, JSX, and JSON files consistently
- [ ] No conflicts between ESLint and Prettier configurations
- [ ] Accessibility rules properly enforce WCAG 2.1 AA standards
- [ ] Security rules catch sensitive data handling violations
- [ ] Performance rules identify medical workflow bottlenecks
- [ ] Configuration files are properly documented with rule explanations
- [ ] Integration with TypeScript strict mode works without conflicts

---

# PROMPT 2: Jest Testing Framework Configuration

Configure Jest testing framework with React Testing Library and healthcare-specific testing patterns, including utilities for secure patient data mocking, clinical workflow testing, and accessibility testing automation.

## PROJECT CONTEXT
**Tech Stack**: React with TypeScript frontend and NestJS backend, requiring comprehensive testing for medical software reliability. Using feature-based architecture where each feature module contains its own tests.

**Quality Standards**: 
- Minimum 80% test coverage for business logic
- Focus on healthcare workflow testing
- Patient safety through comprehensive testing
- Medical data security testing patterns

## EPIC CONTEXT
Part of **Project Setup Epic (EPIC-000)** establishing testing foundation for healthcare application development. Testing is critical for patient safety and medical software compliance.

## TASK CONTEXT
**Goal**: Set up Jest testing framework with healthcare-specific testing utilities and patterns for reliable medical software development.

**Priority**: HIGH - Essential for patient safety and medical software quality assurance.

**Healthcare Testing Focus**:
- Secure patient data mocking patterns
- Clinical workflow testing utilities  
- Accessibility testing automation
- Medical system integration test patterns

## TECHNICAL REQUIREMENTS

**Jest Configuration Must Include**:
- TypeScript support with ts-jest
- React Testing Library setup
- Test environment configuration for medical workflows
- Coverage reporting with healthcare-specific thresholds
- Mock utilities for medical data and external services
- Custom matchers for healthcare-specific assertions

**Testing Utilities to Create**:
- Patient data mocking utilities (secure, HIPAA-compliant)
- Clinical workflow testing helpers
- Accessibility testing automation
- Medical form testing utilities
- API mocking for healthcare services

**File Structure to Create**:
```
jest.config.js
setupTests.ts
src/
├── utils/
│   └── test-utils/
│       ├── index.ts
│       ├── patient-mocks.ts
│       ├── clinical-test-utils.ts
│       ├── accessibility-utils.ts
│       └── api-mocks.ts
```

## FUNCTIONAL REQUIREMENTS

1. **Patient Data Mocking System**
   - Generate realistic but fake patient data
   - Ensure no real patient information in test data
   - Support various medical scenarios and edge cases
   - HIPAA-compliant data generation patterns

2. **Clinical Workflow Testing**
   - Test appointment scheduling workflows
   - Validate prescription management processes
   - Test clinical documentation workflows
   - Verify billing and payment processes

3. **Accessibility Testing Automation**
   - Automated WCAG compliance testing
   - Screen reader compatibility verification
   - Keyboard navigation testing
   - Medical form accessibility validation

4. **Healthcare Integration Testing**
   - Mock external medical systems
   - Test clinical data synchronization
   - Validate healthcare API integrations
   - Test medical device connectivity patterns

## TESTING REQUIREMENTS

**Unit Tests**:
- Test healthcare-specific utility functions
- Validate patient data mocking generates appropriate data
- Test clinical workflow helper functions
- Verify accessibility testing utilities work correctly

**Integration Tests**:
- Test complete Jest setup with sample healthcare components
- Validate testing utilities work with real healthcare scenarios
- Test coverage reporting captures medical workflow coverage
- Ensure mock data utilities integrate with test framework

## DOCUMENTATION REQUIREMENTS

**Create Documentation**:
- Healthcare testing patterns guide
- Patient data mocking best practices
- Clinical workflow testing examples
- Accessibility testing automation guide

## VALIDATION CRITERIA
- [ ] Jest configuration supports TypeScript and React Testing Library
- [ ] Patient data mocking utilities generate secure, realistic test data
- [ ] Clinical workflow testing helpers support common medical scenarios
- [ ] Accessibility testing automation catches WCAG violations
- [ ] Coverage reporting provides meaningful insights for healthcare code
- [ ] Testing utilities are well-documented with medical examples
- [ ] Integration with existing TypeScript configuration works seamlessly
- [ ] Healthcare-specific test patterns are established and documented

---

# PROMPT 3: Git Hooks and Development Automation

Implement pre-commit hooks, commit message validation, and development automation scripts to enforce code quality gates and streamline healthcare application development workflow.

## PROJECT CONTEXT
**Development Workflow**: Team-based development with strict quality requirements for healthcare software. Code quality is critical for patient safety and regulatory compliance.

**Quality Gates**: Automated enforcement of coding standards, testing requirements, and security patterns before code reaches the repository.

## EPIC CONTEXT
Final component of **Project Setup Epic (EPIC-000)** development environment, providing automated quality enforcement that prevents low-quality code from entering the healthcare codebase.

## TASK CONTEXT
**Goal**: Implement Git hooks and development scripts that automatically enforce healthcare code quality standards and prevent commits that don't meet medical software requirements.

**Priority**: MEDIUM - Important for maintaining code quality consistency across development team.

**Automation Focus**:
- Pre-commit quality validation
- Commit message standardization
- Automated testing enforcement
- Development workflow optimization

## TECHNICAL REQUIREMENTS

**Git Hooks to Implement**:
- Pre-commit hook with ESLint, Prettier, and testing
- Commit message validation for conventional commits
- Pre-push hook for final quality validation
- Post-merge hook for dependency updates

**Development Scripts to Create**:
- Setup script for new developer onboarding
- Quality check script for full codebase validation
- Test runner scripts for different testing scenarios
- Build verification scripts

**Tools to Configure**:
- Husky for Git hooks management
- lint-staged for selective file processing
- commitizen for standardized commit messages
- concurrently for parallel script execution

## FUNCTIONAL REQUIREMENTS

1. **Pre-commit Quality Gates**
   - Run ESLint on staged files with healthcare rules
   - Execute Prettier formatting on staged files
   - Run relevant tests for changed code
   - Validate TypeScript compilation
   - Check for healthcare-specific security patterns

2. **Commit Message Enforcement**
   - Enforce conventional commit format
   - Require specific scopes for medical features
   - Validate commit message clarity and completeness
   - Support healthcare-specific commit types

3. **Development Workflow Scripts**
   - New developer environment setup
   - Full codebase quality validation
   - Healthcare-specific testing scenarios
   - Build and deployment preparation

4. **Team Collaboration Tools**
   - Shared development environment configuration
   - Consistent tooling across team members
   - Automated dependency management
   - Development status reporting

## TESTING REQUIREMENTS

**Integration Tests**:
- Verify Git hooks prevent commits with quality violations
- Test commit message validation with various formats
- Validate development scripts work across different environments
- Test new developer setup process end-to-end

**Manual Tests**:
- Simulate commits with ESLint violations (should be blocked)
- Test commits with failing tests (should be blocked)
- Verify proper commit message formats are accepted
- Test development script execution on fresh environment

## DOCUMENTATION REQUIREMENTS

**Create Documentation**:
- Git workflow guide for healthcare development
- Commit message standards and examples
- Development script usage instructions
- Troubleshooting guide for common setup issues

## VALIDATION CRITERIA
- [ ] Pre-commit hooks successfully prevent low-quality code commits
- [ ] Commit message validation enforces healthcare development standards
- [ ] Development scripts automate common workflow tasks effectively
- [ ] Git hooks work consistently across different operating systems
- [ ] New developer setup process is streamlined and well-documented
- [ ] Quality gates provide clear feedback on violations
- [ ] Healthcare-specific development patterns are enforced
- [ ] Team can adopt the workflow without significant friction

---

# PROMPT 4: Development Environment Documentation and Validation

Create comprehensive documentation for the development environment setup and validate the complete development toolchain through systematic testing with healthcare-specific scenarios.

## PROJECT CONTEXT
**Documentation Standards**: Clear, actionable documentation that enables new team members to set up and use the healthcare development environment efficiently.

**Validation Requirements**: Systematic testing of all development tools to ensure they work together seamlessly for healthcare application development.

## EPIC CONTEXT
Completion deliverable for **Project Setup Epic (EPIC-000)**, ensuring the development environment is fully documented and validated for healthcare software development team adoption.

## TASK CONTEXT
**Goal**: Document the complete development environment setup and validate all tools work together effectively for healthcare application development.

**Priority**: MEDIUM - Important for team adoption and long-term maintainability of development practices.

**Focus Areas**:
- New developer onboarding documentation
- Development tool usage guides  
- Healthcare-specific development patterns
- Environment validation and troubleshooting

## FUNCTIONAL REQUIREMENTS

1. **Comprehensive Setup Documentation**
   - Step-by-step development environment setup
   - Tool configuration explanations
   - Healthcare-specific development patterns
   - Common troubleshooting scenarios

2. **Development Workflow Guides**
   - Daily development workflow with quality tools
   - Healthcare code review processes
   - Testing strategies for medical software
   - Deployment preparation procedures

3. **Environment Validation Suite**
   - Automated validation of development tool setup
   - Healthcare-specific code quality verification
   - Testing framework validation
   - Integration verification across tool chain

4. **Team Onboarding Materials**
   - New developer checklist
   - Healthcare development best practices
   - Tool usage examples and tutorials
   - Development environment troubleshooting

## TECHNICAL REQUIREMENTS

**Documentation to Create**:
```
docs/
├── development/
│   ├── environment-setup.md
│   ├── code-quality-tools.md
│   ├── testing-guide.md
│   ├── git-workflow.md
│   └── troubleshooting.md
├── healthcare-patterns/
│   ├── medical-data-handling.md
│   ├── accessibility-requirements.md
│   ├── security-patterns.md
│   └── performance-optimization.md
└── onboarding/
    ├── new-developer-guide.md
    ├── development-checklist.md
    └── tool-tutorials.md
```

**Validation Scripts to Create**:
- Environment health check script
- Development tool integration verification
- Healthcare code pattern validation
- Complete workflow end-to-end testing

## TESTING REQUIREMENTS

**Integration Tests**:
- Complete new developer setup process validation
- End-to-end development workflow testing
- Healthcare-specific development scenario testing
- Cross-platform development environment validation

**Manual Tests**:
- New developer follows documentation to set up environment
- Healthcare code patterns work as documented
- Quality tools function as described in guides
- Troubleshooting documentation resolves common issues

**Automation Tests**:
- Environment health check executes successfully
- Development tool chain integration verified
- Healthcare-specific patterns validated automatically
- Documentation accuracy verified through automated testing

## DOCUMENTATION REQUIREMENTS

**Primary Documentation**:
- **Environment Setup Guide**: Complete instructions for development environment configuration
- **Healthcare Development Patterns**: Medical software-specific coding patterns and practices
- **Tool Usage Guides**: Detailed instructions for ESLint, Prettier, Jest, and Git hooks
- **Troubleshooting Guide**: Solutions for common development environment issues

**Supporting Materials**:
- **New Developer Checklist**: Step-by-step onboarding process
- **Code Quality Examples**: Examples of proper healthcare code patterns
- **Testing Scenarios**: Healthcare-specific testing examples and patterns
- **Team Standards**: Agreed-upon development practices and conventions

## VALIDATION CRITERIA
- [ ] New developer can set up environment in under 30 minutes following documentation
- [ ] All development tools function correctly with healthcare-specific configurations
- [ ] Documentation covers all healthcare development patterns and requirements
- [ ] Environment validation scripts verify correct setup automatically
- [ ] Troubleshooting guide resolves common setup and usage issues
- [ ] Healthcare code quality patterns are clearly documented with examples
- [ ] Development workflow documentation supports efficient team collaboration
- [ ] All tool integrations work seamlessly for healthcare application development