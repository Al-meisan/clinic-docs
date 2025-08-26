# PROMPTS: TASK-002 - Terraform Infrastructure Base

# ~~PROMPT 1: VPC Foundation Module Creation~~

Create a Terraform VPC module that establishes the foundational networking infrastructure for the MedFlow healthcare
management system with proper subnet segmentation, gateway configuration, and multi-AZ deployment.

## PROJECT CONTEXT

The MedFlow healthcare management system requires secure, scalable AWS infrastructure foundation using Terraform for
Infrastructure as Code. The system must support both single-doctor and multi-provider clinic operations with
cost-effective deployments optimized for the Algerian market. The infrastructure will serve as the foundation for RDS
databases, ECS services, Cognito authentication, and S3 storage.

## EPIC CONTEXT

This task is part of the Project Setup Epic focusing on establishing core infrastructure foundations that enable all
subsequent development work. The infrastructure must be reproducible across environments and follow security-first
design principles.

## TASK CONTEXT

**Task Goal:** Create foundational AWS infrastructure using Terraform modules for VPC, security groups, and basic
networking components that will support the MedFlow healthcare management system.

**Priority Requirements:**

- VPC with proper CIDR blocks and subnet configuration across multiple availability zones
- Internet Gateway and NAT Gateway configuration for public/private subnet routing
- Multi-environment support (dev, staging, production)
- Infrastructure provisioning time under 15 minutes

## BUSINESS DOMAIN CONTEXT

**Healthcare Data Security:** The infrastructure must comply with healthcare data protection requirements, ensuring
patient data isolation and secure network communications.

**Cost Optimization:** Infrastructure design must be cost-effective for small clinics while maintaining scalability for
multi-provider operations.

## TECHNICAL REQUIREMENTS

**VPC Module Specifications:**

- Create VPC with configurable CIDR blocks (default: 10.0.0.0/16)
- Public subnets (2) across different AZs for load balancers: 10.0.1.0/24, 10.0.2.0/24
- Private subnets (2) across different AZs for application servers: 10.0.10.0/24, 10.0.11.0/24
- Database subnets (2) across different AZs for RDS: 10.0.20.0/24, 10.0.21.0/24
- Internet Gateway for public subnet internet access
- NAT Gateway in each AZ for private subnet outbound internet access
- Route tables properly configured for each subnet tier

**File Structure:**

```
infrastructure/
├── modules/
│   └── vpc/
│       ├── main.tf          # VPC, subnets, gateways, route tables
│       ├── variables.tf     # Input parameters with validation
│       ├── outputs.tf       # VPC ID, subnet IDs, security group IDs
│       └── README.md        # Module documentation
```

**Resource Naming Convention:**
Format: `{environment}-medflow-{resource-type}-{description}`
Example: `dev-medflow-vpc`, `staging-medflow-public-subnet-1a`

**Required Outputs:**

- vpc_id
- public_subnet_ids (list)
- private_subnet_ids (list)
- database_subnet_ids (list)
- internet_gateway_id
- nat_gateway_ids (list)

## FUNCTIONAL REQUIREMENTS

1. **Multi-AZ Deployment**
    - Deploy subnets across at least 2 availability zones
    - Ensure high availability for database and application tiers
    - Configure proper routing between zones

2. **Network Segmentation**
    - Public subnets for internet-facing resources (load balancers)
    - Private subnets for application servers (no direct internet access)
    - Database subnets for RDS instances (most restricted access)

3. **Variable Configuration**
    - Environment-specific CIDR block configuration
    - Availability zone selection based on region
    - Optional features (single NAT for cost optimization in dev)

## TESTING REQUIREMENTS

**Validation Tests:**

- Terraform plan execution without errors
- Resource creation and proper tagging validation
- Network connectivity testing between subnet tiers
- CIDR block non-overlap validation
- AZ distribution verification

**Integration Tests:**

- VPC endpoint accessibility testing
- Route table configuration validation
- Security group attachment verification

## DOCUMENTATION REQUIREMENTS

**Module README.md:**

- Purpose and architecture overview
- Usage examples for each environment
- Variable descriptions and default values
- Output descriptions and usage patterns
- Network architecture diagram (ASCII or reference to external diagram)

## VALIDATION CRITERIA

- [ ] VPC created with configurable CIDR blocks and proper DNS settings
- [ ] Public subnets (2) created across different AZs with internet gateway routing
- [ ] Private subnets (2) created across different AZs with NAT gateway routing
- [ ] Database subnets (2) created across different AZs with no internet routing
- [ ] Internet Gateway attached to VPC and configured in public route tables
- [ ] NAT Gateways deployed in public subnets with proper EIP allocation
- [ ] All resources follow naming convention and include required tags
- [ ] Module outputs provide all necessary IDs for dependent modules
- [ ] Terraform validation passes without warnings or errors
- [ ] Module documentation includes usage examples and architecture description

# ~~PROMPT 2: Security Groups Module Creation~~

Create a Terraform security groups module that implements multi-tier security architecture following the principle of
least privilege for web, application, and database tiers of the MedFlow healthcare management system.

## PROJECT CONTEXT

The MedFlow healthcare management system processes sensitive patient health information and requires robust network
security controls. The multi-tier architecture needs properly configured security groups to ensure secure communication
between tiers while preventing unauthorized access. The system must support both single-doctor and multi-provider clinic
configurations.

## EPIC CONTEXT

This security groups module is part of the foundational infrastructure setup that enables secure deployment of
application components. It establishes the security boundaries that will protect patient data and ensure HIPAA-compliant
network security.

## TASK CONTEXT

**Task Goal:** Define security groups for web tier, application tier, and database tier with minimal required access
following least privilege principle.

**Security Priority:** All security group rules must be documented and follow healthcare data protection requirements
with explicit allow-lists rather than broad access patterns.

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance:** Security groups must support HIPAA compliance requirements by implementing network-level
access controls and logging capabilities.

**Multi-Tenancy Security:** The infrastructure must isolate different clinic deployments while allowing controlled
shared services access.

## TECHNICAL REQUIREMENTS

**Security Group Architecture:**

- **Web Tier Security Group:** Internet-facing load balancers and CDN endpoints
- **Application Tier Security Group:** NestJS backend services and internal APIs
- **Database Tier Security Group:** RDS instances and data storage services

**File Structure:**

```
infrastructure/
├── modules/
│   └── security-groups/
│       ├── main.tf          # Security group definitions and rules
│       ├── variables.tf     # Port configurations and environment settings
│       ├── outputs.tf       # Security group IDs
│       └── README.md        # Security documentation
```

**Security Rules Specifications:**

*Web Tier (ALB/CloudFront):*

- Inbound: HTTP (80), HTTPS (443) from 0.0.0.0/0
- Outbound: HTTP (80), HTTPS (443) to application tier only

*Application Tier (ECS/EC2):*

- Inbound: HTTP (3000, 8080) from web tier security group only
- Outbound: HTTPS (443) for external APIs, database port to database tier

*Database Tier (RDS):*

- Inbound: PostgreSQL (5432) from application tier security group only
- No outbound rules (default deny all)

## FUNCTIONAL REQUIREMENTS

1. **Least Privilege Implementation**
    - No security group allows broader access than required
    - Source/destination restrictions use security group references, not CIDR blocks where possible
    - All rules include descriptions explaining their purpose

2. **Environment-Specific Configuration**
    - Development: May include additional debug ports (22 for SSH debugging)
    - Staging: Production-like but with monitoring access
    - Production: Minimal required access only

3. **Logging and Monitoring**
    - VPC Flow Logs enabled for security group analysis
    - CloudWatch integration for security event monitoring

## TESTING REQUIREMENTS

**Security Validation Tests:**

- Port scanning validation (only allowed ports accessible)
- Cross-tier connectivity testing (application to database)
- External access validation (web tier internet accessibility)
- Unauthorized access prevention testing

**Compliance Tests:**

- Security group rules audit for least privilege compliance
- Network isolation testing between environments
- VPC Flow Logs functionality verification

## DOCUMENTATION REQUIREMENTS

**Security Documentation:**

- Security group purpose and tier assignment
- Network flow diagrams showing allowed communications
- Compliance mapping to healthcare security requirements
- Incident response procedures for security group modifications

## VALIDATION CRITERIA

- [ ] Web tier security group allows HTTP/HTTPS from internet, restricts outbound to app tier
- [ ] Application tier security group allows access from web tier only, restricts outbound appropriately
- [ ] Database tier security group allows access from app tier only, no outbound rules
- [ ] All security group rules include descriptive names and purposes
- [ ] No security groups allow unrestricted (0.0.0.0/0) access except web tier HTTP/HTTPS
- [ ] Security group references used instead of CIDR blocks for internal communications
- [ ] Environment-specific rule variations properly implemented
- [ ] VPC Flow Logs configured for security monitoring
- [ ] All security groups follow naming convention with environment prefix
- [ ] Module outputs provide security group IDs for dependent resources

# ~~PROMPT 3: Multi-Environment Configuration Setup~~

Review the current terraform multienvironment configuration and only if necessary Create Terraform environment
configuration structure that supports independent deployment of dev, staging, and
production environments with parameterized configurations and proper state isolation for the MedFlow infrastructure.

## PROJECT CONTEXT

The MedFlow healthcare management system requires consistent infrastructure deployment across multiple environments
while maintaining proper isolation and environment-specific configurations. The system must support both single-doctor
and multi-provider clinic modes with cost optimization for smaller deployments.

## EPIC CONTEXT

This multienvironment setup enables the development lifecycle by providing isolated environments for development,
testing, and production deployment. It establishes the foundation for CI/CD pipeline integration and ensures consistent
infrastructure patterns.

## TASK CONTEXT

**Task Goal:** Support for dev, staging, and production environments with parameterized configurations where each
environment can be deployed independently with appropriate resource sizing.

**Environment Requirements:**

- Development: Cost-optimized, single AZ, minimal resources
- Staging: Production-like, reduced capacity, full feature testing
- Production: High availability, multi-AZ, full monitoring

## BUSINESS DOMAIN CONTEXT

**Cost Management:** Development environment must minimize costs for small clinic development teams while staging and
production provide appropriate service levels for healthcare operations.

**Data Isolation:** Each environment must maintain complete data isolation to prevent healthcare information from being
exposed across environments.

## TECHNICAL REQUIREMENTS

**Directory Structure:**

```
infrastructure/
├── environments/
│   ├── dev/
│   │   ├── main.tf              # Dev-specific resource definitions
│   │   ├── variables.tf         # Dev variable declarations
│   │   ├── terraform.tfvars     # Dev variable values
│   │   └── outputs.tf           # Dev outputs
│   ├── staging/
│   │   ├── main.tf              # Staging-specific resources
│   │   ├── variables.tf         # Staging variable declarations  
│   │   ├── terraform.tfvars     # Staging variable values
│   │   └── outputs.tf           # Staging outputs
│   └── prod/
│       ├── main.tf              # Production resources
│       ├── variables.tf         # Production variable declarations
│       ├── terraform.tfvars     # Production variable values
│       └── outputs.tf           # Production outputs
├── modules/                     # Shared modules (vpc, security-groups)
└── shared/
    ├── backend.tf              # Remote state configuration
    └── variables.tf            # Common variable definitions
```

**Environment-Specific Configurations:**

*Development Environment:*

- VPC CIDR: 10.0.0.0/16
- Single NAT Gateway (cost optimization)
- Smaller instance sizes
- No multi-AZ deployment for cost savings
- Development-specific tags

*Staging Environment:*

- VPC CIDR: 10.1.0.0/16
- Multi-AZ deployment
- Production-like sizing but reduced capacity
- Staging-specific monitoring
- Blue-green deployment testing

*Production Environment:*

- VPC CIDR: 10.2.0.0/16
- Full high availability (multi-AZ)
- Production instance sizes
- Comprehensive monitoring and alerting
- Backup and disaster recovery

## FUNCTIONAL REQUIREMENTS

1. **State Isolation**
    - Each environment maintains separate Terraform state
    - Remote state backend configuration (S3 + DynamoDB)
    - State file encryption and access controls

2. **Variable Management**
    - Common variables defined in shared location
    - Environment-specific values in terraform.tfvars
    - Sensitive variables managed through AWS Secrets Manager references

3. **Resource Naming and Tagging**
    - Environment prefix in all resource names
    - Consistent tagging strategy across environments
    - Cost allocation tags for billing management

## TESTING REQUIREMENTS

**Environment Deployment Tests:**

- Independent environment creation and destruction
- Cross-environment isolation verification
- Resource naming and tagging validation
- State file isolation and backend configuration testing

**Configuration Tests:**

- Variable validation across environments
- Resource sizing verification per environment
- Network isolation testing between environments

## DOCUMENTATION REQUIREMENTS

**Environment Documentation:**

- Environment-specific configuration guide
- Deployment procedures for each environment
- Resource sizing rationale and cost estimates
- Environment promotion procedures (dev -> staging -> prod)

## VALIDATION CRITERIA

- [ ] Three separate environment directories with complete configurations
- [ ] Each environment has isolated Terraform state with remote backend
- [ ] Environment-specific variable files with appropriate resource sizing
- [ ] Common variables defined in shared location to avoid duplication
- [ ] All resources include environment-specific naming and tagging
- [ ] Dev environment optimized for cost (single AZ, smaller instances)
- [ ] Staging environment configured for production-like testing
- [ ] Production environment configured for high availability and monitoring
- [ ] Backend state configuration supports team collaboration
- [ ] Documentation includes deployment procedures for each environment

Execute this prompt with the least amount of changes by reviewing the current multi env configuration in terrafrom and
doing only the necessary changes to achieve the missing criteria

# ~~PROMPT 4: Terraform State Management and Backend Configuration~~

Review the current terraform code and if necessary Create Terraform backend configuration for secure, collaborative
state management with S3 storage, DynamoDB locking, and proper access controls to support team development workflows for
the MedFlow infrastructure.

## PROJECT CONTEXT

The MedFlow healthcare management system requires collaborative infrastructure management with secure state handling
that supports multiple developers and CI/CD pipelines. The system processes sensitive healthcare data requiring audit
trails and secure access controls for all infrastructure changes.

## EPIC CONTEXT

This backend configuration enables team collaboration and establishes the foundation for CI/CD pipeline integration. It
ensures infrastructure state consistency and provides audit trails required for healthcare compliance.

## TASK CONTEXT

**Task Goal:** Configure Terraform backend for state management that supports team collaboration with S3 backend,
DynamoDB locking, and proper encryption.

**Critical Requirements:**

- Secure state file storage with encryption
- State locking to prevent concurrent modifications
- Access controls for infrastructure management
- Audit logging for compliance requirements

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance:** Infrastructure state management must provide audit trails and secure access controls to
support healthcare data protection requirements and regulatory compliance.

**Team Collaboration:** Multiple developers and CI/CD systems need coordinated access to infrastructure state without
conflicts or security issues.

## TECHNICAL REQUIREMENTS

**Backend Configuration Components:**

- S3 bucket for state storage with versioning and encryption
- DynamoDB table for state locking
- IAM roles and policies for access control
- CloudTrail logging for audit requirements

**S3 Backend Configuration:**

- Bucket naming: `medflow-terraform-state-{environment}-{region}`
- Versioning enabled for state history
- Server-side encryption with AWS KMS
- Public access blocked
- Lifecycle policies for cost optimization

**DynamoDB Locking:**

- Table naming: `medflow-terraform-locks-{environment}`
- Primary key: `LockID` (String)
- Point-in-time recovery enabled
- Server-side encryption enabled

## FUNCTIONAL REQUIREMENTS

1. **State Security**
    - State files encrypted at rest and in transit
    - Access restricted to authorized users and CI/CD systems
    - Audit logging of all state access and modifications

2. **Multi-Environment Support**
    - Separate state storage per environment
    - Environment-specific access controls
    - Consistent naming and configuration patterns

3. **Disaster Recovery**
    - State file versioning for rollback capability
    - Cross-region backup for production environments
    - Documented recovery procedures

## TESTING REQUIREMENTS

**Backend Functionality Tests:**

- State locking mechanism validation
- Concurrent access prevention testing
- State encryption verification
- Access control policy testing

**Disaster Recovery Tests:**

- State file recovery from backups
- Cross-region restoration testing
- Version rollback functionality

## DOCUMENTATION REQUIREMENTS

**Backend Setup Guide:**

- Initial bootstrap procedure
- Environment-specific configuration steps
- Access control configuration
- Troubleshooting common issues
- Disaster recovery procedures

## VALIDATION CRITERIA

- [ ] S3 bucket created with proper encryption and versioning
- [ ] DynamoDB table configured for state locking
- [ ] Backend configuration supports all three environments (dev, staging, prod)
- [ ] IAM roles and policies restrict access to authorized personnel
- [ ] State files are encrypted and properly isolated per environment
- [ ] Audit logging configured for all backend operations
- [ ] Bootstrap process documented and tested
- [ ] Disaster recovery procedures documented and validated
- [ ] Backend supports both local development and CI/CD pipeline access
- [ ] Cross-region backup configured for production environment

Execute this prompt with the least amount of changes by reviewing the current backend state configuration in terraform
and doing only the necessary changes to achieve the missing criteria, don't forget to cleanup and remove unused S3 state
buckets (use aws cli)

# PROMPT 5: Infrastructure Validation and Testing Framework

Create a comprehensive testing and validation framework for Terraform infrastructure including automated tests, security
scanning, and deployment validation to ensure reliable and secure infrastructure provisioning for the MedFlow system.

## PROJECT CONTEXT

The MedFlow healthcare management system requires robust infrastructure validation to ensure patient data security and
system reliability. The infrastructure must be thoroughly tested before deployment and continuously monitored for
compliance and security issues.

## EPIC CONTEXT

This testing framework ensures infrastructure quality and security, providing confidence in deployments and supporting
healthcare compliance requirements. It establishes automated validation that integrates with CI/CD pipelines.

## TASK CONTEXT

**Task Goal:** Create infrastructure testing framework that validates Terraform code, security configurations, and
deployment success with automated test execution.

**Quality Requirements:**

- Terraform code passes static analysis (tflint, tfsec)
- Infrastructure deployment tested in isolated environment
- Security configurations validated against healthcare compliance standards

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance:** All infrastructure changes must be validated for security compliance and patient data
protection requirements before deployment to production environments.

**Risk Management:** Automated testing reduces the risk of infrastructure misconfigurations that could lead to security
vulnerabilities or service outages.

## TECHNICAL REQUIREMENTS

**Testing Framework Structure:** 

```
infrastructure/
├── tests/
│   ├── security/
│   │   ├── tfsec.yml        # Security scanning configuration
│   │   └── checkov.yml      # Compliance checking rules
│   ├── validation/
│   │   ├── plan-tests.sh    # Terraform plan validation
│   │   ├── apply-tests.sh   # Deployment validation
│   │   └── connectivity-tests.sh # Network connectivity tests
│   ├── integration/
│   │   ├── vpc-integration.sh    # VPC functionality tests
│   │   └── security-groups.sh    # Security group validation
│   └── scripts/
│       ├── run-all-tests.sh     # Test orchestration
│       └── cleanup-test-env.sh  # Test environment cleanup
├── .github/
│   └── workflows/
│       ├── terraform-test.yml   # CI/CD testing workflow
│       └── security-scan.yml    # Security scanning workflow
└── Makefile                     # Local testing commands
```

**Testing Categories:**

*Static Analysis:*

- TFLint for Terraform best practices
- TFSec for security vulnerability scanning
- Checkov for compliance and security policies
- Custom rules for healthcare-specific requirements

*Plan Validation:*

- Terraform plan execution without errors
- Resource count and naming validation
- Cost estimation and validation
- Environment-specific configuration verification

*Integration Testing:*

- Network connectivity between tiers
- Security group rule validation
- DNS resolution testing
- Load balancer health checks

## FUNCTIONAL REQUIREMENTS

1. **Automated Test Execution**
    - Local testing via Makefile commands
    - CI/CD integration with GitHub Actions
    - Test results reporting and artifact collection

2. **Security Validation**
    - Continuous security scanning of infrastructure code
    - Compliance checking against healthcare standards
    - Vulnerability assessment and reporting

3. **Environment Testing**
    - Isolated test environment provisioning
    - Full deployment validation testing
    - Automated cleanup after testing

## TESTING REQUIREMENTS

**Test Validation:**

- All test scripts execute without errors
- Security scanners identify known vulnerabilities
- Integration tests validate network connectivity
- CI/CD pipeline executes all test phases successfully

**Performance Testing:**

- Infrastructure provisioning completes within 15 minutes
- Test execution time under 10 minutes for CI/CD integration
- Cleanup procedures remove all test resources

## DOCUMENTATION REQUIREMENTS

**Testing Documentation:**

- Testing framework overview and architecture
- Local development testing procedures
- CI/CD testing integration guide
- Security scanning configuration and customization
- Troubleshooting guide for test failures

## VALIDATION CRITERIA

- [ ] TFLint configuration validates Terraform best practices
- [ ] TFSec scans identify and report security vulnerabilities
- [ ] Checkov validates compliance with security policies
- [ ] Terraform plan tests execute successfully for all environments
- [ ] Network connectivity tests validate multi-tier communication
- [ ] Security group tests verify least privilege implementation
- [ ] CI/CD workflow integrates all testing phases
- [ ] Test results are properly reported and archived
- [ ] Automated cleanup removes all test resources
- [ ] Testing framework documentation is complete and accurate
- [ ] Makefile provides convenient local testing commands
- [ ] Healthcare-specific compliance rules are implemented and validated