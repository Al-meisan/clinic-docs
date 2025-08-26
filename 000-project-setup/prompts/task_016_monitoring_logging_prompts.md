# PROMPTS: TASK-016 - Monitoring and Logging Setup

# PROMPT 1: CloudWatch Infrastructure Setup and Basic Metrics

Create AWS CloudWatch infrastructure configuration with Terraform for monitoring EC2, RDS, and Lambda metrics, including custom application metrics publishing from the NestJS backend application.

## PROJECT CONTEXT

**Tech Stack:** NestJS backend with TypeScript, AWS infrastructure managed via Terraform, PostgreSQL database, React frontend  
**Architecture:** Domain-driven design with feature-based modules, separation of concerns between UI, business logic, and data access  
**Current State:** Backend NestJS application is operational with database connectivity, AWS infrastructure foundation is established

## EPIC CONTEXT

Project setup and foundational infrastructure epic focusing on production readiness, operational monitoring, and system reliability for healthcare environment operations.

## TASK CONTEXT

**Task Type:** INFRASTRUCTURE  
**Complexity:** HIGH  
**Goal:** Configure comprehensive monitoring infrastructure for production-ready clinic management system  
**Business Value:** Essential for troubleshooting, performance optimization, maintaining system reliability in healthcare environment

## BUSINESS DOMAIN CONTEXT

- Healthcare system availability is critical for patient care operations
- System performance directly impacts patient care workflows  
- Compliance requirements mandate monitoring and audit capabilities
- Infrastructure monitoring guides capacity planning for clinic growth

## FUNCTIONAL REQUIREMENTS

1. **AWS CloudWatch Integration**
   - Set up CloudWatch monitoring for EC2 instances, RDS database, and Lambda functions
   - Configure custom dashboards displaying key infrastructure metrics
   - Implement automatic metric collection for CPU, memory, disk, and network usage

2. **Custom Application Metrics**
   - Publish custom application metrics from NestJS backend to CloudWatch
   - Track API response times, request counts, and error rates
   - Implement metric batching to optimize CloudWatch API calls

## TECHNICAL REQUIREMENTS

**Performance:**
- Metric publishing latency < 30 seconds for real-time monitoring
- Dashboard load time < 5 seconds for responsive monitoring interface
- Batch metrics to prevent CloudWatch API throttling

**Security:**
- Use IAM roles and policies for CloudWatch access
- Encrypt metrics data in transit and at rest
- Secure API keys and AWS credentials using environment variables

**Infrastructure:**
- Terraform configuration for CloudWatch log groups and dashboards
- Environment-specific metric namespaces (dev, staging, production)
- Cost-effective CloudWatch usage with appropriate retention policies

## TESTING REQUIREMENTS

**Unit Tests:**
- Test metrics service methods for proper metric formatting
- Test error handling when CloudWatch service is unavailable
- Test metric batching logic and throttling prevention

**Integration Tests:**
- Test CloudWatch integration in AWS test environment
- Verify metrics appear in CloudWatch within expected timeframe
- Test dashboard loading and metric visualization

**Manual Tests:**
- Verify infrastructure metrics are collected automatically
- Test custom metric publishing from application
- Confirm dashboards display correctly across environments

## DOCUMENTATION REQUIREMENTS

- Update infrastructure documentation with CloudWatch configuration
- Document custom metrics being published and their meaning
- Create operational runbook for CloudWatch dashboard access and interpretation

## VALIDATION CRITERIA

- [ ] Terraform configuration creates CloudWatch log groups and dashboards
- [ ] Infrastructure metrics (CPU, memory, disk, network) automatically collected for all resources
- [ ] Custom application metrics successfully published from NestJS backend
- [ ] CloudWatch dashboards display key metrics with proper visualization
- [ ] Metric publishing handles errors gracefully with local fallback logging
- [ ] IAM roles configured with least-privilege access to CloudWatch
- [ ] All environments (dev, staging, production) have separate metric namespaces
- [ ] Unit tests achieve >= 80% coverage for metrics service
- [ ] Integration tests pass in AWS test environment
- [ ] Documentation updated with monitoring configuration details

# PROMPT 2: Application Performance Monitoring Implementation

Implement comprehensive application performance monitoring (APM) system with response time tracking, throughput measurement, and performance baseline establishment for the NestJS backend application.

## PROJECT CONTEXT

**Tech Stack:** NestJS backend with TypeScript, PostgreSQL with TypeORM, AWS CloudWatch for metrics  
**Code Standards:** Domain-driven design, structured logging with correlation IDs, consistent error handling  
**Current State:** Basic CloudWatch infrastructure is configured, NestJS application with authentication and database operations

## EPIC CONTEXT

Project setup epic focused on production readiness and operational monitoring for healthcare system reliability and performance optimization.

## TASK CONTEXT

**Task Type:** BACKEND  
**Complexity:** HIGH  
**Goal:** Implement application-level monitoring for response times, throughput, database query performance, and user experience metrics  
**Business Value:** Enable proactive performance optimization and ensure healthcare workflows remain responsive

## BUSINESS DOMAIN CONTEXT

- Response time directly impacts patient care workflow efficiency
- Database query performance affects appointment scheduling and patient record access
- API throughput monitoring ensures system can handle clinic operational load
- Performance degradation detection prevents service disruptions during patient care

## FUNCTIONAL REQUIREMENTS

1. **API Response Time Monitoring**
   - Track response times for all API endpoints with percentile calculations (p50, p95, p99)
   - Identify slow endpoints and performance bottlenecks
   - Monitor response time trends and detect performance degradation

2. **Database Query Performance Tracking**
   - Monitor TypeORM query execution times with query identification
   - Track database connection pool utilization and wait times
   - Identify N+1 query problems and inefficient database operations

3. **Throughput and Load Monitoring**
   - Track requests per second across all endpoints
   - Monitor concurrent user sessions and active connections
   - Measure application resource utilization under load

## TECHNICAL REQUIREMENTS

**Performance:**
- Performance monitoring overhead < 5ms per request
- Real-time metric aggregation and publishing
- Efficient memory usage for performance data collection

**Implementation:**
- NestJS interceptor for request/response time measurement
- TypeORM query logging and performance analysis
- Custom middleware for throughput and load tracking
- Integration with existing CloudWatch metrics infrastructure

**Data Collection:**
- Correlation IDs for request tracing across services
- Performance data aggregation with configurable time windows
- Baseline performance establishment for comparison and alerting

## TESTING REQUIREMENTS

**Unit Tests:**
- Test performance interceptor timing accuracy
- Test query performance logging with mock TypeORM
- Test metric aggregation and calculation logic
- Test performance data formatting for CloudWatch

**Integration Tests:**
- Test end-to-end performance monitoring in test environment
- Verify performance metrics are published to CloudWatch
- Test performance monitoring under simulated load conditions

**Load Tests:**
- Performance monitoring accuracy under high request volume
- Verify monitoring overhead remains within acceptable limits
- Test metric aggregation performance with concurrent requests

## DOCUMENTATION REQUIREMENTS

- Document performance monitoring architecture and data flow
- Create performance baseline documentation for different endpoint types
- Update API documentation with performance monitoring implementation
- Document performance metric interpretation and troubleshooting guide

## VALIDATION CRITERIA

- [ ] NestJS interceptor measures and logs response times for all API endpoints
- [ ] Performance metrics include percentile calculations (p50, p95, p99)
- [ ] Database query performance tracking integrated with TypeORM
- [ ] Throughput metrics (requests/second) tracked and published to CloudWatch
- [ ] Correlation IDs implemented for request tracing and performance analysis
- [ ] Performance monitoring overhead measured and stays below 5ms per request
- [ ] Performance baselines established for critical API endpoints
- [ ] CloudWatch dashboards display application performance metrics
- [ ] Unit tests achieve >= 80% coverage for performance monitoring components
- [ ] Integration tests verify performance metrics accuracy
- [ ] Load tests confirm monitoring scalability under high volume
- [ ] Documentation includes performance monitoring architecture and baseline interpretation

# PROMPT 3: Centralized Logging with Structured JSON Logging

Implement centralized logging system using Winston with CloudWatch transport, structured JSON logging format, and correlation ID tracking across the NestJS backend application and React frontend.

## PROJECT CONTEXT

**Tech Stack:** NestJS backend with TypeScript, React frontend, AWS CloudWatch Logs, Winston logger  
**Architecture:** Domain-driven design with feature modules, consistent error handling patterns  
**Current State:** Basic CloudWatch infrastructure configured, NestJS application operational with authentication

## EPIC CONTEXT

Project setup epic focused on operational monitoring, troubleshooting capabilities, and production readiness for healthcare system management.

## TASK CONTEXT

**Task Type:** FULLSTACK  
**Complexity:** HIGH  
**Goal:** Aggregate structured logs from all application components with searchable centralized storage and proper log levels  
**Business Value:** Enable rapid troubleshooting, audit trail for healthcare compliance, and operational visibility

## BUSINESS DOMAIN CONTEXT

- Healthcare compliance requires comprehensive audit logging of data access events
- Patient data access must be tracked and logged for regulatory compliance
- Troubleshooting system issues must not disrupt patient care operations
- Log data supports incident analysis and system improvement initiatives

## FUNCTIONAL REQUIREMENTS

1. **Backend Structured Logging**
   - Implement Winston logger with CloudWatch transport in NestJS
   - Use JSON format for all log entries with consistent structure
   - Include correlation IDs for request tracing across services
   - Log levels: ERROR, WARN, INFO, DEBUG with appropriate usage

2. **Frontend Error Logging**
   - Implement React Error Boundary with structured error reporting
   - Capture frontend errors with context (user agent, URL, stack trace)
   - Send frontend errors to backend logging endpoint

3. **Log Correlation and Context**
   - Generate unique correlation IDs for each request/user session
   - Include user context, patient context, and appointment context in logs
   - Maintain correlation across frontend and backend operations

## TECHNICAL REQUIREMENTS

**Log Structure:**
- Consistent JSON format: timestamp, level, message, service, correlationId, context
- Structured context data for healthcare operations (patientId, appointmentId, userId)
- Stack traces for errors with sanitized sensitive information

**Performance:**
- Asynchronous log processing to prevent blocking application operations
- Log batching for CloudWatch transport to optimize API calls
- Configurable log levels per environment (DEBUG in dev, INFO+ in production)

**Security:**
- Sanitize sensitive information (PII, passwords, tokens) from logs
- Secure transport to CloudWatch with proper IAM permissions
- Log retention policies per environment compliance requirements

## TESTING REQUIREMENTS

**Unit Tests:**
- Test Winston logger configuration and transport setup
- Test log formatting and structured data inclusion
- Test correlation ID generation and propagation
- Test sensitive data sanitization in log messages

**Integration Tests:**
- Test log aggregation in CloudWatch from both frontend and backend
- Verify correlation ID tracking across frontend-backend requests
- Test log searchability and filtering in CloudWatch Logs

**Manual Tests:**
- Verify log entries appear in CloudWatch within expected timeframe
- Test log correlation across user workflows (login, patient access, appointments)
- Confirm sensitive data is properly sanitized in production logs

## DOCUMENTATION REQUIREMENTS

- Document logging architecture and log format specifications
- Create logging best practices guide for development team
- Document CloudWatch Logs access and searching procedures
- Update troubleshooting guide with log analysis techniques

## VALIDATION CRITERIA

- [ ] Winston logger configured with CloudWatch transport in NestJS application
- [ ] All log entries use consistent JSON structure with required fields
- [ ] Correlation IDs generated and tracked across frontend-backend requests
- [ ] React Error Boundary captures and reports frontend errors with context
- [ ] Log levels properly configured per environment (DEBUG/INFO/WARN/ERROR)
- [ ] Sensitive information (PII, passwords, tokens) sanitized from all log entries
- [ ] Logs aggregated and searchable in CloudWatch Logs across all services
- [ ] Log retention policies configured per environment requirements
- [ ] Asynchronous logging prevents blocking application operations
- [ ] Unit tests achieve >= 80% coverage for logging components
- [ ] Integration tests verify end-to-end log correlation functionality
- [ ] Documentation includes logging architecture and troubleshooting procedures

# PROMPT 4: Error Tracking and Alert System

Implement automated error detection, tracking, and notification system with CloudWatch Alarms, SNS notifications, and custom error dashboard for real-time error monitoring and alerting.

## PROJECT CONTEXT

**Tech Stack:** NestJS backend, React frontend, AWS CloudWatch, SNS for notifications  
**Current State:** Structured logging system operational, CloudWatch infrastructure configured  
**Architecture:** Centralized error handling, consistent error reporting patterns across application

## EPIC CONTEXT

Project setup epic focused on operational monitoring, proactive issue detection, and system reliability for healthcare operations.

## TASK CONTEXT

**Task Type:** FULLSTACK  
**Complexity:** HIGH  
**Goal:** Automated error detection and notification system providing real-time alerts with actionable information  
**Business Value:** Minimize system downtime impact on patient care, enable rapid incident response

## BUSINESS DOMAIN CONTEXT

- System errors during patient care operations require immediate attention
- Healthcare providers need reliable system availability for patient safety
- Error trends indicate system health and guide capacity planning
- Automated alerting ensures 24/7 monitoring without manual oversight

## FUNCTIONAL REQUIREMENTS

1. **CloudWatch Alarms Configuration**
   - Set up multi-threshold alarms for error rates, response times, and system health
   - Configure alarm states (OK, ALARM, INSUFFICIENT_DATA) with appropriate thresholds
   - Implement composite alarms combining multiple metrics for comprehensive monitoring

2. **SNS Notification System**
   - Configure SNS topics for different alert severity levels (critical, warning, info)
   - Set up email, SMS, and webhook notifications for development team
   - Implement notification routing based on error type and severity

3. **Custom Error Dashboard**
   - Create centralized error monitoring dashboard with real-time error metrics
   - Display error rate trends, top errors, and resolution tracking
   - Include error context and correlation for rapid troubleshooting

## TECHNICAL REQUIREMENTS

**Alerting Logic:**
- Error rate thresholds: > 5% errors for WARNING, > 10% for CRITICAL
- Response time thresholds: > 2s average for WARNING, > 5s for CRITICAL
- Database connection failures trigger immediate CRITICAL alerts
- Frontend JavaScript errors aggregated and tracked

**Notification Configuration:**
- Critical alerts: immediate notification via multiple channels
- Warning alerts: batched notifications every 15 minutes
- Info alerts: daily summary reports
- Alert fatigue prevention with intelligent grouping and throttling

**Error Tracking:**
- Error fingerprinting for duplicate detection and grouping
- Error context capture (user session, request details, system state)
- Resolution tracking and post-incident analysis data

## TESTING REQUIREMENTS

**Unit Tests:**
- Test error detection and classification logic
- Test alert threshold calculations and trigger conditions
- Test notification formatting and routing logic
- Test error dashboard data aggregation and display

**Integration Tests:**
- Test CloudWatch alarm triggering with simulated error conditions
- Test SNS notification delivery across all configured channels
- Test error dashboard real-time updates during error scenarios
- Test alert escalation and resolution workflows

**Manual Tests:**
- Trigger test errors to verify end-to-end alerting pipeline
- Verify notification delivery to all configured recipients
- Test alert fatigue prevention during sustained error conditions
- Confirm error resolution tracking and dashboard updates

## DOCUMENTATION REQUIREMENTS

- Create alerting runbook with response procedures for different error types
- Document alert thresholds and escalation procedures
- Update operational documentation with error dashboard access and interpretation
- Create incident response guide with error tracking and resolution procedures

## VALIDATION CRITERIA

- [ ] CloudWatch alarms configured with appropriate thresholds for error rates and response times
- [ ] SNS topics configured for critical, warning, and info alert levels
- [ ] Multi-channel notifications (email, SMS) configured and tested
- [ ] Custom error dashboard displays real-time error metrics and trends
- [ ] Error fingerprinting groups similar errors and prevents duplicate alerts
- [ ] Alert fatigue prevention with intelligent grouping and throttling implemented
- [ ] Frontend JavaScript errors captured and integrated into alerting system
- [ ] Database connection and critical system errors trigger immediate notifications
- [ ] Error context and correlation information included in alerts for troubleshooting
- [ ] Unit tests achieve >= 80% coverage for error tracking and alerting components
- [ ] Integration tests verify end-to-end alert triggering and notification delivery
- [ ] Alerting runbook created with incident response procedures
- [ ] Error dashboard provides actionable information for rapid issue resolution

# PROMPT 5: Health Check Monitoring System

Implement comprehensive health check monitoring system for all critical services and dependencies with automated failover detection, service availability monitoring, and integration with existing alerting infrastructure.

## PROJECT CONTEXT

**Tech Stack:** NestJS backend, PostgreSQL database, AWS infrastructure, existing CloudWatch monitoring and alerting  
**Current State:** Error tracking and alerting system operational, CloudWatch infrastructure configured  
**Architecture:** Domain-driven design with service dependencies, database connectivity, external service integrations

## EPIC CONTEXT

Project setup epic focused on system reliability, proactive monitoring, and ensuring healthcare system availability for patient care operations.

## TASK CONTEXT

**Task Type:** BACKEND  
**Complexity:** MEDIUM  
**Goal:** Continuous health monitoring of all critical services with automatic failover alerts and dependency health checks  
**Business Value:** Prevent service disruptions during patient care, enable proactive maintenance and capacity planning

## BUSINESS DOMAIN CONTEXT

- Healthcare system availability is critical for patient safety and care continuity
- Service dependencies (database, authentication, external APIs) must be monitored
- Proactive health monitoring prevents patient care workflow interruptions  
- Health trends guide infrastructure scaling and maintenance scheduling

## FUNCTIONAL REQUIREMENTS

1. **Service Health Endpoints**
   - Implement /health endpoint with comprehensive service status checks
   - Check database connectivity, response times, and connection pool status
   - Monitor external service dependencies (AWS Cognito, third-party APIs)
   - Include detailed health information with service-specific metrics

2. **Automated Health Monitoring**
   - Continuous health checks with configurable intervals (30s for critical, 5min for non-critical)
   - Health status tracking with historical data and trend analysis
   - Integration with existing CloudWatch alarms for health status changes

3. **Dependency Chain Monitoring**
   - Monitor cascading dependency health (database → authentication → API availability)
   - Identify single points of failure and service bottlenecks
   - Track service recovery times and availability percentages

## TECHNICAL REQUIREMENTS

**Health Check Implementation:**
- NestJS health check module with custom health indicators
- Database health checks with connection pool monitoring
- Memory and CPU utilization checks for application health
- External service health verification with timeout handling

**Monitoring Integration:**
- Health metrics published to CloudWatch for historical tracking
- Integration with existing alerting system for health status changes
- Health dashboard integration showing service topology and status

**Performance:**
- Health check endpoints respond within 1 second for rapid monitoring
- Minimal impact on application performance during health checks
- Efficient health data collection without affecting critical operations

## TESTING REQUIREMENTS

**Unit Tests:**
- Test health check indicators for all monitored services
- Test health endpoint response formatting and timing
- Test failure detection and recovery state tracking
- Test health metric calculation and reporting accuracy

**Integration Tests:**
- Test health monitoring under simulated failure conditions
- Test health check integration with CloudWatch metrics publishing
- Test alert triggering when health checks fail or recover
- Test health dashboard updates during service state changes

**Manual Tests:**
- Verify health endpoints provide accurate service status information
- Test health monitoring during planned maintenance scenarios
- Confirm health alerts integrate properly with existing notification system
- Validate health dashboard accuracy during various service states

## DOCUMENTATION REQUIREMENTS

- Document health check architecture and monitored services
- Create health monitoring runbook with interpretation guidelines
- Update operational procedures with health check integration
- Document service dependency mapping and critical path identification

## VALIDATION CRITERIA

- [ ] /health endpoint implemented with comprehensive service status checks
- [ ] Database connectivity and performance monitoring integrated
- [ ] External service dependency health verification (AWS Cognito, APIs) implemented
- [ ] Automated health monitoring runs continuously with appropriate intervals
- [ ] Health metrics published to CloudWatch for historical tracking and alerting
- [ ] Health status changes trigger alerts through existing notification system
- [ ] Health check endpoints respond within 1 second for rapid monitoring
- [ ] Service dependency chain monitoring identifies critical path failures
- [ ] Health dashboard displays service topology and real-time status information
- [ ] Health check implementation has minimal performance impact on application
- [ ] Unit tests achieve >= 80% coverage for health monitoring components
- [ ] Integration tests verify health monitoring under failure and recovery scenarios
- [ ] Health monitoring runbook created with service interpretation guidelines
- [ ] Service availability percentages tracked and reported for capacity planning