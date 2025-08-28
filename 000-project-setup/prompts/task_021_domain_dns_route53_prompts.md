# PROMPTS: TASK-021 - Domain and DNS Management with Route53

# ~~PROMPT 1: Route53 Module for Domain Management and SSL Certificates~~

Create a comprehensive Terraform Route53 module that manages DNS records, SSL certificates, and domain configuration for
the MedFlow healthcare platform, supporting multi-environment deployments with proper subdomain structure.

## PROJECT CONTEXT

The MedFlow healthcare management system requires proper domain management for professional healthcare services. The
platform uses the custom domain "al-meisan.io" (purchased from Hostinger) with environment-specific subdomains for
frontend and backend services. SSL certificates and proper DNS routing are critical for healthcare data security and
HIPAA compliance.

before started we have all this configured on AWS (managed outside terraform).

* a Route 53 hosted zone (name: al-meisan.io, Hosted zone ID, Z00917051VVCQVEH9WN4P)
* a configured certificate with the status issued for medflow.al-meisan.io: (identifier:
  a1f39393-f9ff-440b-b873-e34589faf170, ARN: arn:aws:acm:us-east-1:568653450770:
  certificate/a1f39393-f9ff-440b-b873-e34589faf170)
* a configured certificate with the status issued for al-meisan.io, *.al-meisan.io, *.medflow.al-meisan.io and www.al-meisan.io (identifier:
    a1f39393-f9ff-440b-b873-e34589faf170, ARN: arn:aws:acm:us-east-1:568653450770:
    certificate/a1f39393-f9ff-440b-b873-e34589faf170)
* if you need the arn or id of any of this element use the name to retrieve it naver hard code the id or the ARN

## EPIC CONTEXT

This task is part of the infrastructure deployment epic, establishing professional domain management that enables secure
healthcare services access. The DNS configuration must support multi-environment deployments while maintaining security
and performance standards required for medical applications.

## TASK CONTEXT

**Task Goal:** Create Route53 module managing DNS records, SSL certificates via ACM, and proper domain routing for all
environments with healthcare-grade security.

**Given Information:**

- Hosted zone already created for "al-meisan.io"
- Nameservers updated in Hostinger
- SSL certificate requested for base domain
- Production: app.medflow.al-meisan.io (frontend), api.medflow.al-meisan.io (backend)
- Other environments: app.[env].medflow.al-meisan.io, api.[env].medflow.al-meisan.io

## BUSINESS DOMAIN CONTEXT

**Healthcare Security Requirements:** Medical platforms require SSL/TLS encryption for all data transmission, with
proper certificate management and renewal processes.

**Professional Presence:** Healthcare providers need professional domain structure that inspires patient trust while
supporting technical requirements for multi-tenant operations.

## TECHNICAL REQUIREMENTS

**Route53 Module Structure:**

```
infrastructure/modules/route53/
├── main.tf              # Route53 records, ACM certificates
├── variables.tf         # Environment-specific configuration
├── outputs.tf           # DNS names, certificate ARNs
├── validation.tf        # Domain validation records
└── README.md            # Module documentation
```

**DNS Record Configuration:**

```hcl
# Production records
app.medflow.al-meisan.io    → CloudFront distribution
api.medflow.al-meisan.io    → ALB for backend services

# Environment-specific records
app.dev.medflow.al-meisan.io     → Dev CloudFront
api.dev.medflow.al-meisan.io     → Dev ALB
app.staging.medflow.al-meisan.io → Staging CloudFront
api.staging.medflow.al-meisan.io → Staging ALB
```

**SSL Certificate Management:**

- Wildcard certificate for *.medflow.al-meisan.io
- Additional certificate for *.dev.medflow.al-meisan.io and *.staging.medflow.al-meisan.io
- Automatic DNS validation via Route53
- Certificate renewal automation

## FUNCTIONAL REQUIREMENTS

1. **Multi-Environment DNS Management**
    - Automatic subdomain creation based on environment
    - A records for ALB endpoints
    - CNAME records for CloudFront distributions
    - Health check configuration for failover

2. **SSL Certificate Automation**
    - ACM certificate creation and validation
    - DNS validation record management
    - Certificate status monitoring
    - Multi-region certificate support for CloudFront

3. **Domain Security Features**
    - CAA records for certificate authority authorization
    - SPF records for email security
    - DMARC policy configuration
    - DNS query logging for security monitoring

4. **Integration Requirements**
    - Output certificate ARNs for ALB/CloudFront modules
    - Domain names for application configuration
    - Validation status for deployment pipelines
    - Health check endpoints for monitoring

## TESTING REQUIREMENTS

**DNS Validation:**

- Certificate validation completion verification
- DNS propagation testing across regions
- SSL certificate chain validation
- Domain resolution performance testing

**Security Testing:**

- SSL/TLS configuration scanning
- Certificate expiration monitoring
- DNS security best practices validation
- CAA record effectiveness testing

---

# ~~PROMPT 2: CloudFront CDN Module for Frontend Delivery~~

Create a Terraform module for CloudFront CDN configuration that delivers the React frontend application with optimal
performance, security headers, and multi-environment support for the MedFlow healthcare platform.

## PROJECT CONTEXT

The MedFlow React frontend requires global content delivery with healthcare-grade security headers, optimal caching
strategies, and multi-region availability. CloudFront provides edge location delivery, SSL termination, and security
features essential for medical applications.

## EPIC CONTEXT

This module enables professional frontend delivery with performance optimization crucial for healthcare workflows. The
CDN must support Arabic RTL content, handle medical imaging efficiently, and provide security headers required for
healthcare compliance.

## TASK CONTEXT

**Task Goal:** Create CloudFront module with S3 origin, custom domain support, security headers, and
healthcare-optimized caching policies.

**Performance Requirements:**

- Page load time < 3 seconds globally
- Time to first byte < 500ms
- Image optimization for medical content
- Arabic font delivery optimization

## BUSINESS DOMAIN CONTEXT

**Healthcare Performance Needs:** Medical professionals require fast access to patient data and clinical tools. Slow
applications impact patient care quality and clinic efficiency.

**Global Accessibility:** Support for healthcare providers across Algeria with optimal performance regardless of
location, including rural areas with limited connectivity.

## TECHNICAL REQUIREMENTS

**CloudFront Module Structure:**

```
infrastructure/modules/cloudfront/
├── main.tf              # Distribution, behaviors, origins
├── variables.tf         # Configuration parameters
├── outputs.tf           # Distribution ID, domain names
├── headers.tf           # Security headers configuration
├── cache-policies.tf    # Healthcare-optimized caching
└── README.md           # Module documentation
```

**Security Headers Configuration:**

```hcl
# Healthcare-compliant security headers
Content-Security-Policy : "default-src 'self'; script-src 'self' 'unsafe-inline' https://cognito.al-meisan.io"
X-Frame-Options : "DENY"
X-Content-Type-Options : "nosniff"
Strict-Transport-Security : "max-age=31536000; includeSubDomains"
Referrer-Policy : "strict-origin-when-cross-origin"
Permissions-Policy : "camera=(), microphone=(), geolocation=()"
```

**Cache Behavior Optimization:**

```hcl
# Static assets (JS, CSS, images)
cache_behavior {
  path_pattern     = "static/*"
  target_origin_id = "S3-${var.bucket_name}"
  compress         = true
  ttl {
    default = 86400  # 1 day
    max = 604800 # 7 days
  }
}

# API calls (no caching)
cache_behavior {
  path_pattern     = "api/*"
  target_origin_id = "ALB-${var.alb_domain}"
  allowed_methods = ["GET", "HEAD", "OPTIONS", "PUT", "POST", "PATCH", "DELETE"]
  cached_methods = ["GET", "HEAD"]
  ttl {
    default = 0
    max     = 0
  }
}
```

## FUNCTIONAL REQUIREMENTS

1. **Multi-Origin Configuration**
    - S3 bucket origin for static assets
    - ALB origin for API proxying
    - Custom error page handling
    - Origin failover configuration

2. **Performance Optimization**
    - Brotli and gzip compression
    - HTTP/2 and HTTP/3 support
    - Optimized cache behaviors by content type
    - Arabic font preloading

3. **Security Features**
    - AWS WAF integration
    - Origin access identity for S3
    - Geo-restriction capabilities
    - DDoS protection via Shield

4. **Healthcare-Specific Features**
    - HIPAA-compliant logging
    - Medical image optimization
    - Secure cookie handling
    - Session affinity for clinical workflows

## TESTING REQUIREMENTS

**Performance Testing:**

- Global latency measurements
- Cache hit ratio analysis
- Compression effectiveness
- Arabic content rendering speed

**Security Validation:**

- Security header verification
- SSL/TLS configuration testing
- Origin access restriction validation
- WAF rule effectiveness

---

# ~~PROMPT 3: ECS Fargate Module for Backend Container Orchestration~~

Create a comprehensive Terraform ECS Fargate module that deploys the NestJS backend application with auto-scaling,
health monitoring, and healthcare-grade security configurations.

## PROJECT CONTEXT

The MedFlow NestJS backend requires containerized deployment with automatic scaling based on healthcare workload
patterns. The platform must handle variable clinic loads, from single-doctor practices to multi-provider facilities,
while maintaining sub-second response times for critical medical operations.

## EPIC CONTEXT

This module establishes the core application runtime infrastructure, enabling reliable backend service deployment with
zero-downtime updates crucial for continuous healthcare operations. The container orchestration must support rolling
deployments without disrupting active medical consultations.

## TASK CONTEXT

**Task Goal:** Create ECS Fargate module with task definitions, services, auto-scaling policies, and integration with
ALB for load balancing.

**Scaling Requirements:**

- Minimum 2 tasks for high availability
- Scale up to 10 tasks based on CPU/memory
- Healthcare peak hours consideration (9 AM - 12 PM, 4 PM - 7 PM)
- Rapid scale-up for emergency scenarios

## BUSINESS DOMAIN CONTEXT

**Healthcare Availability:** Medical systems require 99.9% uptime as downtime directly impacts patient care. The
infrastructure must handle sudden load spikes during medical emergencies or clinic rush hours.

**Operational Efficiency:** Automatic scaling reduces infrastructure costs during low-usage periods while ensuring
performance during peak clinic hours.

## TECHNICAL REQUIREMENTS

**ECS Module Structure:**

```
infrastructure/modules/ecs/
├── main.tf              # Cluster, services, task definitions
├── variables.tf         # Container configuration
├── outputs.tf           # Service names, task ARNs
├── task-definition.tf   # Container specifications
├── auto-scaling.tf      # Scaling policies
├── monitoring.tf        # CloudWatch integration
└── README.md           # Module documentation
```

**Task Definition Configuration:**

```hcl
# NestJS backend container
container_definitions = [
  {
    name  = "medflow-backend"
    image = "${var.ecr_repository_url}:${var.image_tag}"

    portMappings = [
      {
        containerPort = 3000
        protocol      = "tcp"
      }
    ]

    environment = [
      { name = "NODE_ENV", value = var.environment },
      { name = "DATABASE_HOST", value = var.database_endpoint },
      { name = "COGNITO_REGION", value = var.aws_region }
    ]

    secrets = [
      { name = "DATABASE_PASSWORD", valueFrom = var.database_password_arn },
      { name = "JWT_SECRET", valueFrom = var.jwt_secret_arn }
    ]

    healthCheck = {
      command = ["CMD-SHELL", "curl -f http://localhost:3000/health || exit 1"]
      interval    = 30
      timeout     = 5
      retries     = 3
      startPeriod = 60
    }

    logConfiguration = {
      logDriver = "awslogs"
      options = {
        "awslogs-group"         = "/ecs/medflow-backend"
        "awslogs-region"        = var.aws_region
        "awslogs-stream-prefix" = "backend"
      }
    }
  }
]
```

**Auto-Scaling Configuration:**

```hcl
# CPU-based scaling
resource "aws_appautoscaling_policy" "cpu" {
  name               = "${var.environment}-medflow-cpu-scaling"
  policy_type        = "TargetTrackingScaling"
  resource_id        = aws_appautoscaling_target.ecs.resource_id
  scalable_dimension = aws_appautoscaling_target.ecs.scalable_dimension
  service_namespace  = aws_appautoscaling_target.ecs.service_namespace

  target_tracking_scaling_policy_configuration {
    target_value = 70.0
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageCPUUtilization"
    }
    scale_in_cooldown  = 300
    scale_out_cooldown = 60
  }
}

# Memory-based scaling
resource "aws_appautoscaling_policy" "memory" {
  name        = "${var.environment}-medflow-memory-scaling"
  policy_type = "TargetTrackingScaling"

  target_tracking_scaling_policy_configuration {
    target_value = 75.0
    predefined_metric_specification {
      predefined_metric_type = "ECSServiceAverageMemoryUtilization"
    }
  }
}
```

## FUNCTIONAL REQUIREMENTS

1. **Container Management**
    - Blue-green deployment support
    - Rolling update configuration
    - Health check integration
    - Container insights enablement

2. **Service Integration**
    - ALB target group registration
    - Service discovery configuration
    - Secrets Manager integration
    - Parameter Store for config

3. **Healthcare Workload Patterns**
    - Time-based scaling for clinic hours
    - Rapid scale-up for emergencies
    - Database connection pooling
    - Session persistence for consultations

4. **Monitoring and Alerting**
    - CloudWatch dashboards
    - Container-level metrics
    - Application logs aggregation
    - Performance anomaly detection

## TESTING REQUIREMENTS

**Deployment Testing:**

- Zero-downtime deployment validation
- Rollback scenario testing
- Health check effectiveness
- Load balancer integration

**Scaling Validation:**

- Auto-scaling trigger testing
- Scale-up/down timing verification
- Database connection management
- Session persistence during scaling

---

# ~~PROMPT 4: Application Load Balancer Module with SSL Termination~~

Create a Terraform ALB module that provides load balancing for ECS services with SSL termination, health monitoring, and
request routing for the MedFlow backend services.

## PROJECT CONTEXT

The MedFlow backend requires intelligent load balancing with SSL termination, health-based routing, and support for
WebSocket connections used in real-time clinical features. The ALB must handle both REST API traffic and WebSocket
connections for features like real-time appointment updates.

## EPIC CONTEXT

This module provides the entry point for all backend API traffic, ensuring high availability and intelligent request
distribution across container instances. The load balancer is critical for maintaining service reliability during
deployments and scaling events.

## TASK CONTEXT

**Task Goal:** Create ALB module with SSL termination, target group configuration, and health check monitoring for
backend services.

**Performance Requirements:**

- Request routing latency < 10ms
- SSL handshake time < 100ms
- WebSocket connection support
- 10,000+ concurrent connections

## BUSINESS DOMAIN CONTEXT

**Healthcare API Reliability:** Medical applications require consistent API performance for clinical operations. Load
balancing ensures no single point of failure affects patient care delivery.

**Real-time Features:** Clinical collaboration features require WebSocket support for real-time updates on patient
status, appointment changes, and emergency notifications.

## TECHNICAL REQUIREMENTS

**ALB Module Structure:**

```
infrastructure/modules/alb/
├── main.tf              # ALB, listeners, target groups
├── variables.tf         # Configuration parameters
├── outputs.tf           # ALB DNS, target group ARNs
├── listeners.tf         # HTTP/HTTPS listener rules
├── security.tf          # SSL policies and certificates
└── README.md           # Module documentation
```

**Listener Configuration:**

```hcl
# HTTPS listener with SSL termination
resource "aws_lb_listener" "https" {
  load_balancer_arn = aws_lb.main.arn
  port              = "443"
  protocol          = "HTTPS"
  ssl_policy        = "ELBSecurityPolicy-TLS-1-2-2017-01"
  certificate_arn   = var.certificate_arn

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.backend.arn
  }
}

# HTTP to HTTPS redirect
resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.main.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type = "redirect"
    redirect {
      port        = "443"
      protocol    = "HTTPS"
      status_code = "HTTP_301"
    }
  }
}

# WebSocket support in target group
resource "aws_lb_target_group" "backend" {
  name        = "${var.environment}-medflow-backend"
  port        = 3000
  protocol    = "HTTP"
  vpc_id      = var.vpc_id
  target_type = "ip"

  health_check {
    enabled             = true
    healthy_threshold   = 2
    interval            = 30
    matcher             = "200"
    path                = "/health"
    port                = "traffic-port"
    protocol            = "HTTP"
    timeout             = 5
    unhealthy_threshold = 3
  }

  stickiness {
    type            = "lb_cookie"
    cookie_duration = 86400
    enabled         = true
  }
}
```

**Path-Based Routing Rules:**

```hcl
# API versioning support
resource "aws_lb_listener_rule" "api_v1" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 100

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.backend.arn
  }

  condition {
    path_pattern {
      values = ["/api/v1/*"]
    }
  }
}

# WebSocket routing
resource "aws_lb_listener_rule" "websocket" {
  listener_arn = aws_lb_listener.https.arn
  priority     = 90

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.websocket.arn
  }

  condition {
    path_pattern {
      values = ["/ws/*"]
    }
  }
}
```

## FUNCTIONAL REQUIREMENTS

1. **SSL/TLS Management**
    - TLS 1.2+ enforcement
    - Certificate rotation support
    - SSL policy configuration
    - HTTPS redirect enforcement

2. **Traffic Distribution**
    - Round-robin load balancing
    - Sticky sessions for stateful operations
    - Connection draining configuration
    - Cross-zone load balancing

3. **Health Monitoring**
    - Custom health check endpoints
    - Configurable thresholds
    - Automatic unhealthy instance removal
    - Health status API exposure

4. **Security Features**
    - AWS WAF integration
    - Security group configuration
    - Request header validation
    - Rate limiting support

## TESTING REQUIREMENTS

**Load Testing:**

- 10K concurrent connection handling
- WebSocket connection persistence
- SSL termination performance
- Failover scenario validation

**Security Testing:**

- SSL/TLS configuration scanning
- Security header validation
- WAF rule effectiveness
- DDoS simulation testing

---

# ~~PROMPT 5: ECR Repository Module for Container Registry~~

Create a Terraform ECR module that manages Docker container repositories with lifecycle policies, security scanning, and
multi-environment image management for the MedFlow platform.

## PROJECT CONTEXT

The MedFlow platform requires secure container image storage with automated vulnerability scanning essential for
healthcare applications. ECR provides private Docker repositories with integration to ECS for seamless deployment
workflows.

## EPIC CONTEXT

This module establishes container image management infrastructure, enabling secure storage and distribution of
application images across environments. Container security is critical for healthcare compliance and protecting patient
data.

## TASK CONTEXT

**Task Goal:** Create ECR module with repositories for backend and frontend images, lifecycle policies for cost
optimization, and security scanning configuration.

**Security Requirements:**

- Automated vulnerability scanning
- Image signing and verification
- Encryption at rest
- Access control via IAM

## BUSINESS DOMAIN CONTEXT

**Healthcare Security Standards:** Medical applications require rigorous security validation of all components,
including container images that may process patient data.

**Compliance Requirements:** Container images must be scanned for vulnerabilities and maintain audit trails for
healthcare compliance and security certifications.

## TECHNICAL REQUIREMENTS

**ECR Module Structure:**

```
infrastructure/modules/ecr/
├── main.tf              # Repositories, lifecycle policies
├── variables.tf         # Repository configuration
├── outputs.tf           # Repository URLs, ARNs
├── lifecycle-rules.tf   # Image retention policies
├── security.tf          # Scanning configuration
└── README.md           # Module documentation
```

**Repository Configuration:**

```hcl
# Backend repository with scanning
resource "aws_ecr_repository" "backend" {
  name                 = "medflow-backend-${var.environment}"
  image_tag_mutability = var.environment == "prod" ? "IMMUTABLE" : "MUTABLE"

  encryption_configuration {
    encryption_type = "AES256"
  }

  image_scanning_configuration {
    scan_on_push = true
  }
}

# Lifecycle policy for cost optimization
resource "aws_ecr_lifecycle_policy" "backend" {
  repository = aws_ecr_repository.backend.name

  policy = jsonencode({
    rules = [
      {
        rulePriority = 1
        description  = "Keep last 10 production images"
        selection = {
          tagStatus   = "tagged"
          tagPrefixList = ["prod"]
          countType   = "imageCountMoreThan"
          countNumber = 10
        }
        action = {
          type = "expire"
        }
      },
      {
        rulePriority = 2
        description  = "Remove untagged images after 7 days"
        selection = {
          tagStatus   = "untagged"
          countType   = "sinceImagePushed"
          countUnit   = "days"
          countNumber = 7
        }
        action = {
          type = "expire"
        }
      }
    ]
  })
}
```

**Security Scanning Configuration:**

```hcl
# Enhanced scanning configuration
resource "aws_ecr_registry_scanning_configuration" "main" {
  scan_type = "ENHANCED"

  rule {
    scan_frequency = "CONTINUOUS_SCAN"
    repository_filter {
      filter      = "medflow-*"
      filter_type = "WILDCARD"
    }
  }
}

# Repository policy for cross-account access
resource "aws_ecr_repository_policy" "backend" {
  repository = aws_ecr_repository.backend.name

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Sid    = "AllowPull"
        Effect = "Allow"
        Principal = {
          AWS = var.ecs_task_role_arn
        }
        Action = [
          "ecr:GetDownloadUrlForLayer",
          "ecr:BatchGetImage",
          "ecr:BatchCheckLayerAvailability"
        ]
      }
    ]
  })
}
```

## FUNCTIONAL REQUIREMENTS

1. **Image Management**
    - Multi-environment repositories
    - Tag immutability for production
    - Automated cleanup policies
    - Cross-region replication

2. **Security Features**
    - Vulnerability scanning on push
    - Continuous scanning for CVEs
    - Image signing support
    - Encryption at rest

3. **Access Control**
    - IAM-based authentication
    - Repository-level policies
    - Pull-through cache configuration
    - Service account integration

4. **Cost Optimization**
    - Lifecycle rules by environment
    - Unused image cleanup
    - Storage metrics tracking
    - Replication cost management

## TESTING REQUIREMENTS

**Security Testing:**

- Vulnerability scan result validation
- Access policy effectiveness
- Encryption verification
- Image signing validation

**Operational Testing:**

- Lifecycle policy effectiveness
- Cross-region replication
- Pull performance metrics
- Storage optimization validation

---

# ~~PROMPT 6: S3 Static Assets Module with Lifecycle Policies~~

Create a Terraform S3 module that manages static asset storage for the React frontend with intelligent lifecycle
policies, CloudFront integration, and healthcare-compliant data handling.

## PROJECT CONTEXT

The MedFlow frontend requires secure static asset storage for JavaScript bundles, CSS files, images, and medical
documents. S3 provides cost-effective storage with fine-grained access control essential for healthcare applications
handling sensitive medical imagery.

## EPIC CONTEXT

This module establishes static asset management infrastructure, enabling efficient content delivery while maintaining
security for medical images and documents. The storage solution must handle both public assets (UI resources) and
private medical content.

## TASK CONTEXT

**Task Goal:** Create S3 module with buckets for static assets, lifecycle policies for cost optimization, and CloudFront
origin access identity configuration.

**Storage Requirements:**

- Public assets bucket for UI resources
- Private bucket for medical documents
- Intelligent tiering for cost optimization
- HIPAA-compliant encryption

## BUSINESS DOMAIN CONTEXT

**Medical Document Storage:** Healthcare platforms handle sensitive medical images, lab reports, and clinical documents
requiring secure storage with access audit trails.

**Cost Management:** Medical imaging files can be large; intelligent lifecycle policies help manage storage costs while
maintaining accessibility for active patient records.

## TECHNICAL REQUIREMENTS

**S3 Module Structure:**

```
infrastructure/modules/s3-static/
├── main.tf              # Buckets, policies, CORS
├── variables.tf         # Bucket configuration
├── outputs.tf           # Bucket names, ARNs
├── lifecycle.tf         # Intelligent tiering rules
├── access-policies.tf   # IAM and bucket policies
└── README.md           # Module documentation
```

**Bucket Configuration:**

```hcl
# Public static assets bucket
resource "aws_s3_bucket" "static_assets" {
  bucket = "${var.environment}-medflow-static-assets"
}

resource "aws_s3_bucket_public_access_block" "static_assets" {
  bucket = aws_s3_bucket.static_assets.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

resource "aws_s3_bucket_encryption" "static_assets" {
  bucket = aws_s3_bucket.static_assets.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Lifecycle configuration for cost optimization
resource "aws_s3_bucket_lifecycle_configuration" "static_assets" {
  bucket = aws_s3_bucket.static_assets.id

  rule {
    id     = "archive-old-builds"
    status = "Enabled"

    filter {
      prefix = "builds/"
    }

    transition {
      days          = 30
      storage_class = "STANDARD_IA"
    }

    transition {
      days          = 90
      storage_class = "GLACIER"
    }

    expiration {
      days = 365
    }
  }

  rule {
    id     = "intelligent-tiering-media"
    status = "Enabled"

    filter {
      prefix = "media/"
    }

    transition {
      days          = 0
      storage_class = "INTELLIGENT_TIERING"
    }
  }
}

# CloudFront OAI access policy
data "aws_iam_policy_document" "s3_policy" {
  statement {
    actions = ["s3:GetObject"]
    resources = ["${aws_s3_bucket.static_assets.arn}/*"]

    principals {
      type = "AWS"
      identifiers = [aws_cloudfront_origin_access_identity.oai.iam_arn]
    }
  }
}

resource "aws_s3_bucket_policy" "static_assets" {
  bucket = aws_s3_bucket.static_assets.id
  policy = data.aws_iam_policy_document.s3_policy.json
}
```

**CORS Configuration:**

```hcl
resource "aws_s3_bucket_cors_configuration" "static_assets" {
  bucket = aws_s3_bucket.static_assets.id

  cors_rule {
    allowed_headers = ["*"]
    allowed_methods = ["GET", "HEAD"]
    allowed_origins = var.environment == "prod" ?
      ["https://app.medflow.al-meisan.io"] :
      ["https://app.${var.environment}.medflow.al-meisan.io"]
    expose_headers = ["ETag"]
    max_age_seconds = 3000
  }
}
```

## FUNCTIONAL REQUIREMENTS

1. **Storage Organization**
    - Separate buckets by content type
    - Folder structure for versions
    - Medical document segregation
    - Audit log storage

2. **Access Control**
    - CloudFront OAI integration
    - IAM role-based access
    - Bucket policies for security
    - MFA delete protection

3. **Cost Optimization**
    - Intelligent tiering activation
    - Lifecycle rules by content type
    - Old version cleanup
    - Archive policies for compliance

4. **Healthcare Compliance**
    - Encryption at rest
    - Access logging enabled
    - Versioning for audit trails
    - Object lock for records

## TESTING REQUIREMENTS

**Security Testing:**

- Encryption verification
- Access policy validation
- CORS configuration testing
- Audit trail completeness

**Performance Testing:**

- Upload/download speeds
- CloudFront cache hit rates
- Lifecycle policy effectiveness
- Cost optimization metrics

---

# ~~PROMPT 7: Complete CI/CD Pipeline for Infrastructure Deployment~~

Create comprehensive GitHub Actions workflows that handle infrastructure deployment across environments with approval
gates, security validation, and rollback capabilities for the MedFlow platform.

## PROJECT CONTEXT

The MedFlow platform requires automated infrastructure deployment with strict approval processes for production changes.
The pipeline must handle Terraform deployments, validate security compliance, and maintain audit trails for healthcare
regulatory requirements.

## EPIC CONTEXT

This pipeline enables infrastructure as code practices with automated deployment workflows. The system must support
rapid development iterations while maintaining security controls required for healthcare infrastructure.

## TASK CONTEXT

**Task Goal:** Create GitHub Actions workflows for infrastructure deployment with environment-specific approval gates,
security scanning, and automated rollback capabilities.

**Deployment Requirements:**

- Automated deployment to dev environment
- Manual approval for staging/production
- Security scanning before deployment
- Rollback capabilities

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance:** Infrastructure changes must be tracked, approved, and auditable for healthcare compliance.
Production changes require multiple approvals and security validation.

**Operational Excellence:** Automated infrastructure deployment reduces human error while maintaining security controls
essential for patient data protection.

## TECHNICAL REQUIREMENTS

**Workflow Structure:**

```
.github/workflows/
├── infrastructure-plan.yml     # Terraform plan on PR
├── infrastructure-deploy.yml   # Environment deployment
├── infrastructure-destroy.yml  # Teardown workflow
└── infrastructure-drift.yml    # Drift detection
```

**Infrastructure Plan Workflow:**

```yaml
name: Infrastructure Plan

on:
  pull_request:
    paths:
      - 'clinic-infra/**'
      - '.github/workflows/infrastructure-*.yml'

env:
  TERRAFORM_VERSION: "1.5.0"
  AWS_REGION: "eu-west-1"

jobs:
  terraform-checks:
    name: Terraform Validation
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Format Check
        run: |
          cd clinic-infra
          terraform fmt -check -recursive

      - name: Terraform Init
        run: |
          cd clinic-infra/environments/dev
          terraform init -backend=false

      - name: Terraform Validate
        run: |
          cd clinic-infra/environments/dev
          terraform validate

      - name: TFSec Security Scan
        uses: aquasecurity/tfsec-action@v1.0.3
        with:
          working_directory: clinic-infra
          soft_fail: false

      - name: Checkov Security Scan
        uses: bridgecrewio/checkov-action@master
        with:
          directory: clinic-infra
          framework: terraform
          output_format: github_failed_only
          download_external_modules: true

  terraform-plan:
    name: Terraform Plan - ${{ matrix.environment }}
    needs: terraform-checks
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [ dev, staging, prod ]
    permissions:
      contents: read
      pull-requests: write

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2
        with:
          terraform_version: ${{ env.TERRAFORM_VERSION }}

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Init
        run: |
          cd clinic-infra/environments/${{ matrix.environment }}
          terraform init

      - name: Terraform Plan
        id: plan
        run: |
          cd clinic-infra/environments/${{ matrix.environment }}
          terraform plan -out=tfplan -input=false

      - name: Post Plan to PR
        uses: actions/github-script@v6
        if: github.event_name == 'pull_request'
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          script: |
            const output = `#### Terraform Plan - ${{ matrix.environment }} 📋

            <details><summary>Show Plan</summary>

            \`\`\`terraform
            ${{ steps.plan.outputs.stdout }}
            \`\`\`

            </details>`;

            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
```

**Infrastructure Deploy Workflow:**

```yaml
name: Infrastructure Deploy

on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        type: choice
        options:
          - dev
          - staging
          - prod
      action:
        description: 'Deployment action'
        required: true
        type: choice
        options:
          - plan
          - apply
          - destroy

jobs:
  approval:
    name: Deployment Approval
    runs-on: ubuntu-latest
    if: ${{ inputs.environment != 'dev' }}
    environment: ${{ inputs.environment }}-approval
    steps:
      - name: Approval Required
        run: echo "Approval required for ${{ inputs.environment }} deployment"

  deploy:
    name: Deploy to ${{ inputs.environment }}
    needs: [ approval ]
    if: ${{ always() && (needs.approval.result == 'success' || inputs.environment == 'dev') }}
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Terraform Init
        run: |
          cd clinic-infra/environments/${{ inputs.environment }}
          terraform init

      - name: Create Terraform Plan
        if: ${{ inputs.action == 'plan' || inputs.action == 'apply' }}
        run: |
          cd clinic-infra/environments/${{ inputs.environment }}
          terraform plan -out=tfplan -input=false

      - name: Apply Terraform Changes
        if: ${{ inputs.action == 'apply' }}
        run: |
          cd clinic-infra/environments/${{ inputs.environment }}
          terraform apply -auto-approve tfplan

      - name: Post-Deployment Validation
        if: ${{ inputs.action == 'apply' }}
        run: |
          cd clinic-infra
          ./scripts/validate.sh ${{ inputs.environment }}

      - name: Notify Deployment Status
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: |
            Infrastructure ${{ inputs.action }} for ${{ inputs.environment }}
            Status: ${{ job.status }}
            Actor: ${{ github.actor }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

## FUNCTIONAL REQUIREMENTS

1. **Environment Management**
    - Separate approval workflows
    - Environment-specific secrets
    - State file locking
    - Concurrent deployment prevention

2. **Security Validation**
    - Pre-deployment security scanning
    - Policy compliance checking
    - Secret rotation validation
    - Access control verification

3. **Deployment Safety**
    - Plan review before apply
    - Automated rollback triggers
    - Health check validation
    - Drift detection alerts

4. **Audit and Compliance**
    - Deployment audit trails
    - Change tracking
    - Approval documentation
    - Cost impact analysis

## TESTING REQUIREMENTS

**Pipeline Testing:**

- Approval gate functionality
- Rollback mechanism validation
- Security scan effectiveness
- Multi-environment deployment

**Operational Testing:**

- Concurrent deployment handling
- State lock management
- Error recovery procedures
- Notification accuracy

---

# ~~PROMPT 8: Backend Container Build and Deployment Pipeline~~

Create GitHub Actions workflows that build, test, scan, and deploy the NestJS backend containers to ECR and ECS with
healthcare-compliant security validation.

## PROJECT CONTEXT

The MedFlow backend requires automated container building with comprehensive security scanning, testing, and deployment
to ECS. The pipeline must ensure only secure, tested containers reach production environments handling patient data.

## EPIC CONTEXT

This pipeline enables continuous deployment of backend services with security-first practices. Healthcare applications
require rigorous validation before deployment to protect patient data and ensure service reliability.

## TASK CONTEXT

**Task Goal:** Create container build pipeline with multi-stage testing, security scanning, and automated deployment to
ECS across environments.

**Build Requirements:**

- Multi-stage Docker builds
- Comprehensive security scanning
- Test execution in containers
- Automatic deployment on merge

## BUSINESS DOMAIN CONTEXT

**Healthcare Security:** Container images processing patient data must be free of vulnerabilities. Security scanning and
validation are mandatory before deployment to any environment.

**Deployment Reliability:** Healthcare services require zero-downtime deployments with automatic rollback capabilities
to prevent service disruption during patient care.

## TECHNICAL REQUIREMENTS

**Backend Build Workflow:**

```yaml
name: Backend Build and Deploy

on:
  push:
    branches: [ main, develop ]
    paths:
      - 'clinic-backend/**'
      - '.github/workflows/backend-*.yml'
  pull_request:
    paths:
      - 'clinic-backend/**'

env:
  ECR_REPOSITORY: medflow-backend
  AWS_REGION: eu-west-1
  DOCKERFILE_PATH: ./clinic-backend/Dockerfile

jobs:
  test:
    name: Test Backend
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15-alpine
        env:
          POSTGRES_USER: medflow
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: medflow_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: clinic-backend/package-lock.json

      - name: Install dependencies
        run: |
          cd clinic-backend
          npm ci

      - name: Run linting
        run: |
          cd clinic-backend
          npm run lint

      - name: Run type checking
        run: |
          cd clinic-backend
          npm run type-check

      - name: Run unit tests
        run: |
          cd clinic-backend
          npm run test:cov
        env:
          DATABASE_HOST: localhost
          DATABASE_PORT: 5432
          DATABASE_USER: medflow
          DATABASE_PASSWORD: testpass
          DATABASE_NAME: medflow_test

      - name: Run integration tests
        run: |
          cd clinic-backend
          npm run test:integration
        env:
          DATABASE_URL: postgresql://medflow:testpass@localhost:5432/medflow_test

      - name: Upload coverage reports
        uses: codecov/codecov-action@v3
        with:
          directory: clinic-backend/coverage
          flags: backend
          fail_ci_if_error: true

  build:
    name: Build and Scan Container
    needs: test
    runs-on: ubuntu-latest
    if: github.event_name == 'push'

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build Docker image
        id: build-image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build \
            --cache-from $ECR_REGISTRY/$ECR_REPOSITORY:latest \
            --build-arg BUILDKIT_INLINE_CACHE=1 \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG \
            -t $ECR_REGISTRY/$ECR_REPOSITORY:latest \
            -f $DOCKERFILE_PATH \
            clinic-backend

      - name: Run Trivy security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}
          format: 'sarif'
          output: 'trivy-results.sarif'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'

      - name: Upload Trivy scan results
        uses: github/codeql-action/upload-sarif@v2
        if: always()
        with:
          sarif_file: 'trivy-results.sarif'

      - name: Push to ECR
        if: success()
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

  deploy-dev:
    name: Deploy to Development
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    environment: dev

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Update ECS service
        run: |
          aws ecs update-service \
            --cluster dev-medflow-cluster \
            --service dev-medflow-backend \
            --force-new-deployment \
            --desired-count 2

      - name: Wait for deployment
        run: |
          aws ecs wait services-stable \
            --cluster dev-medflow-cluster \
            --services dev-medflow-backend

      - name: Validate deployment
        run: |
          HEALTH_CHECK=$(curl -s -o /dev/null -w "%{http_code}" https://api.dev.medflow.al-meisan.io/health)
          if [ $HEALTH_CHECK -ne 200 ]; then
            echo "Health check failed with status $HEALTH_CHECK"
            exit 1
          fi

  deploy-prod:
    name: Deploy to Production
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production

    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_PROD_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Blue-Green Deployment
        run: |
          # Create new task definition revision
          TASK_DEFINITION=$(aws ecs describe-task-definition \
            --task-definition prod-medflow-backend \
            --query 'taskDefinition' \
            --output json)

          NEW_TASK_DEF=$(echo $TASK_DEFINITION | \
            jq --arg IMAGE "${{ steps.login-ecr.outputs.registry }}/${{ env.ECR_REPOSITORY }}:${{ github.sha }}" \
            '.containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')

          # Register new task definition
          NEW_TASK_ARN=$(aws ecs register-task-definition \
            --cli-input-json "$NEW_TASK_DEF" \
            --query 'taskDefinition.taskDefinitionArn' \
            --output text)

          # Update service with new task definition
          aws ecs update-service \
            --cluster prod-medflow-cluster \
            --service prod-medflow-backend \
            --task-definition $NEW_TASK_ARN \
            --desired-count 4 \
            --deployment-configuration "maximumPercent=200,minimumHealthyPercent=100"

      - name: Monitor deployment
        run: |
          # Monitor deployment progress
          DEPLOYMENT_ID=$(aws ecs describe-services \
            --cluster prod-medflow-cluster \
            --services prod-medflow-backend \
            --query 'services[0].deployments[0].id' \
            --output text)

          # Wait for deployment with timeout
          timeout 600 bash -c 'until aws ecs wait services-stable --cluster prod-medflow-cluster --services prod-medflow-backend; do sleep 30; done'

      - name: Post-deployment validation
        run: |
          # Comprehensive health checks
          endpoints=("/health" "/api/v1/health" "/metrics")
          for endpoint in "${endpoints[@]}"; do
            response=$(curl -s -o /dev/null -w "%{http_code}" https://api.medflow.al-meisan.io$endpoint)
            if [ $response -ne 200 ]; then
              echo "Health check failed for $endpoint with status $response"
              exit 1
            fi
          done
```

## FUNCTIONAL REQUIREMENTS

1. **Build Optimization**
    - Multi-stage Docker builds
    - Layer caching strategies
    - Build-time secret handling
    - Minimal image size

2. **Security Scanning**
    - Container vulnerability scanning
    - Dependency checking
    - SAST for code analysis
    - License compliance

3. **Deployment Strategies**
    - Blue-green deployments
    - Canary releases
    - Automatic rollback
    - Health check validation

4. **Monitoring Integration**
    - Deployment metrics
    - Performance baselines
    - Error rate tracking
    - Alert configuration

## TESTING REQUIREMENTS

**Pipeline Testing:**

- Build reproducibility
- Security scan accuracy
- Deployment rollback testing
- Multi-environment flows

**Integration Testing:**

- Container startup validation
- Health endpoint testing
- Database connectivity
- Service mesh integration

---

# ~~PROMPT 9: Frontend Build and Deployment Pipeline~~

Create GitHub Actions workflows for building, testing, and deploying the React frontend to S3/CloudFront with cache
invalidation and progressive deployment strategies.

## PROJECT CONTEXT

The MedFlow React frontend requires automated building with optimization for healthcare UI performance. The pipeline
must handle Arabic RTL layouts, medical imaging assets, and ensure rapid global delivery through CloudFront CDN.

## EPIC CONTEXT

This pipeline enables continuous deployment of the frontend application with performance optimization crucial for
healthcare workflows. Medical professionals require fast, reliable access to patient interfaces regardless of location.

## TASK CONTEXT

**Task Goal:** Create frontend build pipeline with testing, optimization, and deployment to S3/CloudFront with
intelligent cache management.

**Performance Requirements:**

- Build time < 5 minutes
- Bundle size optimization
- Arabic font subsetting
- Medical image optimization

## BUSINESS DOMAIN CONTEXT

**Healthcare UI Performance:** Medical professionals need instant access to patient data. Slow interfaces directly
impact patient care quality and clinic efficiency.

**Global Accessibility:** The platform must perform well across Algeria, including areas with limited internet
connectivity, while maintaining security for patient data.

## TECHNICAL REQUIREMENTS

**Frontend Build Workflow:**

```yaml
name: Frontend Build and Deploy

on:
  push:
    branches: [ main, develop ]
    paths:
      - 'clinic-portal/**'
      - '.github/workflows/frontend-*.yml'
  pull_request:
    paths:
      - 'clinic-portal/**'

env:
  NODE_VERSION: '20'
  AWS_REGION: eu-west-1

jobs:
  test-and-build:
    name: Test and Build Frontend
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
          cache-dependency-path: clinic-portal/package-lock.json

      - name: Install dependencies
        run: |
          cd clinic-portal
          npm ci

      - name: Run linting
        run: |
          cd clinic-portal
          npm run lint

      - name: Run type checking
        run: |
          cd clinic-portal
          npm run type-check

      - name: Run unit tests
        run: |
          cd clinic-portal
          npm run test:coverage
        env:
          CI: true

      - name: Build application
        run: |
          cd clinic-portal
          npm run build
        env:
          VITE_API_URL: ${{ github.ref == 'refs/heads/main' && 'https://api.medflow.al-meisan.io' || 'https://api.dev.medflow.al-meisan.io' }}
          VITE_COGNITO_REGION: ${{ env.AWS_REGION }}
          VITE_COGNITO_USER_POOL_ID: ${{ secrets.COGNITO_USER_POOL_ID }}
          VITE_COGNITO_CLIENT_ID: ${{ secrets.COGNITO_CLIENT_ID }}

      - name: Analyze bundle size
        run: |
          cd clinic-portal
          npm run analyze

          # Check bundle size limits
          MAIN_BUNDLE_SIZE=$(find dist/assets -name "*.js" -exec du -b {} + | awk '{sum+=$1} END {print sum}')
          if [ $MAIN_BUNDLE_SIZE -gt 512000 ]; then
            echo "Bundle size exceeds limit: $MAIN_BUNDLE_SIZE bytes"
            exit 1
          fi

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: frontend-build
          path: clinic-portal/dist
          retention-days: 7

  security-scan:
    name: Security Scanning
    needs: test-and-build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run npm audit
        run: |
          cd clinic-portal
          npm audit --production --audit-level=high

      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        with:
          args: --severity-threshold=high --project-path=clinic-portal
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}

      - name: OWASP dependency check
        uses: jeremylong/DependencyCheck-Action@main
        with:
          project: 'MedFlow Frontend'
          path: 'clinic-portal'
          format: 'HTML'
          args: >
            --enableRetired
            --enableExperimental

  deploy-dev:
    name: Deploy to Development
    needs: [ test-and-build, security-scan ]
    if: github.ref == 'refs/heads/develop' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: dev

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: frontend-build
          path: dist

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to S3
        run: |
          # Sync with proper cache headers
          aws s3 sync dist/ s3://dev-medflow-static-assets/ \
            --delete \
            --cache-control "public, max-age=31536000" \
            --exclude "index.html" \
            --exclude "*.json"

          # Upload index.html and JSON with no-cache
          aws s3 cp dist/index.html s3://dev-medflow-static-assets/ \
            --cache-control "no-cache, no-store, must-revalidate"

          aws s3 sync dist/ s3://dev-medflow-static-assets/ \
            --exclude "*" \
            --include "*.json" \
            --cache-control "no-cache, no-store, must-revalidate"

      - name: Invalidate CloudFront
        run: |
          DISTRIBUTION_ID=$(aws cloudfront list-distributions \
            --query "DistributionList.Items[?Origins.Items[?DomainName=='dev-medflow-static-assets.s3.amazonaws.com']].Id" \
            --output text)

          aws cloudfront create-invalidation \
            --distribution-id $DISTRIBUTION_ID \
            --paths "/*"

      - name: Validate deployment
        run: |
          sleep 30  # Wait for CloudFront propagation

          # Check if site is accessible
          RESPONSE=$(curl -s -o /dev/null -w "%{http_code}" https://app.dev.medflow.al-meisan.io)
          if [ $RESPONSE -ne 200 ]; then
            echo "Site health check failed with status $RESPONSE"
            exit 1
          fi

  deploy-prod:
    name: Deploy to Production
    needs: [ test-and-build, security-scan ]
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: frontend-build
          path: dist

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_PROD_DEPLOY_ROLE_ARN }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Deploy to S3 with versioning
        run: |
          # Create deployment version
          VERSION=$(date +%Y%m%d%H%M%S)

          # Upload new version to versioned path
          aws s3 sync dist/ s3://prod-medflow-static-assets/versions/$VERSION/ \
            --cache-control "public, max-age=31536000"

          # Update current version pointer
          echo $VERSION > version.txt
          aws s3 cp version.txt s3://prod-medflow-static-assets/current-version.txt

          # Sync to root (blue-green style)
          aws s3 sync dist/ s3://prod-medflow-static-assets/ \
            --delete \
            --cache-control "public, max-age=31536000" \
            --exclude "index.html" \
            --exclude "*.json"

          # Upload index.html with no-cache
          aws s3 cp dist/index.html s3://prod-medflow-static-assets/ \
            --cache-control "no-cache, no-store, must-revalidate" \
            --metadata version=$VERSION

      - name: Progressive CloudFront invalidation
        run: |
          DISTRIBUTION_ID=$(aws cloudfront list-distributions \
            --query "DistributionList.Items[?Origins.Items[?DomainName=='prod-medflow-static-assets.s3.amazonaws.com']].Id" \
            --output text)

          # Create invalidation with monitoring
          INVALIDATION_ID=$(aws cloudfront create-invalidation \
            --distribution-id $DISTRIBUTION_ID \
            --paths "/*" \
            --query "Invalidation.Id" \
            --output text)

          # Monitor invalidation progress
          while true; do
            STATUS=$(aws cloudfront get-invalidation \
              --distribution-id $DISTRIBUTION_ID \
              --id $INVALIDATION_ID \
              --query "Invalidation.Status" \
              --output text)

            if [ "$STATUS" = "Completed" ]; then
              echo "Invalidation completed successfully"
              break
            fi

            echo "Waiting for invalidation to complete... Status: $STATUS"
            sleep 30
          done

      - name: Post-deployment validation
        run: |
          # Comprehensive validation
          endpoints=(
            "https://app.medflow.al-meisan.io"
            "https://app.medflow.al-meisan.io/manifest.json"
            "https://app.medflow.al-meisan.io/robots.txt"
          )

          for endpoint in "${endpoints[@]}"; do
            response=$(curl -s -o /dev/null -w "%{http_code}" "$endpoint")
            if [ $response -ne 200 ]; then
              echo "Validation failed for $endpoint with status $response"
              exit 1
            fi
          done

          # Verify version deployment
          DEPLOYED_VERSION=$(curl -s https://app.medflow.al-meisan.io/version.json | jq -r '.version')
          echo "Deployed version: $DEPLOYED_VERSION"

      - name: Performance testing
        run: |
          # Run Lighthouse CI
          npm install -g @lhci/cli
          lhci autorun \
            --collect.url=https://app.medflow.al-meisan.io \
            --assert.preset=healthcare \
            --upload.target=temporary-public-storage
```

## FUNCTIONAL REQUIREMENTS

1. **Build Optimization**
    - Tree shaking and dead code elimination
    - Code splitting by route
    - Asset optimization (WebP, AVIF)
    - Arabic font subsetting

2. **Deployment Strategies**
    - Blue-green deployments
    - Versioned deployments
    - Rollback capabilities
    - Progressive rollout

3. **Cache Management**
    - Intelligent cache headers
    - Selective invalidation
    - Asset fingerprinting
    - Edge location warming

4. **Performance Monitoring**
    - Lighthouse CI integration
    - Bundle size tracking
    - Core Web Vitals monitoring
    - RTL layout performance

## TESTING REQUIREMENTS

**Build Testing:**

- Bundle size regression tests
- Asset optimization validation
- Dependency security scanning
- Build reproducibility

**Deployment Testing:**

- Cache invalidation effectiveness
- Global availability testing
- Performance benchmarking
- Rollback procedures

---

# PROMPT 10: Complete Infrastructure Documentation Update

Create comprehensive documentation updates that reflect the complete AWS infrastructure setup, deployment procedures,
and operational runbooks for the MedFlow healthcare platform.

## PROJECT CONTEXT

The MedFlow platform now has comprehensive AWS infrastructure requiring detailed documentation for deployment,
operation, and maintenance. Documentation must serve both development teams and operations staff managing healthcare
infrastructure.

## EPIC CONTEXT

Complete infrastructure documentation ensures consistent deployments, rapid troubleshooting, and knowledge transfer.
Healthcare platforms require detailed operational procedures for compliance and reliability.

## TASK CONTEXT

**Task Goal:** Update all infrastructure documentation including README files, deployment guides, troubleshooting
procedures, and disaster recovery plans.

**Documentation Scope:**

- Infrastructure architecture diagrams
- Deployment procedures
- Operational runbooks
- Troubleshooting guides
- Disaster recovery plans

## BUSINESS DOMAIN CONTEXT

**Healthcare Compliance:** Detailed documentation is required for healthcare compliance audits, demonstrating proper
security controls and operational procedures.

**Operational Excellence:** Clear documentation reduces incident response time and ensures consistent operations
critical for healthcare service delivery.

## TECHNICAL REQUIREMENTS

**Documentation Structure:**

```
clinic-infra/
├── README.md                          # Main infrastructure overview
├── docs/
│   ├── architecture/
│   │   ├── overview.md               # Architecture overview with diagrams
│   │   ├── networking.md             # VPC and network design
│   │   ├── security.md               # Security architecture
│   │   └── data-flow.md              # Data flow diagrams
│   ├── deployment/
│   │   ├── quickstart.md             # Quick deployment guide
│   │   ├── prerequisites.md          # Pre-deployment requirements
│   │   ├── environments.md           # Environment-specific guides
│   │   └── rollback.md               # Rollback procedures
│   ├── operations/
│   │   ├── monitoring.md             # Monitoring and alerting
│   │   ├── scaling.md                # Scaling procedures
│   │   ├── backup-recovery.md        # Backup and recovery
│   │   └── incident-response.md      # Incident response procedures
│   └── troubleshooting/
│       ├── common-issues.md          # Common problems and solutions
│       ├── health-checks.md          # Health check debugging
│       ├── performance.md            # Performance troubleshooting
│       └── security-events.md        # Security event response
```

**Main README.md Update:**

```markdown
# MedFlow Healthcare Platform Infrastructure

## Overview

MedFlow is a comprehensive healthcare management platform designed for Algerian medical practices. This repository
contains Infrastructure as Code (IaC) using Terraform for deploying the complete AWS infrastructure.

### Infrastructure Components

- **Networking**: Multi-AZ VPC with public, private, and database subnets
- **Compute**: ECS Fargate for containerized backend services
- **Storage**: S3 for static assets and medical documents
- **Database**: RDS PostgreSQL with encryption and automated backups
- **CDN**: CloudFront for global content delivery
- **Authentication**: AWS Cognito with healthcare-specific attributes
- **DNS**: Route53 with SSL certificate management
- **Monitoring**: CloudWatch dashboards and alerts

## Quick Start

### Prerequisites

1. **AWS Account Setup**
    - AWS account with appropriate permissions
    - Configured AWS CLI (`aws configure`)
    - Terraform 1.5.0 or higher

2. **Domain Configuration**
    - Domain registered (al-meisan.io)
    - Route53 hosted zone created
    - Nameservers updated with registrar

3. **Required Tools**
   ```bash
   # Install Terraform
   brew install terraform
   
   # Install AWS CLI
   brew install awscli
   
   # Install additional tools
   brew install jq tfsec checkov
   ```

### Deployment Steps

1. **Initialize Backend**
   ```bash
   cd clinic-infra
   ./scripts/init-backend.sh prod
   ```

2. **Deploy Infrastructure**
   ```bash
   # Deploy to development
   ./scripts/deploy.sh dev apply
   
   # Deploy to production (requires approval)
   ./scripts/deploy.sh prod apply
   ```

3. **Validate Deployment**
   ```bash
   ./scripts/validate.sh prod
   ./scripts/validate-backend-integration.sh prod https://api.medflow.al-meisan.io
   ```

## Architecture

### High-Level Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│   CloudFront    │────▶│  ALB (Layer 7)  │────▶│   ECS Fargate   │
│     (CDN)       │     │  Load Balancer  │     │   (Backend)     │
│                 │     │                 │     │                 │
└────────┬────────┘     └─────────────────┘     └────────┬────────┘
         │                                                │
         │                                                │
         ▼                                                ▼
┌─────────────────┐                            ┌─────────────────┐
│                 │                            │                 │
│   S3 Bucket     │                            │  RDS PostgreSQL │
│ (Static Assets) │                            │   (Database)    │
│                 │                            │                 │
└─────────────────┘                            └─────────────────┘
         │                                                │
         │              ┌─────────────────┐               │
         └──────────────│                 │───────────────┘
                        │  AWS Cognito    │
                        │ (Authentication)│
                        │                 │
                        └─────────────────┘
```

### Network Architecture

- **VPC CIDR**: 10.0.0.0/16
- **Public Subnets**: 10.0.1.0/24, 10.0.2.0/24 (ALB)
- **Private Subnets**: 10.0.10.0/24, 10.0.11.0/24 (ECS)
- **Database Subnets**: 10.0.20.0/24, 10.0.21.0/24 (RDS)

## Environment Configuration

### Development Environment

- Cost-optimized with single NAT gateway
- Minimal redundancy for cost savings
- Ideal for testing and development

### Staging Environment

- Production-like configuration
- Full redundancy across AZs
- Performance testing capable

### Production Environment

- High availability across multiple AZs
- Auto-scaling enabled
- Enhanced monitoring and alerting
- Automated backups and disaster recovery

## Security

### Security Features

- **Encryption**: At rest and in transit
- **Network Security**: Security groups and NACLs
- **Access Control**: IAM roles and policies
- **Compliance**: HIPAA-ready architecture
- **Monitoring**: CloudWatch and AWS GuardDuty

### Security Best Practices

1. Regular security scanning with tfsec and checkov
2. Automated vulnerability scanning in CI/CD
3. Least privilege IAM policies
4. Regular key rotation
5. Audit logging enabled

## Monitoring and Alerting

### CloudWatch Dashboards

- Infrastructure health overview
- Application performance metrics
- Cost tracking and optimization
- Security event monitoring

### Alerts Configuration

- CPU/Memory utilization
- Error rates and latency
- Failed deployments
- Security events
- Cost threshold breaches

## Disaster Recovery

### Backup Strategy

- **RDS**: Automated daily snapshots (7-day retention)
- **S3**: Cross-region replication for critical data
- **ECS**: Container images in ECR with versioning
- **Configuration**: Terraform state in S3 with versioning

### Recovery Procedures

1. **Database Recovery**: Restore from snapshot
2. **Application Recovery**: Redeploy from ECR
3. **Static Assets**: Restore from S3 versioning
4. **Full DR**: Multi-region failover capability

## Cost Optimization

### Cost Management

- Environment-based resource sizing
- Auto-scaling for demand-based capacity
- Reserved instances for production
- S3 lifecycle policies
- CloudWatch cost alerts

### Monthly Cost Estimates

- **Development**: ~$150-200
- **Staging**: ~$400-500
- **Production**: ~$800-1200

## Troubleshooting

### Common Issues

1. **Deployment Failures**
   ```bash
   # Check Terraform state
   terraform state list
   
   # Validate configuration
   terraform validate
   
   # Check AWS credentials
   aws sts get-caller-identity
   ```

2. **Container Health Issues**
   ```bash
   # Check ECS service events
   aws ecs describe-services --cluster prod-medflow-cluster --services prod-medflow-backend
   
   # View container logs
   aws logs tail /ecs/medflow-backend --follow
   ```

3. **Database Connectivity**
   ```bash
   # Test RDS connectivity
   ./scripts/test-database-connection.sh prod
   
   # Check security groups
   aws ec2 describe-security-groups --group-ids sg-xxxxxxxxx
   ```

## CI/CD Integration

### GitHub Actions Workflows

- **Infrastructure**: Automated Terraform deployments
- **Backend**: Container build and ECS deployment
- **Frontend**: Static asset deployment to S3/CloudFront
- **Security**: Automated scanning and compliance checks

### Deployment Pipeline

1. Code commit triggers build
2. Automated testing and security scanning
3. Container build and push to ECR
4. Infrastructure validation
5. Progressive deployment with health checks
6. Automated rollback on failure

## Support and Maintenance

### Regular Maintenance Tasks

- [ ] Weekly security patches
- [ ] Monthly cost review
- [ ] Quarterly DR testing
- [ ] Annual security audit

### Getting Help

- **Documentation**: See `/docs` directory
- **Issues**: GitHub Issues
- **Emergency**: On-call rotation schedule

## License

Copyright © 2024 MedFlow Healthcare Platform. All rights reserved.

```

## FUNCTIONAL REQUIREMENTS

1. **Comprehensive Coverage**
   - All infrastructure components documented
   - Step-by-step deployment procedures
   - Troubleshooting for common scenarios
   - Disaster recovery procedures

2. **Healthcare Focus**
   - HIPAA compliance considerations
   - Data privacy procedures
   - Audit trail documentation
   - Incident response plans

3. **Operational Excellence**
   - Monitoring setup guides
   - Performance tuning procedures
   - Scaling runbooks
   - Cost optimization strategies

4. **Knowledge Transfer**
   - Onboarding guides for new team members
   - Architecture decision records
   - Best practices documentation
   - Lessons learned repository

## TESTING REQUIREMENTS

**Documentation Validation:**
- Procedure accuracy testing
- Command execution validation
- Link and reference checking
- Diagram accuracy verification

**Operational Testing:**
- Runbook effectiveness
- Recovery procedure validation
- Monitoring alert accuracy
- Incident response drills