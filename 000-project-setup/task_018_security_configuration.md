# TASK-018: Security Configuration

## Task Overview

### Goal
**What to Build:** Implement comprehensive security configuration including security headers, CORS, rate limiting, input validation, and security best practices for the clinic management system.

**Why:** Critical for healthcare data protection, Algerian healthcare privacy compliance, preventing security vulnerabilities, and protecting sensitive patient information from unauthorized access and attacks.

**Definition of Done:** Complete security hardening implemented across all application layers with penetration testing validation and compliance verification.

### Scope
```yaml
task_type: "FULLSTACK"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Security Headers Configuration**
   - Description: Implement comprehensive HTTP security headers for all responses
   - Acceptance Criteria: All security headers configured including CSP, HSTS, X-Frame-Options, etc.
   - Priority: HIGH

2. **CORS Configuration**
   - Description: Configure Cross-Origin Resource Sharing policies for API security
   - Acceptance Criteria: Restrictive CORS policy allowing only authorized domains
   - Priority: HIGH

3. **Rate Limiting Implementation**
   - Description: Implement rate limiting to prevent abuse and DDoS attacks
   - Acceptance Criteria: Per-user and per-endpoint rate limiting with proper error responses
   - Priority: HIGH

4. **Input Validation and Sanitization**
   - Description: Comprehensive input validation and sanitization for all user inputs
   - Acceptance Criteria: All endpoints validate input data with proper error handling
   - Priority: HIGH

5. **Authentication Security**
   - Description: Secure authentication implementation with session management
   - Acceptance Criteria: Secure token handling, session timeout, and brute force protection
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Security checks add minimal latency per request"
    target: "Minimal performance impact from security measures"
    
  - requirement: "Rate limiting response time"
    target: "Fast rate limit decision making"
    
security:
  - requirement: "OWASP Top 10 vulnerability protection"
    implementation: "Comprehensive protection against common web vulnerabilities"
    
  - requirement: "Healthcare privacy compliance security controls"
    implementation: "Algerian healthcare-specific security requirements and audit controls"
    
  - requirement: "Zero trust security model"
    implementation: "Authentication and authorization required for all operations"
    
compatibility:
  - requirement: "Browser compatibility for security features"
    scope: "Security headers supported across all modern browsers"
    
  - requirement: "Mobile app security integration"
    scope: "Security policies compatible with mobile applications"
```

## Implementation Details

### Technical Approach
```yaml
# Security Headers Implementation
security_headers:
  - header: "Content-Security-Policy (CSP)"
    purpose: "Prevent XSS and code injection attacks"
    configuration: "Strict policy with nonce-based script execution"
    
  - header: "HTTP Strict Transport Security (HSTS)"
    purpose: "Enforce HTTPS connections"
    configuration: "Max-age of 1 year with includeSubDomains"
    
  - header: "X-Frame-Options"
    purpose: "Prevent clickjacking attacks"
    configuration: "DENY for all pages"
    
  - header: "X-Content-Type-Options"
    purpose: "Prevent MIME type sniffing"
    configuration: "nosniff for all responses"

# CORS Configuration
cors_policy:
  - aspect: "Allowed Origins"
    configuration: "Whitelist of authorized domains only"
    environments: "Different policies per environment"
    
  - aspect: "Allowed Methods"
    configuration: "Minimum required HTTP methods"
    restriction: "No unnecessary methods exposed"
    
  - aspect: "Allowed Headers"
    configuration: "Essential headers only"
    security: "Prevent header-based attacks"

# Rate Limiting Strategy
rate_limiting:
  - level: "Global Rate Limiting"
    limits: "1000 requests per hour per IP"
    purpose: "Prevent DDoS and abuse"
    
  - level: "API Endpoint Limiting"
    limits: "Specific limits per endpoint type"
    configuration: "Authentication: 10/min, Search: 100/hour"
    
  - level: "User-based Limiting"
    limits: "Per-user quotas based on role"
    implementation: "Role-based rate limiting with Redis backend"

# Input Validation Framework
input_validation:
  - approach: "Schema-based Validation"
    implementation: "class-validator with custom decorators"
    coverage: "All DTOs and API endpoints"
    
  - approach: "Sanitization Pipeline"
    implementation: "HTML sanitization and XSS prevention"
    tools: "DOMPurify for frontend, validator.js for backend"
    
  - approach: "SQL Injection Prevention"
    implementation: "Parameterized queries and ORM usage"
    verification: "Static analysis and penetration testing"
```

### Integration Points
```yaml
dependencies:
  - dependency: "NestJS Backend Application (TASK-003)"
    type: "APPLICATION"
    status: "AVAILABLE"
    
  - dependency: "React Frontend Application (TASK-007)"
    type: "APPLICATION"
    status: "AVAILABLE"
    
  - dependency: "AWS Cognito Authentication (TASK-005)"
    type: "SERVICE"
    status: "AVAILABLE"
    
provides:
  - deliverable: "Security Middleware"
    interface: "NestJS guards, interceptors, and middleware"
    consumers: "All API endpoints and application routes"
    
  - deliverable: "Frontend Security Components"
    interface: "React security hooks and components"
    consumers: "All React components and pages"
    
  - deliverable: "Security Configuration"
    interface: "Environment-specific security settings"
    consumers: "DevOps, infrastructure, and application configuration"
```

### Code Patterns to Follow
```typescript
// Security Headers Middleware (NestJS)
import { Injectable, NestMiddleware } from '@nestjs/common';
import { Request, Response, NextFunction } from 'express';
import helmet from 'helmet';

@Injectable()
export class SecurityHeadersMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: NextFunction) {
    // Configure Helmet with comprehensive security headers
    helmet({
      contentSecurityPolicy: {
        directives: {
          defaultSrc: ["'self'"],
          scriptSrc: ["'self'", "'nonce-{nonce}'"],
          styleSrc: ["'self'", "'unsafe-inline'"],
          imgSrc: ["'self'", "data:", "https:"],
          connectSrc: ["'self'", process.env.API_URL],
          fontSrc: ["'self'"],
          objectSrc: ["'none'"],
          mediaSrc: ["'self'"],
          frameSrc: ["'none'"]
        },
      },
      hsts: {
        maxAge: 31536000, // 1 year
        includeSubDomains: true,
        preload: true
      },
      noSniff: true,
      frameguard: { action: 'deny' },
      xssFilter: true,
      referrerPolicy: { policy: 'strict-origin-when-cross-origin' }
    })(req, res, next);
  }
}

// CORS Configuration
import { CorsOptions } from '@nestjs/common/interfaces/external/cors-options.interface';

export const corsConfig: CorsOptions = {
  origin: (origin, callback) => {
    const allowedOrigins = process.env.ALLOWED_ORIGINS?.split(',') || [];
    
    // Allow requests with no origin (mobile apps, Postman, etc.)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS policy'), false);
    }
  },
  methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-Requested-With',
    'X-Correlation-ID'
  ],
  credentials: true,
  maxAge: 86400, // 24 hours
};

// Rate Limiting Configuration
import { ThrottlerModule, ThrottlerGuard } from '@nestjs/throttler';
import { APP_GUARD } from '@nestjs/core';

@Module({
  imports: [
    ThrottlerModule.forRoot({
      ttl: 60, // Time window in seconds
      limit: 100, // Max requests per time window
      storage: new ThrottlerStorageRedisService(redisClient),
    }),
  ],
  providers: [
    {
      provide: APP_GUARD,
      useClass: ThrottlerGuard,
    },
  ],
})
export class SecurityModule {}

// Custom Rate Limiting Decorator
import { SetMetadata } from '@nestjs/common';

export const RateLimit = (limit: number, ttl: number) =>
  SetMetadata('rateLimit', { limit, ttl });

// Usage:
@Controller('auth')
export class AuthController {
  @Post('login')
  @RateLimit(5, 900) // 5 attempts per 15 minutes
  async login(@Body() loginDto: LoginDto) {
    // Login implementation
  }
}

// Input Validation with Custom Decorators
import { 
  IsEmail, 
  IsString, 
  MinLength, 
  MaxLength, 
  Matches,
  IsDateString,
  Transform 
} from 'class-validator';
import { sanitize } from 'class-sanitizer';

export class CreatePatientDto {
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  @Matches(/^[a-zA-Z\s\-']+$/, {
    message: 'First name can only contain letters, spaces, hyphens, and apostrophes'
  })
  @Transform(({ value }) => sanitize(value))
  firstName: string;

  @IsString()
  @MinLength(2)
  @MaxLength(50)
  @Matches(/^[a-zA-Z\s\-']+$/, {
    message: 'Last name can only contain letters, spaces, hyphens, and apostrophes'
  })
  @Transform(({ value }) => sanitize(value))
  lastName: string;

  @IsEmail()
  @Transform(({ value }) => value.toLowerCase().trim())
  email: string;

  @IsDateString()
  dateOfBirth: string;
}

// Authentication Security Guard
import { Injectable, CanActivate, ExecutionContext, UnauthorizedException } from '@nestjs/common';
import { JwtService } from '@nestjs/jwt';
import { Request } from 'express';

@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(private jwtService: JwtService) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest<Request>();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      throw new UnauthorizedException('Access token is required');
    }

    try {
      const payload = await this.jwtService.verifyAsync(token, {
        secret: process.env.JWT_SECRET,
      });
      
      // Security checks
      if (this.isTokenExpired(payload)) {
        throw new UnauthorizedException('Token has expired');
      }
      
      if (this.isTokenBlacklisted(token)) {
        throw new UnauthorizedException('Token has been revoked');
      }
      
      request.user = payload;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid access token');
    }
  }

  private extractTokenFromHeader(request: Request): string | undefined {
    const [type, token] = request.headers.authorization?.split(' ') ?? [];
    return type === 'Bearer' ? token : undefined;
  }

  private isTokenExpired(payload: any): boolean {
    return Date.now() >= payload.exp * 1000;
  }

  private isTokenBlacklisted(token: string): boolean {
    // Check against Redis blacklist
    return false; // Implementation depends on blacklist strategy
  }
}

// Frontend Security Hook
import { useEffect, useCallback } from 'react';
import DOMPurify from 'dompurify';

export const useSecurity = () => {
  // CSP nonce management
  const getNonce = useCallback(() => {
    const metaTag = document.querySelector('meta[name="csp-nonce"]');
    return metaTag?.getAttribute('content') || '';
  }, []);

  // HTML sanitization
  const sanitizeHtml = useCallback((dirty: string): string => {
    return DOMPurify.sanitize(dirty, {
      ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'u'],
      ALLOWED_ATTR: [],
    });
  }, []);

  // Input validation
  const validateInput = useCallback((value: string, type: 'email' | 'name' | 'text'): boolean => {
    const patterns = {
      email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
      name: /^[a-zA-Z\s\-']{2,50}$/,
      text: /^[a-zA-Z0-9\s\-'.,!?]{1,200}$/,
    };
    return patterns[type].test(value);
  }, []);

  // Security event monitoring
  const reportSecurityEvent = useCallback((event: string, details: any) => {
    // Report to monitoring service
    console.warn('Security Event:', { event, details, timestamp: new Date() });
  }, []);

  return {
    getNonce,
    sanitizeHtml,
    validateInput,
    reportSecurityEvent,
  };
};

// Secure API Client
class SecureApiClient {
  private baseURL: string;
  private timeout: number = 30000;

  constructor(baseURL: string) {
    this.baseURL = baseURL;
    this.setupInterceptors();
  }

  private setupInterceptors() {
    // Request interceptor for security headers
    axios.interceptors.request.use((config) => {
      // Add security headers
      config.headers['X-Requested-With'] = 'XMLHttpRequest';
      config.headers['X-Content-Type-Options'] = 'nosniff';
      
      // Add CSRF token if available
      const csrfToken = this.getCSRFToken();
      if (csrfToken) {
        config.headers['X-CSRF-Token'] = csrfToken;
      }

      return config;
    });

    // Response interceptor for error handling
    axios.interceptors.response.use(
      (response) => response,
      (error) => {
        if (error.response?.status === 429) {
          // Rate limit exceeded
          throw new Error('Too many requests. Please try again later.');
        }
        return Promise.reject(error);
      }
    );
  }

  private getCSRFToken(): string | null {
    const metaTag = document.querySelector('meta[name="csrf-token"]');
    return metaTag?.getAttribute('content') || null;
  }
}
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Security middleware, guards, and validation functions"
  - coverage_target: "95%"
  - key_scenarios: ["input validation", "rate limiting", "authentication checks"]
  
integration_tests:
  - focus: "End-to-end security flow testing"
  - test_environment: "Security testing environment with simulated attacks"
  - key_flows: ["authentication flow", "CORS policy enforcement", "rate limit handling"]
  
security_tests:
  - focus: "Penetration testing and vulnerability assessment"
  - tools: "OWASP ZAP, Burp Suite, custom security test suite"
  - scope: ["SQL injection", "XSS", "CSRF", "authentication bypass"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "Legitimate user requests processed successfully"
    steps: ["User authenticates", "Makes authorized request", "Security checks pass", "Response received"]
    expected_result: "Request processed with appropriate security headers"
    
security_tests:
  - scenario: "SQL injection attempt blocked"
    trigger: "Malicious SQL injection in input parameters"
    expected_behavior: "Input validation rejects request with error message"
    
  - scenario: "Rate limit enforcement"
    trigger: "Excessive requests from single IP/user"
    expected_behavior: "429 status code returned with retry-after header"
    
  - scenario: "XSS attempt prevented"
    trigger: "Script injection in user input"
    expected_behavior: "Input sanitized and CSP headers prevent execution"
    
edge_cases:
  - scenario: "Security headers in error responses"
    conditions: "Application errors and exception responses"
    expected_behavior: "Security headers present even in error responses"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "NestJS security middleware and guards"
  - "React security hooks and input validation"
  - "AWS Cognito secure authentication integration"
  
architecture_patterns:
  - "Defense in depth security strategy"
  - "Zero trust security model implementation"
  - "Secure coding practices across all layers"
  
code_standards:
  - "Input validation on all user inputs"
  - "Secure error handling without information disclosure"
  - "Security-focused code review requirements"
```

### From Epic
```yaml
business_rules:
  - "Healthcare data requires Algerian healthcare privacy compliance"
  - "Patient information must be securely stored and access-controlled"
  - "Audit trail required for all data access and modifications"
  
domain_model:
  - "Patient data access requires authenticated and authorized users"
  - "Provider permissions based on role and scope"
  - "Appointment data protected with appropriate access controls"
  
integration_points:
  - "AWS security services integration for compliance"
  - "Third-party API security for external integrations"
  - "Multi-tenant security isolation requirements"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] All security headers implemented and configured correctly
- [ ] CORS policy enforced with whitelist-based origin control
- [ ] Rate limiting operational with per-user and per-endpoint limits
- [ ] Input validation implemented for all API endpoints
- [ ] Authentication security with session management and brute force protection

### Technical Acceptance  
- [ ] Code follows secure coding standards and best practices
- [ ] Security tests written and passing (>= 95% coverage)
- [ ] Penetration testing completed with no critical vulnerabilities
- [ ] Performance impact of security measures within acceptable limits
- [ ] Healthcare privacy compliance requirements met and documented

### Quality Acceptance
- [ ] Security review completed and approved by security team
- [ ] No critical security issues identified in static analysis
- [ ] Security documentation updated with implementation details
- [ ] Security incident response procedures documented
- [ ] Ready for production deployment with security hardening complete

---
**Last Updated:** August 23, 2025