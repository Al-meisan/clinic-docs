# TASK-005: AWS Cognito Integration

## Task Overview

### Goal
**What to Build:** Configure AWS Cognito user pool with custom attributes for healthcare roles, integrate with backend authentication system, and establish OAuth2 scope-based authorization for the MedFlow healthcare management platform.

**Why:** Provides enterprise-grade authentication and user management specifically tailored for healthcare environments, supporting role-based access control with medical licensing validation and scope-based permissions.

**Definition of Done:** AWS Cognito user pool operational with custom attributes, backend can validate JWT tokens, user registration/login workflows functional, and role-based scopes properly assigned.

### Scope
```yaml
task_type: "BACKEND"
complexity: "HIGH"
```

## Requirements

### Functional Requirements
1. **Cognito User Pool Configuration**
   - Description: Create user pool with healthcare-specific custom attributes, password policies, and MFA options
   - Acceptance Criteria: User pool supports medical license validation, role assignment, and clinic association
   - Priority: HIGH

2. **Custom Attributes for Healthcare Roles**
   - Description: Define custom attributes for medical licenses, specialties, clinic association, and permission scopes
   - Acceptance Criteria: Users can be assigned healthcare provider roles with proper credentials validation
   - Priority: HIGH

3. **OAuth2 Scope Implementation**
   - Description: Implement scope-based authorization system aligned with healthcare workflow permissions
   - Acceptance Criteria: Scopes properly control access to patient data, clinical documentation, and billing features
   - Priority: HIGH

4. **Backend JWT Integration**
   - Description: Configure JWT token validation, user context extraction, and automatic token refresh
   - Acceptance Criteria: Backend validates all tokens, extracts user context, and handles token expiration gracefully
   - Priority: HIGH

### Technical Requirements
```yaml
performance:
  - requirement: "Token validation response time"
    target: "< 100ms for JWT token validation"
    
security:
  - requirement: "Healthcare data protection compliance"
    implementation: "Healthcare privacy-compliant user management with audit logging"
    
compatibility:
  - requirement: "Multi-language user attributes"
    scope: "Support for Arabic and French user profiles"
```

## Implementation Details

### Technical Approach
```yaml
cognito_configuration:
  user_pool_settings:
    - mfa_configuration: "OPTIONAL"
    - password_policy: "Strong passwords with 12+ characters"
    - account_recovery: "Email only verification"
    - username_attributes: ["email"]
    - auto_verified_attributes: ["email"]
    
  custom_attributes:
    - medical_license: "string (required for providers)"
    - license_state: "string (license jurisdiction)"
    - specialties: "string (JSON array of medical specialties)"
    - clinic_id: "string (associated clinic UUID)"
    - user_role: "string (system role assignment)"
    - preferred_language: "string (ar/fr language preference)"
    - npi_number: "string (National Provider Identifier)"
    
  user_groups:
    - SystemAdministrators (admin:full scope)
    - ClinicManagers (clinic:manage + operational scopes)
    - HealthcareProviders (clinical + prescription scopes)
    - ClinicalStaff (clinical scopes, limited patient write)
    - AdministrativeStaff (patient + appointment scopes)

backend_integration:
  jwt_configuration:
    - token_validation: "AWS Cognito JWT verification"
    - user_context: "Extract user ID, clinic ID, and scopes"
    - token_refresh: "Automatic refresh token handling"
    - scope_validation: "Middleware for endpoint access control"
    
  authentication_flow:
    - login: "Email/password with Cognito authentication"
    - registration: "User signup with email verification"
    - password_reset: "Secure password reset workflow"
    - profile_update: "User profile and custom attribute updates"
```

### Integration Points
```yaml
dependencies:
  - dependency: "AWS account with Cognito service access"
    type: "SERVICE"
    status: "AVAILABLE"
    
  - dependency: "Backend NestJS application"
    type: "API"
    status: "IN_PROGRESS"
    
  - dependency: "Terraform infrastructure for Cognito resources"
    type: "INFRASTRUCTURE"
    status: "PLANNED"
    
provides:
  - deliverable: "User authentication service"
    interface: "JWT token validation and user context"
    consumers: "All backend API endpoints requiring authentication"
    
  - deliverable: "Role-based authorization system"
    interface: "OAuth2 scopes and permission validation"
    consumers: "Frontend application and API access control"
    
  - deliverable: "User management interface"
    interface: "User registration, profile updates, and role assignment"
    consumers: "Admin interfaces and user self-service features"
```

### Code Patterns to Follow

**Cognito Service Integration:**
```typescript
// src/infrastructure/aws/cognito/cognito.service.ts
@Injectable()
export class CognitoService {
  private cognitoIdentityServiceProvider: CognitoIdentityServiceProvider;
  
  constructor(private configService: ConfigService) {
    this.cognitoIdentityServiceProvider = new CognitoIdentityServiceProvider({
      region: this.configService.get('AWS_REGION'),
    });
  }
  
  async authenticateUser(email: string, password: string): Promise<AuthResult> {
    try {
      const params = {
        AuthFlow: 'USER_PASSWORD_AUTH',
        ClientId: this.configService.get('COGNITO_CLIENT_ID'),
        AuthParameters: {
          USERNAME: email,
          PASSWORD: password,
        },
      };
      
      const result = await this.cognitoIdentityServiceProvider.initiateAuth(params).promise();
      return this.parseAuthResult(result);
    } catch (error) {
      this.logger.error('Authentication failed', error);
      throw new UnauthorizedException('Invalid credentials');
    }
  }
}
```

**JWT Authentication Guard:**
```typescript
// src/common/guards/jwt-auth.guard.ts
@Injectable()
export class JwtAuthGuard implements CanActivate {
  constructor(
    private cognitoService: CognitoService,
    private reflector: Reflector,
  ) {}
  
  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);
    
    if (!token) {
      throw new UnauthorizedException('Authentication token required');
    }
    
    try {
      const user = await this.cognitoService.validateToken(token);
      request.user = user;
      return true;
    } catch (error) {
      throw new UnauthorizedException('Invalid or expired token');
    }
  }
}
```

**Scope-based Authorization Guard:**
```typescript
// src/common/guards/scopes.guard.ts
@Injectable()
export class ScopesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}
  
  canActivate(context: ExecutionContext): boolean {
    const requiredScopes = this.reflector.getAllAndOverride<string[]>('scopes', [
      context.getHandler(),
      context.getClass(),
    ]);
    
    if (!requiredScopes) {
      return true;
    }
    
    const { user } = context.switchToHttp().getRequest();
    const userScopes = user.scopes || [];
    
    return requiredScopes.some(scope => userScopes.includes(scope));
  }
}
```

**Custom Decorators:**
```typescript
// src/common/decorators/require-scopes.decorator.ts
export const RequireScopes = (...scopes: string[]) => 
  SetMetadata('scopes', scopes);

// src/common/decorators/current-user.decorator.ts
export const CurrentUser = createParamDecorator(
  (data: unknown, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest();
    return request.user;
  },
);
```

## Testing Requirements

### Test Coverage
```yaml
unit_tests:
  - focus: "Cognito service methods and JWT validation"
  - coverage_target: "90%"
  - key_scenarios: ["token_validation", "user_authentication", "scope_checking"]
  
integration_tests:
  - focus: "Authentication flow and user management"
  - test_environment: "Test Cognito user pool"
  - key_flows: ["login_workflow", "token_refresh", "scope_validation"]
  
e2e_tests:
  - focus: "Complete authentication and authorization workflows"
  - user_scenarios: ["user_registration", "login_logout", "protected_endpoint_access"]
```

### Test Scenarios
```yaml
happy_path:
  - scenario: "User authentication and token validation"
    steps: ["User login", "Token generation", "Protected endpoint access"]
    expected_result: "Successful authentication with proper user context"
    
error_cases:
  - scenario: "Invalid credentials and expired tokens"
    trigger: "Wrong password or expired JWT token"
    expected_behavior: "Clear error messages with appropriate HTTP status codes"
    
edge_cases:
  - scenario: "Scope validation with insufficient permissions"
    conditions: "User accessing endpoint requiring higher-level scopes"
    expected_behavior: "403 Forbidden with scope requirement information"
```

## Context References

### From Project
```yaml
tech_stack_references:
  - "AWS Cognito for user authentication and management"
  - "OAuth2 scope-based authorization system"
  - "JWT tokens with custom claims for healthcare roles"
  - "NestJS guards and decorators for access control"
  
architecture_patterns:
  - "Role-based access control with healthcare-specific scopes"
  - "Multi-tenant architecture with clinic boundaries"
  - "Audit trail for authentication and authorization events"
  
code_standards:
  - "Consistent error handling for authentication failures"
  - "Secure token storage and transmission"
  - "Proper scope validation for all protected endpoints"
```

### From Epic
```yaml
business_rules:
  - "OAuth2 scope-based authorization system"
  - "Healthcare provider roles require medical license validation"
  - "Cross-clinic data access is prohibited unless explicitly authorized"
  - "All authentication events must be logged for audit purposes"
  
integration_points:
  - "Foundation for all user management and access control"
  - "Required for frontend authentication and protected routes"
  - "Enables role-based feature access across all epics"
```

## Acceptance Criteria

### Functional Acceptance
- [ ] AWS Cognito user pool configured with healthcare-specific attributes
- [ ] User registration and login workflows operational
- [ ] JWT token validation working with proper user context extraction
- [ ] Scope-based authorization system functional
- [ ] Role assignment and permission validation working

### Technical Acceptance  
- [ ] Backend authentication guards and decorators implemented
- [ ] Token refresh mechanism working automatically
- [ ] Custom user attributes properly stored and retrieved
- [ ] Multi-language support for user profiles functional
- [ ] Audit logging captures all authentication events

### Quality Acceptance
- [ ] Authentication flow tested with all user roles
- [ ] Security testing confirms proper access control
- [ ] Performance meets token validation time requirements
- [ ] Error handling provides clear feedback without exposing sensitive information
- [ ] Documentation includes user management and role assignment procedures
