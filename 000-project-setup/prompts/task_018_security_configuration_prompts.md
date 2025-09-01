# PROMPTS: TASK-018 - Security Configuration

# PROMPT 1: Security Headers Middleware Implementation

Implement comprehensive HTTP security headers middleware for the NestJS backend application to protect against common web vulnerabilities including XSS, clickjacking, and content type sniffing attacks.

## PROJECT CONTEXT

**Tech Stack**: NestJS with TypeScript backend following domain-driven design architecture patterns.

**Current Backend Structure**: 
- NestJS application with controller, service, repository pattern
- Folder structure following lowercase with dashes naming convention
- Custom middleware and guards architecture already established
- Error handling and logging infrastructure in place

## EPIC CONTEXT

**Business Requirements**: Healthcare application requiring healthcare privacy compliance and comprehensive security protection for sensitive patient data. Zero trust security model implementation with defense in depth strategy.

**Integration Points**: Security middleware must integrate with existing authentication system (JWT authentication) and apply to all API routes consistently.

## TASK CONTEXT

**Functional Requirement**: Implement comprehensive HTTP security headers for all responses including CSP, HSTS, X-Frame-Options, X-Content-Type-Options, and other OWASP-recommended headers.

**Technical Requirement**: Security checks must add less than 50ms latency per request while providing comprehensive protection against OWASP Top 10 vulnerabilities.

## BUSINESS DOMAIN CONTEXT

**Healthcare Data Protection**: Patient information requires strict security controls with audit trails. All HTTP responses must include appropriate security headers to prevent client-side attacks and ensure data protection compliance.

**Compliance Requirements**: healthcare privacy compliance mandates comprehensive security controls including protection against common web vulnerabilities through appropriate HTTP security headers.

## FUNCTIONAL REQUIREMENTS

1. **Content Security Policy (CSP)**: Implement strict CSP with nonce-based script execution to prevent XSS attacks
2. **HSTS Configuration**: Enforce HTTPS connections with 1-year max-age and includeSubDomains
3. **Anti-Clickjacking**: Configure X-Frame-Options to DENY for all pages
4. **MIME Type Protection**: Implement X-Content-Type-Options nosniff for all responses
5. **Referrer Policy**: Configure strict-origin-when-cross-origin referrer policy

## TECHNICAL REQUIREMENTS

**Implementation Pattern**:
```typescript
// Expected middleware structure
@Injectable()
export class SecurityHeadersMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Configure Helmet with comprehensive security headers
    // Apply CSP with environment-specific configuration
    // Set HSTS with proper configuration
    // Apply all security headers consistently
  }
}
```

**Configuration Requirements**:
- Environment-specific CSP policies (stricter for production)
- Support for dynamic nonce generation for scripts
- Error response security headers inclusion
- Performance optimization for header processing

## TESTING REQUIREMENTS

**Unit Tests**:
- Test security headers are applied to all responses
- Test CSP nonce generation and application
- Test HSTS header configuration
- Test error response header inclusion
- Test middleware integration with existing guard system

**Integration Tests**:
- Test security headers in actual HTTP responses
- Test CSP policy enforcement in browser
- Test HSTS enforcement behavior
- Verify no performance degradation beyond 50ms requirement

**Security Tests**:
- Penetration testing for XSS prevention
- Clickjacking protection verification
- MIME type sniffing prevention testing

## DOCUMENTATION REQUIREMENTS

- Update security middleware documentation
- Document CSP policy configuration and customization
- Create security header troubleshooting guide
- Update deployment documentation with security header verification steps

## VALIDATION CRITERIA

- [ ] All HTTP responses include comprehensive security headers (CSP, HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy)
- [ ] CSP policy configured with nonce-based script execution and environment-specific settings
- [ ] HSTS configured with 1-year max-age and includeSubDomains
- [ ] Security headers applied to error responses and all application endpoints
- [ ] Middleware performance impact verified to be under 50ms per request
- [ ] Unit tests achieve >= 95% coverage for security header functionality
- [ ] Integration tests verify browser security enforcement
- [ ] Security review completed with no critical vulnerabilities identified
- [ ] Documentation updated with implementation details and troubleshooting guide

---

# PROMPT 2: CORS Configuration Setup

Implement secure Cross-Origin Resource Sharing (CORS) configuration for the NestJS backend with environment-specific origin whitelisting and restrictive policy enforcement to prevent unauthorized cross-origin access.

## PROJECT CONTEXT

**Tech Stack**: NestJS with TypeScript backend, React frontend with TypeScript, backend infrastructure supporting multi-environment deployment (dev, staging, production).

**Current Application State**: 
- NestJS backend with API endpoints established
- Environment configuration system in place
- Frontend React application consuming backend APIs
- Authentication system using JWT tokens

## EPIC CONTEXT

**Business Requirements**: Healthcare application requiring strict access controls and healthcare privacy compliance. CORS policies must prevent unauthorized access while supporting legitimate frontend applications.

**Security Model**: Zero trust security model requiring explicit authorization for all cross-origin requests with comprehensive logging of access attempts.

## TASK CONTEXT

**Functional Requirement**: Configure Cross-Origin Resource Sharing policies for API security with whitelist-based origin control and environment-specific configurations.

**Technical Requirement**: CORS policy must allow only authorized domains while supporting necessary headers and methods for application functionality.

## BUSINESS DOMAIN CONTEXT

**Healthcare Security Requirements**: Patient data APIs require strict access controls to prevent unauthorized access from malicious websites or applications.

**Multi-Environment Support**: Different CORS policies required for development (localhost), staging (test domains), and production (production domains only).

## FUNCTIONAL REQUIREMENTS

1. **Origin Whitelisting**: Implement environment-specific allowed origins configuration
2. **Method Restriction**: Configure minimum required HTTP methods only
3. **Header Control**: Allow essential headers only, prevent unnecessary header exposure
4. **Credentials Handling**: Secure handling of credentials in cross-origin requests
5. **Preflight Optimization**: Efficient preflight request handling for performance

## TECHNICAL REQUIREMENTS

**Configuration Structure**:
```typescript
// Expected CORS configuration
export const corsConfig: CorsOptions = {
  origin: (origin, callback) => {
    // Environment-specific origin validation
    // Development vs production origin handling
    // Logging of allowed/denied origins
  },
  credentials: true, // For authentication cookies/tokens
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: ['Content-Type', 'Authorization', 'X-Requested-With'],
  maxAge: 86400 // 24 hours for preflight caching
};
```

**Environment-Specific Origins**:
- Development: localhost:3000, localhost:5173 (Vite dev server)
- Staging: staging domain(s)
- Production: production domain(s) only

## TESTING REQUIREMENTS

**Unit Tests**:
- Test origin validation logic for each environment
- Test allowed/denied origins processing
- Test CORS headers in responses
- Test preflight request handling

**Integration Tests**:
- Test cross-origin requests from allowed origins
- Test blocked requests from unauthorized origins  
- Test preflight OPTIONS requests
- Verify CORS headers in actual HTTP responses

**Security Tests**:
- Test CORS bypass attempt prevention
- Verify unauthorized origin blocking
- Test credential handling security

## DOCUMENTATION REQUIREMENTS

- Document CORS configuration per environment
- Create troubleshooting guide for CORS issues
- Update deployment documentation with origin management
- Document CORS policy rationale and security implications

## VALIDATION CRITERIA

- [ ] CORS policy enforced with whitelist-based origin control
- [ ] Environment-specific origin configuration implemented (dev, staging, production)
- [ ] Only necessary HTTP methods and headers allowed
- [ ] Unauthorized origins properly blocked with appropriate logging
- [ ] Preflight requests handled efficiently with proper caching
- [ ] Credentials handling secured for authentication flows
- [ ] Unit tests achieve >= 95% coverage for CORS functionality
- [ ] Integration tests verify CORS enforcement in browser
- [ ] Security review confirms no CORS bypass vulnerabilities
- [ ] Documentation updated with configuration and troubleshooting details

---

# PROMPT 3: Rate Limiting System Implementation

Implement comprehensive rate limiting system for the NestJS backend using Redis for distributed rate limiting with per-user, per-IP, and per-endpoint limits to prevent abuse and DDoS attacks.

## PROJECT CONTEXT

**Tech Stack**: NestJS with TypeScript backend, Redis for caching and session storage, backend infrastructure with auto-scaling capabilities.

**Current Infrastructure**: 
- NestJS application with guard and interceptor architecture
- Redis integration established for session management
- Authentication system providing user context
- Monitoring and logging infrastructure in place

## EPIC CONTEXT

**Business Requirements**: Healthcare application requiring protection against abuse and ensuring service availability for critical patient care workflows.

**Performance Requirements**: Rate limiting decisions must be made within 10ms to maintain application responsiveness while providing robust protection.

## TASK CONTEXT

**Functional Requirement**: Implement rate limiting to prevent abuse and DDoS attacks with per-user and per-endpoint rate limiting with proper error responses.

**Technical Requirement**: Rate limiting response time < 10ms with comprehensive protection against various attack patterns.

## BUSINESS DOMAIN CONTEXT

**Healthcare Service Availability**: Critical patient care workflows require consistent system availability. Rate limiting must prevent abuse while never blocking legitimate medical operations.

**User Role Considerations**: Different user roles (providers, staff, patients) may require different rate limit thresholds based on their typical usage patterns.

## FUNCTIONAL REQUIREMENTS

1. **Global Rate Limiting**: 1000 requests per hour per IP address for DDoS protection
2. **API Endpoint Limiting**: Specific limits per endpoint type (authentication: 10/min, search: 100/hour)
3. **User-based Limiting**: Per-user quotas based on role with role-specific thresholds
4. **Rate Limit Headers**: Proper HTTP headers indicating remaining quota and reset time
5. **Bypass Mechanisms**: Emergency bypass capability for critical healthcare operations

## TECHNICAL REQUIREMENTS

**Implementation Structure**:
```typescript
// Expected rate limiting architecture
@Injectable()
export class RateLimitingGuard implements CanActivate {
  constructor(
    private redis: Redis,
    private configService: ConfigService
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    // IP-based rate limiting
    // User-based rate limiting (if authenticated)
    // Endpoint-specific rate limiting
    // Redis-based distributed tracking
    // Rate limit header application
  }
}

// Rate limit configuration
interface RateLimitConfig {
  global: { requests: number; window: number }; // per IP
  endpoints: Record<string, { requests: number; window: number }>;
  users: Record<string, { requests: number; window: number }>; // by role
}
```

**Redis Integration**:
- Distributed rate limit counters using Redis
- Sliding window or fixed window algorithm
- Efficient Redis operations for performance
- Proper key expiration and cleanup

## TESTING REQUIREMENTS

**Unit Tests**:
- Test rate limit calculation and enforcement logic
- Test Redis integration and counter management
- Test rate limit header generation
- Test different rate limit policies per user role
- Test bypass mechanism functionality

**Integration Tests**:
- Test rate limiting across multiple requests
- Test distributed rate limiting with multiple application instances
- Test rate limit enforcement with concurrent requests
- Verify rate limit headers in HTTP responses

**Performance Tests**:
- Verify rate limiting decision time < 10ms requirement
- Test Redis performance under high load
- Load testing with rate limit enforcement

**Security Tests**:
- Test DDoS protection effectiveness
- Test rate limit bypass attempt prevention
- Verify no memory leaks in rate limit tracking

## DOCUMENTATION REQUIREMENTS

- Document rate limiting policies and thresholds
- Create rate limit configuration guide for different environments
- Update API documentation with rate limit information
- Document emergency bypass procedures for critical operations

## VALIDATION CRITERIA

- [ ] Per-user, per-IP, and per-endpoint rate limiting implemented
- [ ] Redis-based distributed rate limiting operational
- [ ] Rate limiting decisions completed within 10ms requirement
- [ ] Proper HTTP 429 responses with retry-after headers
- [ ] Role-based rate limiting thresholds configured appropriately
- [ ] Emergency bypass mechanism implemented for critical operations
- [ ] Rate limit headers included in all responses
- [ ] Unit tests achieve >= 95% coverage for rate limiting functionality
- [ ] Integration tests verify distributed rate limiting behavior
- [ ] Performance tests confirm < 10ms response time requirement
- [ ] Security review validates DDoS protection effectiveness
- [ ] Documentation updated with rate limiting policies and procedures

---

# PROMPT 4: Input Validation Framework Implementation

Implement comprehensive input validation and sanitization framework for all API endpoints using class-validator, custom decorators, and XSS prevention to ensure all user inputs are properly validated and sanitized before processing.

## PROJECT CONTEXT

**Tech Stack**: NestJS with TypeScript backend using DTOs with class-validator and class-transformer, existing controller and service architecture.

**Current Application State**:
- NestJS application with DTO pattern established
- Controller endpoints consuming request DTOs
- class-validator integration partially implemented
- Global exception filter for error handling

## EPIC CONTEXT

**Business Requirements**: Healthcare application handling sensitive patient data requiring comprehensive input validation to prevent injection attacks and ensure data integrity.

**Compliance Requirements**: healthcare privacy compliance requires proper data validation and sanitization to protect patient information from malicious inputs.

## TASK CONTEXT

**Functional Requirement**: Comprehensive input validation and sanitization for all user inputs with all endpoints validating input data with proper error handling.

**Technical Requirement**: Schema-based validation with custom decorators covering all DTOs and API endpoints, with HTML sanitization and XSS prevention.

## BUSINESS DOMAIN CONTEXT

**Healthcare Data Integrity**: Patient medical records, appointment information, and clinical documentation require strict validation to ensure data accuracy and prevent corruption.

**Security Compliance**: Healthcare systems are frequent targets for injection attacks. Comprehensive input validation is essential for protecting sensitive medical data.

## FUNCTIONAL REQUIREMENTS

1. **DTO Validation**: All request DTOs must use class-validator with comprehensive validation rules
2. **XSS Prevention**: HTML content sanitization using DOMPurify-equivalent server-side sanitization
3. **SQL Injection Prevention**: Parameterized queries enforcement and input sanitization
4. **Custom Validation**: Healthcare-specific validation decorators (medical record numbers, insurance IDs, etc.)
5. **Error Handling**: Detailed validation error responses without exposing internal structure

## TECHNICAL REQUIREMENTS

**Validation Framework Structure**:
```typescript
// Expected DTO validation pattern
export class CreatePatientDto {
  @IsNotEmpty()
  @IsString()
  @Length(2, 50)
  @Matches(/^[a-zA-Z\s]+$/, { message: 'Name must contain only letters and spaces' })
  firstName: string;

  @IsEmail()
  @IsNotEmpty()
  email: string;

  @IsOptional()
  @IsPhoneNumber()
  phone?: string;

  @IsDateString()
  @IsBefore(new Date().toISOString())
  dateOfBirth: string;
}

// Custom healthcare validators
@ValidatorConstraint({ name: 'medical-record-number', async: false })
export class MedicalRecordNumberValidator implements ValidatorConstraintInterface {
  validate(value: string): boolean {
    // Medical record number format validation
  }
}
```

**Sanitization Pipeline**:
- Server-side HTML sanitization for rich text inputs
- Special character encoding for database safety
- Input normalization and formatting
- File upload validation and sanitization

## TESTING REQUIREMENTS

**Unit Tests**:
- Test all DTO validation rules with valid and invalid inputs
- Test custom validation decorators
- Test sanitization functionality for various input types
- Test error message generation and formatting
- Test edge cases and boundary conditions

**Integration Tests**:
- Test end-to-end validation through API endpoints
- Test validation error responses in HTTP format
- Test sanitization in actual request processing
- Verify SQL injection prevention

**Security Tests**:
- XSS attempt prevention testing
- SQL injection attempt testing
- File upload security validation
- Malicious input handling testing

## DOCUMENTATION REQUIREMENTS

- Document validation rules and custom decorators
- Create validation error handling guide
- Update API documentation with validation requirements
- Document sanitization policies and procedures

## VALIDATION CRITERIA

- [ ] All API endpoints implement comprehensive input validation using class-validator
- [ ] Custom healthcare-specific validation decorators implemented and tested
- [ ] XSS prevention through server-side HTML sanitization operational
- [ ] SQL injection prevention through parameterized queries enforced
- [ ] Comprehensive error handling for validation failures with user-friendly messages
- [ ] File upload validation and sanitization implemented
- [ ] Unit tests achieve >= 95% coverage for validation functionality
- [ ] Integration tests verify end-to-end validation behavior
- [ ] Security tests confirm XSS and injection attack prevention
- [ ] Performance impact of validation within acceptable limits (<50ms per request)
- [ ] Documentation updated with validation rules and error handling procedures

---

# PROMPT 5: Authentication Security Enhancement

Enhance the existing JWT authentication system with secure session management, brute force protection, token handling best practices, and comprehensive security logging to provide robust authentication security.

## PROJECT CONTEXT

**Tech Stack**: NestJS backend with JWT authentication integration, JWT token handling, existing authentication guards and decorators.

**Current Authentication State**:
- JWT authentication integration established
- JWT token validation implemented
- Basic authentication guards in place
- User role and permission system operational

## EPIC CONTEXT

**Business Requirements**: Healthcare application requiring secure authentication to protect sensitive patient data with healthcare privacy-compliant access controls.

**Security Model**: Zero trust authentication model requiring comprehensive session management, brute force protection, and audit logging for all authentication events.

## TASK CONTEXT

**Functional Requirement**: Secure authentication implementation with session management, secure token handling, session timeout, and brute force protection.

**Technical Requirement**: Authentication security enhancements must maintain current functionality while adding comprehensive security controls and logging.

## BUSINESS DOMAIN CONTEXT

**Healthcare Access Control**: Medical professionals require secure but efficient access to patient data. Authentication must be robust without impeding clinical workflows.

**Compliance Requirements**: healthcare privacy mandates secure authentication with comprehensive audit trails for all access to protected health information.

## FUNCTIONAL REQUIREMENTS

1. **Secure Token Handling**: JWT refresh token rotation and secure storage practices
2. **Session Management**: Configurable session timeouts with automatic renewal for active users
3. **Brute Force Protection**: Account lockout after failed login attempts with exponential backoff
4. **Security Logging**: Comprehensive authentication event logging for audit compliance
5. **Multi-Factor Authentication**: Enhanced MFA support for high-privilege accounts

## TECHNICAL REQUIREMENTS

**Enhanced Authentication Architecture**:
```typescript
// Expected authentication enhancement structure
@Injectable()
export class EnhancedAuthService extends AuthService {
  async authenticateUser(credentials: LoginCredentials): Promise<AuthResult> {
    // Brute force protection check
    // Rate limiting for authentication attempts
    // Secure credential validation
    // Session creation with security controls
    // Comprehensive security logging
  }

  async refreshToken(refreshToken: string): Promise<TokenRefreshResult> {
    // Token rotation implementation
    // Session validation and renewal
    // Security event logging
  }

  async logSecurityEvent(event: SecurityEvent): Promise<void> {
    // Structured security logging
    // Audit trail maintenance
  }
}

// Security event tracking
interface SecurityEvent {
  eventType: 'login_success' | 'login_failure' | 'token_refresh' | 'logout';
  userId?: string;
  ipAddress: string;
  userAgent: string;
  timestamp: Date;
  additionalData?: Record<string, any>;
}
```

**Session Security Controls**:
- Configurable session timeout policies
- Session activity tracking and renewal
- Concurrent session management
- Secure session invalidation

## TESTING REQUIREMENTS

**Unit Tests**:
- Test brute force protection logic and account lockout
- Test token refresh and rotation functionality
- Test session timeout and renewal mechanisms
- Test security event logging accuracy
- Test authentication flow with various scenarios

**Integration Tests**:
- Test end-to-end authentication with security controls
- Test session management across requests
- Test brute force protection in realistic scenarios
- Verify security logging integration

**Security Tests**:
- Penetration testing for authentication bypass attempts
- Brute force attack simulation and protection verification
- Token security and JWT vulnerability testing
- Session hijacking prevention testing

## DOCUMENTATION REQUIREMENTS

- Update authentication security documentation
- Document session management policies and configuration
- Create security incident response procedures for authentication events
- Update API documentation with enhanced authentication requirements

## VALIDATION CRITERIA

- [ ] Secure JWT token handling with refresh token rotation implemented
- [ ] Session management with configurable timeouts and automatic renewal operational
- [ ] Brute force protection with account lockout and exponential backoff functional
- [ ] Comprehensive security event logging for all authentication activities
- [ ] Enhanced MFA support for high-privilege accounts implemented
- [ ] Session security controls preventing hijacking and unauthorized access
- [ ] Unit tests achieve >= 95% coverage for authentication security functionality
- [ ] Integration tests verify end-to-end authentication security
- [ ] Security tests confirm protection against common authentication attacks
- [ ] Performance impact of security enhancements within acceptable limits
- [ ] Security review validates authentication security implementation
- [ ] Documentation updated with security procedures and incident response